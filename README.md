# BiGAnts: network-constrained biclustering of patients and multi-omics data

## Data input

The algorithm needs as an input one csv matrix with gene expression/methylation/any other numerical data and one tsv file with a network.

### Numerical data

Numerical data is accepted in the following format:
- genes as rows.
- patients as columns.
- first column - genes ids (can be any ids).

For instance:

|   | Unnamed: 0 | GSM748056 | GSM748059 | ... | GSM748278 | GSM748279 | GSM1465989 |
|---|------------|-----------|-----------|-----|-----------|-----------|------------|
| 0 | 1454       | 0.053769  | 0.117412  | ... | -0.392363 | -1.870838 | -1.432554  |
| 1 | 201931     | -0.618279 | 0.278637  | ... | 0.803541  | -0.514947 | 2.361925   |
| 2 | 8761       | 0.215820  | -0.343865 | ... | 0.700430  | 0.073281  | -0.977656  |
| 3 | 2703       | -0.504701 | 1.295049  | ... | 1.861972  | 0.601808  | 0.191013   |
| 4 | 26207      | -0.626415 | -0.646977 | ... | 2.331724  | 2.339122  | -0.100924  |

### Network

An interaction network should be present as a tsv table with two columns that represent two interacting genes. **Without a header!**

For instance:

|   | 6416 | 2318 |
|---|------|------|
| 0 | 6416 | 5371 |
| 1 | 6416 | 351  |
| 2 | 6416 | 409  |
| 3 | 6416 | 5932 |
| 4 | 6416 | 1956 |

## To run the application

The algorithm takes as an imput the following arguments:
- path_to_expr - path to the numerical data 
- path_to_net - path to the network file
- L_min - minimum number of genes in a bicluster
- L_max - maximal number of genes in a bicluster

There are 2 example of gene expression datasets in the folder "data"
- GSE30219 - a Non Small Cell Lung Cancer dataset from GEO for patients with either adenocarcinoma or squamous cell carcinome. 
- TCGA pan cancer dataset with patients that have luminal or basal breast cancer.
And one example of PPI network from Bioigrid with experementally validated interactions.

To run the algorithm with NSCLC dataset and small subnetworks (between 10 and 15 genes), run the following command:
` python script_main.py 'data/gse30219.csv' 'data/biogrid.human.entrez.tsv' 10 15`

### Dependencies

- pandas
- numpy
- networkx
- multiprocessing
- matplotlib
- sklearn
- scipy




# BiGAnts-PyPI-package
