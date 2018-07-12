# Builtin Processors

DataFlows comes with a few built-in processors which do most of the heavy lifting in many common scenarios, leaving you to implement only the minimum code that is specific to your specific problem.

### Load and Save Data
- **load** - Loads data from various source types (local files, remote URLS, Google Spreadsheets, databases...)
- **printer** - Just prints whatever it sees. Good for debugging.

- **dump_to_path** - Store the results to a specified path on disk, in a valid datapackage
- **dump_to_zip** - Store the results in a valid datapackage, all files archived in one zipped file
- **dump_to_sql** - Store the results in a relational database (creates one or more tables or updates existing tables)

### Manipulate row-by-row
- **delete_fields** - Removes some columns from the data
- **add_computed_field** - Adds new fields whose values are based on existing columns
- **find_replace** - Look for specific patterns in specific fields and replace them with new data
- **set_type** - Parse incoming data based on provided schema, validate the data in the process

### Manipulate the entire resource
- **sort_rows** - Sort incoming data based on key
- **unpivot** - Unpivot a table - convert one row with multiple value columns to multiple rows with one value column
 - **filter_rows** - Filter rows based on inclusive and exclusive value filters

### Manipulate package
- **add_metadata** - Add high-level metadata about your package
- **concatenate** - Concatenate multiple streams of data to a single one, resolving differently named columns along the way
- **duplicate** - Duplicate a single stream of data to make two streams

### API Reference

#### load
Loads data from various source types (local files, remote URLS, Google Spreadsheets, databases...)

```python
def load(path, name=None, **options):
    pass
```

- `path` - location of the data that is to be loaded. This can be either:
    - a local path (e.g. `/path/to/the/data.csv`)
    - a remote URL (e.g. `https://path.to/the/data.csv`)
    - Other supported links, based on the current support of schemes and formats in [tabulator](https://github.com/frictionlessdata/tabulator-py#schemes)
- `options` - based on the loaded file, extra options (e.g. `sheet` for Excel files etc., see the link to tabulator above)

#### printer
Just prints whatever it sees. Good for debugging.

#### dump_to_path
Store the results to a specified path on disk, in a valid datapackage

```python
def dump_to_path(out_path='.', 
                 force_format=True, format='csv', 
                 counters={}, 
                 add_filehash_to_path=False, 	
                 pretty_Descriptor=True):
    pass
```

- `out-path` - Name of the output path where `datapackage.json` will be stored.

  This path will be created if it doesn't exist, as well as internal data-package paths.

  If omitted, then `.` (the current directory) will be assumed.

- `force-format` - Specifies whether to force all output files to be generated with the same format
    - if `True` (the default), all resources will use the same format
    - if `False`, format will be deduced from the file extension. Resources with unknown extensions will be discarded.
- `format` - Specifies the type of output files to be generated (if `force-format` is true): `csv` (the default) or `json`
- `add-filehash-to-path`: Specifies whether to include file md5 hash into the resource path. Defaults to `False`. If `True` Embeds hash in path like so:
    - If original path is `path/to/the/file.ext`
    - Modified path will be `path/to/the/HASH/file.ext`
- `counters` - Specifies whether to count rows, bytes or md5 hash of the data and where it should be stored. An object with the following properties:

    | Key                  | Purpose                                | Default       |
    | -------------------- | -------------------------------------- | ------------- |
    | datapackage-rowcount | the # of rows in the entire package    | count_of_rows |
    | datapackage-bytes    | the # of bytes in the entire package   | bytes         |
    | datapackage-hash     | hash of the data in the entire package | hash          |
    | resource-rowcount    | the # of rows in each resource         | count_of_rows |
    | resource-bytes       | the # of bytes in each resource        | bytes         |
    | resource-hash        | hash of the data in the resource       | hash          |

    Each of these attributes could be set to null in order to prevent the counting.
    Each property could be a dot-separated string, for storing the data inside a nested object (e.g. `stats.rowcount`)
- `pretty-descriptor`: Specifies how datapackage descriptor (`datapackage.json`) file will look like:
    - `False` - descriptor will be written in one line.
    - `True` (default) - descriptor will have indents and new lines for each key, so it becomes more human-readable.


- `add_filehash_to_path` - Should paths for data files be injected with the data hash (i.e. instead of `path/to/data.csv`, have `path/to/<data.csv hash>/data.csv`) (default: `False`)

- `pretty_descriptor` - Should the resulting descriptor be JSON pretty-formatted with indentation for readability, or as compact as possible (default `True`)

#### dump_to_zip
Store the results in a valid datapackage, all files archived in one zipped file

```python
def dump_to_zip(out_file, 
                force_format=True, format='csv', 
                counters={}, 
                add_filehash_to_path=False, pretty_Descriptor=True):
    pass
```

- `out_file` - the path of the output zip file

(rest of parameters are detailed in `dump_to_path`)
#### dump_to_sql
Store the results in a relational database (creates one or more tables or updates existing tables)

### Manipulate row-by-row
#### delete_fields
Delete fields (columns) from streamed resources

`delete_fields` accepts a list of resources and list of fields to remove

_Note: if multiple resources provided, all of them should contain all fields to delete_

```python
def delete_fields(fields, resources=None):
    pass
```

- `fields` - List of field (column) names to be removed
- `resources`
  - A name of a resource to operate on
  - A regular expression matching resource names
  - A list of resource names
  - `None` indicates operation should be done on all resources

#### add_computed_field
Add field(s) to streamed resources

`add_computed_field` accepts a list of resources and fields to add to existing resource. It will output the rows for each resource with new field(s) (columns) in it. `add_computed_field` allows to perform various operations before inserting value into targeted field.

Adds new fields whose values are based on existing columns

```python
def add_computed_field(fields, resources=None):
    pass
```

- `fields` - List of actions to be performed on the targeted fields. Each list item is an object with the following keys:
  - `operation`: operation to perform on values of pre-defined columns of the same row. available operations:
    - `constant` - add a constant value
    - `sum` - summed value for given columns in a row.
    - `avg` - average value from given columns in a row.
    - `min` - minimum value among given columns in a row.
    - `max` - maximum value among given columns in a row.
    - `multiply` - product of given columns in a row.
    - `join` - joins two or more column values in a row.
    - `format` - Python format string used to form the value Eg:  `my name is {first_name}`.
  - `target` - name of the new field.
  - `source` - list of columns the operations should be performed on (Not required in case of `format` and `constant`).
  - `with` - String passed to `constant`, `format` or `join` operations
    - in `constant` - used as constant value
    - in `format` - used as Python format string with existing column values Eg: `{first_name} {last_name}`
    - in `join` - used as delimiter
- `resources`
  - A name of a resource to operate on
  - A regular expression matching resource names
  - A list of resource names
  - `None` indicates operation should be done on all resources


#### find_replace.py
Look for specific patterns in specific fields and replace them with new data

```python
def find_replace(fields, resources=None):
    pass
```

- `fields`- list of fields to replace values in. Each list item is an object with the following keys:
  - `name` - name of the field to replace value
  - `patterns` - list of patterns to find and replace from field
    - `find` - String, interpreted as a regular expression to match field value
    - `replace` - String, interpreted as a regular expression to replace matched pattern
- `resources`
  - A name of a resource to operate on
  - A regular expression matching resource names
  - A list of resource names
  - `None` indicates operation should be done on all resources

#### set_type.py
Sets a field's data type and type options and validates its data based on its new type definition.

This processor modifies the last resource in the package.

```python
def set_Type(name, **options):
    pass
```

- `name` - the name of the field to modify
- `options` - options to set for the field. Most common ones would be:
  - `type` - set the data type (e.g. `string`, `integer`, `number` etc.)
  - `format` - e.g. for date fields 
  etc.
 (more info on possible options can be found in the [tableschema spec](https://frictionlessdata.io/specs/table-schema/))

### Manipulate the entire resource
#### sort_rows.py
Sort incoming data based on key.

`sort_rows` accepts a list of resources and a key (as a Python format string on row fields).
It will output the rows for each resource, sorted according to the key (in ascending order).


```python
def sort_rows(key, resources=None):
    pass
```

- `key` - String, which would be interpreted as a Python format string used to form the key (e.g. `{<field_name_1>}:{field_name_2}`)
- `resources`
  - A name of a resource to operate on
  - A regular expression matching resource names
  - A list of resource names
  - `None` indicates operation should be done on all resources

#### unpivot.py
Unpivot a table - convert one row with multiple value columns to multiple rows with one value column

```python
def unpivot(unpivot_fields, extra_keys, extra_value, resources=None):
    pass
```

- `unpivot_fields` - List of source field definitions, each definition is an object containing at least these properties:
  - `name` - Either simply the name, or a regular expression matching the name of original field to unpivot.
  - `keys` - A Map between target field name and values for original field
    - Keys should be target field names from `extra_keys`
    - Values may be either simply the constant value to insert, or a regular expression matching the `name`.
- `extra_keys` - List of target field definitions, each definition is an object containing at least these properties (unpivoted column values will go here)
  - `name` - Name of the target field
  - `type` - Type of the target field
- `extra_value` - Target field definition - an object containing at least these properties (unpivoted cell values will go here)
  - `name` - Name of the target field
  - `type` - Type of the target field
- `resources`
  - A name of a resource to operate on
  - A regular expression matching resource names
  - A list of resource names
  - `None` indicates operation should be done on all resources


#### filter_rows.py
Filter rows based on inclusive and exclusive value filters.
`filter_rows` accepts equality and inequality conditions and tests each row in the selected resources. If none of the conditions validate, the row will be discarded.

```python
def filter_rows(equals=tuple(), not_equals=tuple(), resources=None):
    pass
```

- `in` - Mapping of keys to values which translate to `row[key] == value` conditions
- `out` - Mapping of keys to values which translate to `row[key] != value` conditions
- `resources`
  - A name of a resource to operate on
  - A regular expression matching resource names
  - A list of resource names
  - `None` indicates operation should be done on all resources

Both `in` and `out` should be a list of dicts.

### Manipulate package
#### add_metadata.py
Add high-level metadata about your package

```python
def add_metadata(**metadata):
    pass
```

- `metadata` - Any allowed property (according to the [spec]([https://frictionlessdata.io/specs/data-package/#metadata)) can be provided here.

#### concatenate.py
Concatenate multiple streams of data to a single one, resolving differently named columns along the way.

```python
def concatenate(fields, target={}, resources=None):
    pass
```

- `fields` - Mapping of fields between the sources and the target, so that the keys are the _target_ field names, and values are lists of _source_ field names.
  This mapping is used to create the target resources schema.
  Note that the target field name is _always_ assumed to be mapped to itself.
- `target` - Target resource to hold the concatenated data. Should define at least the following properties:
  - `name` - name of the resource
  - `path` - path in the data-package for this file.
  If omitted, the target resource will receive the name `concat` and will be saved at `data/concat.csv` in the datapackage.
- `resources`
  - A name of a resource to operate on
  - A regular expression matching resource names
  - A list of resource names
  - `None` indicates operation should be done on all resources
  Resources to concatenate must appear in consecutive order within the data-package.

#### duplicate.py
Duplicate a single stream of data to make two streams

`duplicate` accepts the name of a single resource in the datapackage. 
It will then duplicate it in the output datapackage, with a different name and path.
The duplicated resource will appear immediately after its original.

```python
def duplicate(source=None, target_name=None, target_path=None):
    pass
```

- `source` - The name of the resource to duplicate. 
- `target_name` - Name of the new, duplicated resource.
- `target_path` - Path for the new, duplicated resource.
