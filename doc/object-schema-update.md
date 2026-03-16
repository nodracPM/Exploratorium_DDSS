# Object Schema Update

## Summary

This update adds object-level `reference` and `year` fields, plus a localized `desc` field on `object_desc`.

## Files Changed

- [db/ddl.sql](/home/pmcc/Desktop/social service/Exploratorium_MMSS/db/ddl.sql)
- [scripts/csv2sql.sh](/home/pmcc/Desktop/social service/Exploratorium_MMSS/scripts/csv2sql.sh)
- [doc/object-schema-update.md](/home/pmcc/Desktop/social service/Exploratorium_MMSS/doc/object-schema-update.md)

## Schema Changes

### `object`

Added:

- `reference TEXT DEFAULT NULL`
- `year INTEGER DEFAULT NULL`

Constraints added:

- `CHECK (reference IS NULL OR trim(reference) <> '')`
- `CHECK (year IS NULL OR year BETWEEN 0 AND 9999)`

Reasoning:

- `reference` remains optional, but blank or whitespace-only values are rejected.
- `year` remains optional, but when present it must be a valid non-negative year-like value.

### `object_desc`

Added:

- `desc TEXT DEFAULT NULL`

Constraint added:

- `CHECK (desc IS NULL OR trim(desc) <> '')`

Reasoning:

- `desc` is optional to preserve compatibility with existing rows and partial imports.
- When supplied, the value must contain real content.

## View Changes

### `v_contexts`

Added:

- `object_desc.desc AS ObjectDescription`
- `object.reference AS ObjectReference`
- `object.year AS ObjectReferenceYear`

### `v_object_context`

Added:

- `od.desc AS Description`
- `o.reference AS Reference`
- `o.year AS ReferenceYear`

## Import Pipeline Changes

The ODS import path now supports the new object fields end-to-end through [scripts/csv2sql.sh](/home/pmcc/Desktop/social service/Exploratorium_MMSS/scripts/csv2sql.sh).

### `render_object`

Updated to load:

- `object_id`
- `object_code`
- `reference`
- `year`

Implementation details:

- `reference` is emitted with `str_or_NULL`, so empty cells become `NULL`.
- `year` is emitted with a new `int_or_NULL` helper, so empty cells become `NULL` and non-numeric values fail fast during import generation.

### `render_object_desc`

Updated to load:

- English label plus `object_desc_en` into `object_desc.desc`
- Spanish label plus `object_desc_es` into `object_desc.desc`

Implementation details:

- Empty description cells are converted to `NULL`.
- Non-empty descriptions remain language-specific and are inserted into the corresponding `object_desc` rows.

## Design Notes

- `reference` and `year` were placed on `object` because they describe the canonical object, not a specific translation.
- `desc` was placed on `object_desc` because descriptive text can vary by language.
- All three new fields are nullable so existing data remains valid.
- The added checks improve data quality without making migration brittle.

## Compatibility Impact

- Existing rows remain valid because the new fields default to `NULL`.
- Positional `INSERT` statements for `object` and `object_desc` may need adjustment if they do not specify column names.
- `SELECT *` against the updated views will now return additional columns.

## Operational Result

After these changes, updating the object catalog data in the `.ods` source and running the database import script is sufficient to load:

- `object.reference`
- `object.year`
- `object_desc.desc` for `en`
- `object_desc.desc` for `es`

provided the corresponding ODS-derived columns are present:

- `reference`
- `year`
- `object_desc_en`
- `object_desc_es`

## Recommended Next Steps

- Keep using explicit column names in any future hand-written `INSERT` statements.
- Add a regression test that runs the importer with a sample object row containing `reference`, `year`, `object_desc_en`, and `object_desc_es`.
- Add a negative test confirming non-numeric `year` values are rejected during import generation.
