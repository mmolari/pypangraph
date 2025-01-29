# PyPangraph

PyPangraph is a python library to load, explore and analyze graphs produced by [Pangraph](https://github.com/neherlab/pangraph).

## installation

PyPangraph can be installed via pip:
```bash
pip install pypangraph
```

## quick usage

Loading a graph:
```python
import pypangraph as pp

graph = pp.Pangraph.from_json("path/to/graph.json")
```

TODO: add more examples

More examples can be found in the documentation.

## development

Install the package in dev mode with:

```bash
pip install -e .
```