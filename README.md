# styx
ETL/ELT declarative mapping syntax using TOML

## Introduction

styx is aiming towards becoming a language agnostic ETL/ELT syntax to map from one structure to another structure.

| From Structure | To Structure |
| -------------- | ------------ |
| JSON {} | JSON {} |

## Definition

styx is all valid TOML made up a header and 3 sections.

### Sections

| Sections | Required |
| -------- | -------- |
| header | :heavy_check_mark: |
| preprocessing |   |
| fields | :heavy_check_mark: |
| postprocessing |  |



### Preprocessing

Begins with the section header: `preprocessing`

```toml
[preprocess]  # Order matters for preprocessing
```

Preprocessing is then made up of subsections that will be sorted alphanumerically, and invoked in that order. So
we recommend prepending a numeral to the front of the section header to ensure proper processing (espcially
for dependent operations):

#### Preprocessing Subsection Fields
| Field | Required | Description |
| -------- | -------- | -------- |
| input_path | :heavy_check_mark: | `Path` to *get* data from the *From Structure* |
| output_path | :heavy_check_mark: | `Path` to *set* data from the *To Structure* |
| function | :heavy_check_mark: | `Function` to invoke on `input_path` before setting in `output_path`
| or_else |  | Optional value to *set* in `output_path` if value not found in `input_path`  |
| on_throw |  | Action to perform if exception is thrown during `transform`. Valid values are: `or_else`, `throw`, `skip` |

#### Example
```toml
[preprocess]  

    [preprocess.01-action]
    path = "fields.custentity_lead_geocodes"
    transform = "parseJson"
    or_else = {}
    on_throw = "throw"
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
