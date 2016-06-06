# Feature Storage in OMERO

This is a proposal for storing image features in OMERO, following on from the work done in [OMERO.features](https://github.com/ome/omero-features).

Initial use-case: Storage of WND-CHARM features in the IDR.


## Summary

Initially:
- Use OMERO.tables
- Searchable columns (typically metadata) must be of atomic/scalar type
- Feature value columns can be of atomic/scalar or array type

In future:
- Annotate OMERO objects with the table and row offset, since bulk retrieval of tables rows by offset are supported.

Main issues:
- PyTables HDF5 tables are limited in the number of columns, which means array column types may be needed.
- Array column types cannot be queried using PyTables
- PyTables queries are limited in the number of parameters that can be passed to queries such as `SELECT rows WHERE id in (i0, i1, ...)`, which makes it difficult to do a cross-join between the OMERO database and OMERO.tables.


## OMERO.tables format


### Metadata columns

A column containing information about the features, for example object IDs or parameters.

These must be of scalar type, currently these are:

OMERO ID column types:
- `FileColumn`
- `ImageColumn`
- `RoiColumn`
- `WellColumn`
- `PlateColumn`

Numeric types:
- `BoolColumn`
- `LongColumn`
- `DoubleColumn`

String type:
- `StringColumn` (fixed width)

There must be at least one ID column type to allow a feature row to be linked to an OMERO object.
Multiple ID columns are allowed, for example to redundantly link a feature to an ROI and Image.
Empty ID columns should be set to `-1`.


### Feature columns

A column containing a feature value or vector.

Feature columns can be of any type, including array types:
- `FloatArrayColumn`
- `DoubleArrayColumn`
- `LongArrayColumn`

Array columns are useful when there are a large number of features since the performance of PyTables decreases when there are a large number of columns.
A disadvantage of array columns is they cannot be queried.


### Column names

Column names beginning with `_` are reserved for internal use.
It is strongly recommended that names match the [regular-expression](http://www.regular-expressions.info/posixbrackets.html) `[[:alpha:]][[:word:]]*`, i.e. any letter followed by zero or more alphanumeric chatters or underscore.
This is equivalent to `[^\W\d_]\w*` in the Python 2 `re` module, or `[A-Za-z][A-Za-z0-9_]` if plain ASCII is used.

For ease of parsing OMERO ID columns should be named in the form `<Type><ID>`, for example an `ImageColumn` should be named `ImageID`.

If the metadata includes the channel, Z-index or T-index the following names are recommended:
- Channel: `C`
- Z-index: `Z`
- T-index: `T`

Zero-based indexing should be used for `C`/`Z`/`T`.


### Column descriptions

Columns have an optional `description` field.
This should be either empty or contain a `json` object including opening/closing braces (`{ }`).
Top-level keys beginning with `_` are reserved for internal use, all other keys will be ignored and can be used for client purposes.

Currently the following internal keys are defined:
- `_metadata` boolean (`true`/`false`): If `true` this is a metadata column, if omitted this is assumed to be `false`.
The intention of this key is to allow analysis of the data in a table without fully understanding the metadata.


## Example

This example has:
- Three metadata columns:
  - `RoiID`: OMERO ROI ID
  - `ImageID`: OMERO image ID
  - `C`: Channel index
- Two feature columns:
  - `Feature-A`: A feature consisting of a single `double`
  - `Feature-B`: A feature consisting of an array of three doubles

|Column type |Roi               |Image             |Long              |Double         |DoubleArray[3]   |
|------------|------------------|------------------|------------------|---------------|-----------------|
|Name        |RoiID             |ImageID           |C                 |Feature-A      |Feature-B        |
|Description |`{_metadata:true}`|`{_metadata:true}`|`{_metadata:true}`|               |                 |
|Example data|101               |23                |1                 |10.54          |[0.23, 3.1, 2.6] |
|Example data|5637              |-1                |0                 |-764567.889    |[-9.0, 12.1, 0.2]|


## Resources

- https://www.openmicroscopy.org/site/support/omero5/developers/Tables.html
- https://www.openmicroscopy.org/site/support/omero5/sysadmins/server-tables.html
- http://www.pytables.org/usersguide/condition_syntax.html
