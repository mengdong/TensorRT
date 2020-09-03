# Surgeon

## Table of Contents

- [Introduction](#introduction)
- [Subtools](#subtools)
- [Usage](#usage)
- [Examples](#examples)
- [\[EXPERIMENTAL\] JSON Configuration Format](#experimental-json-configuration-format)


## Introduction

The `surgeon` tool uses [ONNX-GraphSurgeon](https://github.com/NVIDIA/TensorRT/tree/master/tools/onnx-graphsurgeon)
to modify an ONNX model.

Although this tool can only operate on and export ONNX models, it will automatically
attempt to convert supported formats to ONNX.


## Subtools

`surgeon` provides subtools to perform different functions:

- `extract` can extract subgraphs from models. It can also be used for changing the shapes/datatypes of the
    model inputs/outputs by specifying the existing inputs/outputs, but with different shapes/datatypes. This
    can be useful to make an input dimension dynamic, for example.

- [EXPERIMENTAL] `prepare` and `operate` work together to perform a 2-stage process:
    1. `prepare` generates an editable JSON configuration file for a given model.
    2. `operate` accepts the model and applies modifications according to a JSON configuration file.

    For details on how to edit models using the JSON configuration file, see [Configuration Format](#configuration-format)

    **NOTE:** `prepare` and `operate` are best suited for interactive cases, such as debugging a single model. Configuration files are
    model-specific, and cannot be shared across models. In cases where you need to apply a set of transformations to
    multiple models, you should use the [ONNX-GraphSurgeon](https://github.com/NVIDIA/TensorRT/tree/master/tools/onnx-graphsurgeon)
    Python API directly.


## Usage

See `polygraphy surgeon -h` for usage information.


## Examples

For examples, see [this directory](../../../examples/cli/surgeon)


## **[EXPERIMENTAL]** JSON Configuration Format

The JSON configuration file generated by the `prepare` tool includes various fields that can be modified.

**NOTE:** Tensor names can be changed, but this will cause any existing metadata to be omitted from the tensor.
Generally, there is no need to change tensor names, so this should hardly ever be an issue.

An example configuration file is reproduced below:
```json
{
    "graph_inputs": [
        {
            "name": "x",
            "dtype": "float32",
            "shape": [
                1,
                1,
                2,
                2
            ]
        }
    ],
    "graph_outputs": [
        {
            "name": "y",
            "dtype": "float32",
            "shape": [
                1,
                1,
                2,
                2
            ]
        }
    ],
    "nodes": [
        {
            "id": 0,
            "name": "",
            "op": "Identity",
            "inputs": [
                "x"
            ],
            "outputs": [
                "y"
            ]
        }
    ]
}
```

### Fields

- `dtype`: Any string that maps to a numpy data type, such as `float16`, `int8`, `bool`, etc.
- `shape`: A list of integers. `-1` can be used to indicate a dynamic dimension

### Graph Inputs

The data types and shapes of the graph inputs can be freely modified. For graph inputs, shapes must be provided.

### Graph Outputs

Similar to graph inputs, graph output data types and shapes can be freely modified.
Unlike with graph inputs, shapes are not required for outputs, since they can be inferred
at runtime based on the input shapes.

### Nodes

**IMPORTANT:** You should *not* modify the `id` parameter of nodes.

The `name`, `op`, `inputs`, and `outputs` fields can be freely modified. This provides a convenient way to reconnect
nodes in the graph and change their function.

**NOTE:** Modiyfing node attributes is not supported at this time.