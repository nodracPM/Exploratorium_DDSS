# Object Schema Update

## Summary

This update adds object-level `reference` and `year` fields, plus a localized `desc` field on `object_desc`.

## Files Changed

- [db/ddl.sql](/home/pmcc/Desktop/social service/Exploratorium_MMSS/db/ddl.sql)
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

## Design Notes

- `reference` and `year` were placed on `object` because they describe the canonical object, not a specific translation.
- `desc` was placed on `object_desc` because descriptive text can vary by language.
- All three new fields are nullable so existing data remains valid.
- The added checks improve data quality without making migration brittle.

## Compatibility Impact

- Existing rows remain valid because the new fields default to `NULL`.
- Positional `INSERT` statements for `object` and `object_desc` may need adjustment if they do not specify column names.
- `SELECT *` against the updated views will now return additional columns.

## Recommended Next Steps

- Update import scripts to populate `reference`, `year`, and `desc`.
- Prefer explicit column lists in `INSERT` statements.
- Add tests covering blank `reference`, blank `desc`, and invalid `year` values.
