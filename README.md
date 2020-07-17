# styx
ETL/ELT declarative mapping syntax using TOML

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


### Header

For now, the header only consists of a top-level key `type` to declare the `type` of the definition

Example:
```toml
# mythical_creature.styx

type = "MythicalCreature"
```

### Preprocess

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
| function | :heavy_check_mark: | `Function` to invoke on `input_path` before setting in `output_path`
| or_else |  | Optional value to *set* in `output_path` if value not found in `input_path`  |
| on_throw |  | Action to perform if exception is thrown during `transform`. Valid values are: `or_else`, `throw`, `skip` |

#### Example
```toml
[preprocess]  

    [preprocess.01-action]
    path = ["fields.olympians"]
    function = "parse_json"  # See Functions below
    or_else = {}
    on_throw = "throw"
```


### Postprocess
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
| on_throw |  | Action to perform if exception is thrown during `transform`. Valid values are: `or_else`, `throw`, `skip` |

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
