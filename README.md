# styx :ocean:
ETL/ELT declarative mapping syntax using TOML

Version 0.2.0 - *This is a schema in active development*

## Introduction

styx is aiming towards becoming a language agnostic ETL/ELT syntax to map from one structure to another structure.

| From Structure | To Structure |
| -------------- | ------------ |
| JSON {} | JSON {} |

## Definition

styx is all valid TOML made up a header and 3 sections. Each file is called a Definition. It defines how a mapping should occur unidirectionally.

### Sections

| Sections | Required |
| -------- | -------- |
| header | :heavy_check_mark: |
| preprocess |   |
| fields | :heavy_check_mark: |
| postprocess |  |


## Header

The header  consists of two top-level required keys: `from_type` and `to_type`. 

Example:
```toml
# mythical_creature.styx

from_type = "mythical_creature"
to_type = "DivineCreature
```

## Preprocess

Preprocess is for prepping data before processing the fields to map.

Begins with the section header: `preprocess`

```toml
[preprocess]  # Order matters for preprocess
```

Preprocess is then made up of subsections that will be sorted alphanumerically, and invoked in that order. So
we recommend prepending a numeral to the front of the section header to ensure proper processing (espcially
for dependent operations):

#### Preprocess Subsection Fields
| Field | Required | Description |
| -------- | -------- | -------- |
| input_paths | :heavy_check_mark: | List of `Path`s to *get* data from the *From Structure*. Note: these will be passed to the function as arguments in the order that they are listed. |
| output_path | :heavy_check_mark: | `Path` to *set* data from the *To Structure* |
| function | :heavy_check_mark: | `Function` to invoke on `input_paths` before setting in `output_path`
| or_else |  | Optional value to *set* in `output_path` if value not found in `input_path`  |
| on_throw |  | Action to perform if exception is thrown during `function`. Valid values are: `or_else`, `throw`, `skip` |

#### Example
```toml
[preprocess]  

    [preprocess.01-action]
    path = ["fields.olympians"]
    function = "parse_json"  # See Functions below
    or_else = {}
    on_throw = "throw"
```

## Fields

Fields is where the bulk of the action takes place. It is a declaration of how to unidirectionally map from one structure to another.

Begins with the section header: `fields`

```toml
[fields]
```

Fields is then made up of subsections that correspond to the names of the fields in the `To Structure`.

#### A Simple Example
```toml
[fields]  

    [fields.title]
    input_paths = ["fields.olympian.title"] 

```

#### Fields Subsection Fields

The table header will corrrespond to the output path in the `To Structure`

```toml
    [fields.title]  # 'title' will be the name of the field in the `To Structure`
```

**Available Fields**

| Field | Required | Conditionally Required | Description |
| -------- | -------- | -------- | ---------- |
| input_paths | | :heavy_check_mark:  | List of `Path`s to *get* data from the *From Structure*. Note: these will be passed to the function as arguments in the order that they are listed, if a function is provided |
| possible_paths | | :heavy_check_mark:  | List of potential `Path`s to *get* data from the *From Structure*. *path_condition* will be used to determine which *input_path* to use. Currently does not support multi-argument functions, so please preprocess to prepare data instead.
| path_condition | | :heavy_check_mark:  | Object used to determine the correct input path in *possible_paths*. Must be defined if *possible_paths" is defined. |
| type | | | Styx Definition to use to map this value |
| function | | | `Function` to invoke on `input_paths` before setting in `output_path` |
| or_else |  | :heavy_check_mark: | Optional value to *set* in `output_path` if value not found in `input_path`  |
| on_throw |  | | Action to perform if exception is thrown during `function`. Valid values are: `or_else`, `throw`, `skip` |

**Notes**

Either `input_paths` or `possible_paths` must be defined, but not both.

**Other actions**
Data can be copied to nested objects:

Syntax:

| FIELD_NAME | NESTED_FIELD_NAME | PATH | 
| ---------- | ----------------- | ---- |
| "olympian"   | "title"             |  "fields.olympian_title" |

#### Example

```toml
[fields]
    [fields.olympian]
        type = "Olympian"
        # To Structure = From Structure Path
        olympian.title = "fields.olympian_title"  
        # The data in "fields.olympian_title" will be copied to the nested object "olympian" in the field "title" 
        # before processing with the Olympian Styx definition
```
## Postprocess
Postprocessing structure is the same as preprocessing, but it is to be performed after the `fields` section has been processed.

Begins with the section header: `ppostprocessing`

```toml
[postprocess]  # Order matters here as well
```

Postprocess is then made up of subsections that will be sorted alphanumerically, and invoked in that order. So
we recommend prepending a numeral to the front of the section header to ensure proper processing (espcially
for dependent operations):

#### Postprocess Subsection Fields
| Field | Required | Description |
| -------- | -------- | -------- |
| input_paths | :heavy_check_mark: | List of `Path`s to *get* data from the *From Structure*. Note: these will be passed to the function as arguments in the order that they are listed. |
| output_path | :heavy_check_mark: | `Path` to *set* data from the *To Structure* |
| function | :heavy_check_mark: | `Function` to invoke on `input_path` before setting in `output_path`
| or_else |  | Optional value to *set* in `output_path` if value not found in `input_path`  |
| on_throw |  | Action to perform if exception is thrown during `function`. Valid values are: `or_else`, `throw`, `skip` |

#### Example
```toml
[postprocess]  

    [preprocess.01-action]
    path = ["fields.olympians"]
    function = "parse_json"  # See Functions below
    or_else = {}
    on_throw = "skip"
```

## Functions

Functions are transformers or filters available to perform actions on data. These are declared in a `functions.styx` file. *Implementation is left up to the user or library.*

For example, you could have a `functions.styx` file with this declaration:
```toml
functions = [
  "parse_json",
  "stringify_json",
  "to_camel_case",
  "to_snake_case"
]
```

When the Definition is parsed, it will check your `functions.styx` file to ensure that these functions exist.

TODO: Expand and add more type definitions to `functions.styx`.

## Other

### `const` in input_paths

The special keyword `const` can be used in any `input_paths` to provide a constant value. This value could then be passed to any functions. For example, if you wanted to concatenate two fields with a comma:

```toml
input_paths = ["fields.first_name", "const(',')", "fields.last_name"]
function = "concat"
```
