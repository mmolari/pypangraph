# pypangraph: python utilities to interact with PanGraph's pangenome graphs

This repository contains a collection of utilities to interact with pangrenome graphs produced by [PanGraph](https://github.com/neherlab/pangraph).

The package can be installed using pip with the command:

```bash
pip install git+https://github.com/mmolari/pypangraph
```

Below are some examples showcasing some of the main functions in the package.

## Loading and interacting with pangraph objects:

A pangraph object can be loaded from a json file with:

```python
# load the library
import pypangraph as pp
# this returns a pangraph object
pan = pp.Pangraph.load_json("path/to/pangraph.json")
```

Pangraph objects have two main properties: `blocks` and `paths`. One can cycle through blocks and paths simply with:

```python
len(pan.paths) # n. paths, i.e. number of genomes in the pangraph
for p in pan.paths:
  print(p)

len(pan.blocks) # n. blocks in the pangraph
for b in pan.blocks:
  print(b)
```

blocks and paths can also be accessed with their id:

```python
# access the path of strain NZ_CB0023  
p = pan.paths['NZ_CB0023']

# access the block having block-id XIFBDIEN
b = pan.blocks['XIFBDIEN']
```

### Other pangraph representation

Some functions allow one to have a simpler representation for the pangraph. For example the function `to_paths_dict()` will return a dictionary whose keys are strain ids, and whose items are the corresponding path in the form of a list of block-ids:

```python
pd = pan.to_paths_dict()
# this returns a dictionary with keys -> strains and items -> list of block ids
# {'NZ_CP017934': ['PEOPLTUHUH', 'YVRAYWKHNM', 'KVDFPWBTPZ', ... ],
#  'NZ_CP018695': ['LSMCKQEJBH', 'GYLGWAYIRT', 'ESLHMTQRTH', ... ],
#  ... }
```

The function `to_blockcount_df()` returns a pandas dataframe whose indices are strain ids, and whose columns are block ids. The entries are the number of times a block is present in a given strain. It is the equivalent of a gene presence/absence matrix, but for blocks.

```python
bc = pan.to_blockcount_df()
# |             |   PEOPLTUHUH |   YVRAYWKHNM |   KVDFPWBTPZ |   OEPQASJFSS | ...
# |:------------|-------------:|-------------:|-------------:|-------------:|
# | NZ_CP017934 |            1 |            2 |            1 |            1 |
# | NZ_CP018695 |            1 |            1 |            1 |            0 |
# | NZ_LN824133 |            1 |            2 |            0 |            0 |
# ...
```

The function `to_blockstats_df()` returns a pandas dataframe containing summary statistics on the blocks. The indices of the dataframe are columns, and the columns are `count` (total number of block occurrences), `strains` (number of strains that possess the block), `len` (block consensus sequence length), `duplicated` (boolean, whether the block is duplicated), `core` (whether the block is a core block, i.e. not duplicated and present in all strains).

```python
bs = pan.to_blockstats_df()
# |            |   count |   n. strains |   len | duplicated   | core   |
# |:-----------|--------:|-------------:|------:|:-------------|:-------|
# | PEOPLTUHUH |     103 |          103 |   114 | False        | False  |
# | YVRAYWKHNM |     103 |          103 |  5540 | False        | False  |
# | KVDFPWBTPZ |      97 |           97 |  1609 | False        | False  |
# ...
```


Finally, the function `to_networkx()` returns a representation of the pangraph as a `networkx` graph object. This is convenient also for visualizing parts of the graph.

```python
G = pan.to_networkx()
```

### Block object

Block object have these properties/methods:
```python
bl = pan.blocks['YVRAYWKHNM']
bl.depth() # n. of copies of the block
bl.frequency() # n. of unique genomes that have the block
bl.id # unique id of the block
bl.sequence # consensus sequence of the block
bl.strains() # list of unique strains that have the block 
bl.is_duplicated() # whether the block is duplicated
```

#### Block alignment

Each block also has an alignment property, that can be used to reconstruct the block sequences and alignment:
```python
aln = bl.alignment
alignment, occurrences = aln.generate_alignments()
# generate the alignment for the block.
# returns two objects: the aligned list of sequences
# and the list of identifiers for the sequences (strain, n. occurrences, dna strand)
  
alignment, occurrences = aln.generate_sequences()
# same as above, but instead of alignments returns the sequences, without gaps.
```

### Path object


```python
pt = pan.paths['NZ_CP014647']
pt.block_ids # ordered list of block ids composing the paths
pt.block_nums # ordered list with number of occurrence of each block.
# (duplicated blocks have occurrence n.1, n.2... on each strain)
pt.block_strands # ordered list indicating whether each block is on the direct or reverse dna strand
pt.name # name of the strain/genome
```

## Finding blocks corresponding to locations in the genome

If one has a position or an interval on a particular genome, and wants to find to which pangraph blocks this interval corresponds to, one can instantiate a `Locator` object. This object can then be used to find these blocks.

Nb: positions must be given with julia indexing convention. Numbering starts from 1 and extremes are included.

```python
# instantiate a locator object for the pangraph
loc = pp.Locator(pan)

# returns the block corresponding to a given position in the genome (bl_id).
# It also returns the position of the nucleotide in the block (n.b. relative to block
# occurrence, not consensus!) and the block tag (strain, block n., strand) identifying
# the particular block occurrence. All positions are in 1-based indexing.
bl_id, bl_pos, occ = loc.find_position(strain='NZ_CP014647', pos=4231563)

# finds the list of blocks corresponding to the interval between pos_b and pos_e.
# It returns this list (bl_ids) along with a list of intervals (beg_pos, end_pos)
# corresponding to the coordinates of the interval in the block frame of reference.
# Finally, it returns the corresponding list of block occurrence tags (strain, 
# block n., strand).
# Nb: all positions are considered in 1-based indexing. 
bl_ids, intervals, occs = loc.find_interval(strain='NZ_CP014647', pos_b=5034, pos_e=7028)
```