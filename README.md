# BiGAnts: network-constrained biclustering of patients and multi-omics data
## Table of contents
* [General info](#general-info)
* [Installation](#installation)
* [Data input](#data-input)
* [Main functions](#main-functions)
* [Example](#example)
* [Cite](#cite)
* [Contact](#contact)


## General info
PyPI package for conjoint clustering of networks and omics data. BiGants allows to conjointly cluster patients and genes such that **(i)** biclusters are restricted to functionally related genes connected in molecular interaction networks and **(ii)**  the expression difference between two subgroups of patients is maximized.


## Installation

To install the package from PyPI please run:

`pip install bigants` 

To install the package from git:

`git clone https://github.com/biomedbigdata/BiGAnts-PyPI-package`

`python setup.py install`


## Data input

The algorithm needs as an input one CSV matrix with gene expression/methylation/any other numerical data and one CSV file with a network.

Please note, that even though the algorithm will accept any IDs, all visualisation tools except entrez genes IDs as an input.

### Numerical data

Numerical data is accepted in the following format:
- genes as rows.
- patients as columns.
- first column - genes IDs (can be any IDs).

For instance:

|   | Unnamed: 0 | GSM748056 | GSM748059 | ... | GSM748278 | GSM748279 | GSM1465989 |
|---|------------|-----------|-----------|-----|-----------|-----------|------------|
| 0 | 1454       | 0.053769  | 0.117412  | ... | -0.392363 | -1.870838 | -1.432554  |
| 1 | 201931     | -0.618279 | 0.278637  | ... | 0.803541  | -0.514947 | 2.361925   |
| 2 | 8761       | 0.215820  | -0.343865 | ... | 0.700430  | 0.073281  | -0.977656  |
| 3 | 2703       | -0.504701 | 1.295049  | ... | 1.861972  | 0.601808  | 0.191013   |
| 4 | 26207      | -0.626415 | -0.646977 | ... | 2.331724  | 2.339122  | -0.100924  |

There are 2 examples of gene expression datasets that can be placed in the "data" folder
- GSE30219 - a Non-Small Cell Lung Cancer dataset from GEO for patients with either adenocarcinoma or squamous cell carcinoma. 
- TCGA pan-cancer dataset with patients that have luminal or basal breast cancer.
Both can be found [here](https://drive.google.com/drive/folders/1J0XRrklwcV_Cgy_9Ay_6yJrN_x28Cosk?usp=sharing)

### Network

An interaction network should be present as a CSV table with two columns that represent two interacting genes. **Without a header!**

For instance:

|   | 6416 | 2318 |
|---|------|------|
| 0 | 6416 | 5371 |
| 1 | 6416 | 351  |
| 2 | 6416 | 409  |
| 3 | 6416 | 5932 |
| 4 | 6416 | 1956 |

There is an example of a PPI network from BioiGRID with experimentally validated interactions [here](https://drive.google.com/drive/folders/1J0XRrklwcV_Cgy_9Ay_6yJrN_x28Cosk?usp=sharing).

## Main functions

Here we give a general description of the main functions provided. Please note, that all functions are annotated with dockstrings and therefore the full information can be found with *help()* method, e.g. `help(results.save)`.

1.**data_preprocessing**(path_expr, path_net, log2 = False, size = 2000)

Parameters:

- path_to_expr: *string*, path to the numerical data 
- path_to_net: *string*, path to the network file
- log2: *bool, (default = False)*, indicates if log2 transformation should be applied to the data 
- size: *int, optional (default = 2000)* determines the number of genes that should be pre-selected by variance for the analysis. Shouldn't be higher than 5000.

Returns:

- GE: *pandas data frame*, processed expression data
- G: *networkX graph*, processed network data
- labels: *dict*, for mapping between real genes/patients IDs and the internal ones
- rev_labels: *dict*, additional dictionary for mapping between real genes/patients IDs and the internal ones

2. *BiGAnts**(GE,G,L_g_min,L_g_max) creates a model for the given data:

Parameters:

- GE: *pandas dataframe*, processed expression data
- G: *networkX graph*, processed network data
- L_g_min: *int*, minimal solution subnetwork size
- L_g_max: *int*, maximal solution subnetwork size

Methods:

BiGAnts.**run**(self, n_proc = 1, K = 20, evaporation = 0.5, show_plot = False)

- K: *int, default = 20*, number of ants. Fewer ants - less space exploration. Usually set between 20 and 50      
- n_proc: *int, default = 1*, number of processes that should be used(can not be more than K)
- evaporation, *float, default = 0.5*, the rate at which pheromone evaporates
- show_plot: *bool, default = False*, set true if convergence plots should be shown during the iterations

## Example

Import the package:

```python
from bigants import data_preprocessing
from bigants import BiGAnts
from bigants import results_analysis
```
Set the paths to the expression matrix and the PPI network:

```python
path_expr,path_net ='/data/gse30219_lung.csv', '/data/biogrid.human.entrez.tsv'
```
Load and process the data:

```python
GE,G,labels, _= data_preprocessing(path_expr, path_net)
```
Set the size of subnetworks:
```python
L_g_min = 10
L_g_max = 15
```
Set the model and run the search:

```python
model = BiGAnts(GE,G,L_g_min,L_g_max)
solution,sc= model.run_search()
```
## Results analysis
BiGAnts package also allows a user to save the results and perform an initial analysis. 
The examples below show the basic usage, for more details please use python help() method, e.g. `help(results.save)`.

1. First, the object for results analysis must be created:
```python
results = results_analysis(solution, labels)
```
This will allow to easily access the resulting biclusters and their initial IDs as well as perform a more complicated analysis.

To access IDs of patients in the first bicluster:
```python
results.patients1
```
To access IDs of genes IDs in the first bicluster:
```python
results.genes1
```
Same logic applies to the second bicluster.

2. To save the solution:
```python
#with the initial IDs
results.save(output = "results/results.csv")

#with gene names
results.save(output = "results/results.csv", gene_names = True) 
```
3. Visualise the resulting networks colored with respect to their difference in expression patterns in patients clusters:
```python
results.show_networks(GE, G, output = "results/network.png")
```
4. Visualise a clustermap of the achieved solution alone or also along with the known patients' groups.
Just with the BiGAnts results:

```python
results.show_clustermap(GE, G, solution, labels, output = "results/clustermap.png")
```
If you have a patient's phenotype you would like to use for comparison, please make sure that patients IDs are exactly (!) matching the IDs that were used as an input. The IDs should be represented as a list of two lists, e.g.:

```python
true_classes = ['GSM748056', 'GSM748059',..], ['GSM748278', 'GSM748279', 'GSM1465989']
results.show_clustermap(GE, G, solution, labels, output = "results/clustermap.png", true_labels = true_classes)
```
5. Given a known phenotype in a format described above, BiGAnts can also return Jaccard index of the achieved patients clustering with a given phenotype:

```python
results.jaccard_index(true_labels = true_classes)
```
6. BiGAnts is using [gseapy](https://gseapy.readthedocs.io/en/master/index.html) module to provide a user with a python wrapper for [Enrichr](https://amp.pharm.mssm.edu/Enrichr/) database. 

```python
results.enrichment_analysis(solution, labels, library = 'GO_Biological_Process_2018', "results")
```
After the execution of the given above code, in the */results* directory a user can find a table with enriched pathways as well as enrichment plots. Other available libraries can be used as well, e.g. 'GO_Molecular_Function_2018' and 'GO_Cellular_Component_2018'. In total there are 159 libraries available at the moment and the full list can be found by typing:

```python
import gseapy
gseapy.get_library_name()
```


## Cite
If you use BiGAnts in your research, we kindly ask you to cite the following publication:

`Citation details to be announced` 


## Contact

If you want to contact us regarding BiGAnts, please write an email to [Olga Lazareva](mailto:olga.lazareva@wzw.tum.de?subject=[GitHub]%20BiGAnts%20PyPI).
