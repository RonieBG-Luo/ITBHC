# ITBHC
Irregular Tensor Binary Hashing Consensus Clustering for Single-Cell Multi-Omics Data
https://github.com/RonieBG-Luo/ITBHC

## Dependencies
Python 3.12 (>= 3.9 supported)
Numpy
Pandas
Scipy
evaluating_clustering.py (bundled; provides ACC / ARI / NMI / FMI)

ITBHC is CPU-only; all experiments in this study are conducted on a laptop
with an Intel Core Ultra 5 125H processor (1.20 GHz) and 32 GB RAM.
We suggest installing the dependencies in a conda environment
(conda create -n itbhc python=3.12; pip install numpy pandas scipy).
Thanks to bitwise-XOR Hamming-space clustering, the runtime scales linearly
with the number of cells: datasets with a few thousand cells finish within
minutes.
  
## Run ITBHC
1. Prepare the input data in CSV format (see "Data format" below).
2. Run ITBHC with the built-in reference configuration:
   python ITBHC.py                          # default dataset: scGEM
   python ITBHC.py --dataset GSE --beta 6 --code-lengths 8,128 --lambdas 0.5,1.0
   python ITBHC.py --custom mod1.csv mod2.csv labels.csv   # your own two-modality data

3. Add --verbose to print per-iteration diagnostics of the alternating optimization.

## Output of ITBHC
ITBHC outputs the binary cluster-assignment matrix Q and the predicted cluster
labels, which can be used for downstream analyses and visualized by t-SNE;
when a label file is provided, it also prints ACC, ARI, NMI, FMI and the runtime.

## Scripts
- **`ITBHC.py`** — clustering stage only. Expects the *standardized low-rank
  slices* (IT-TRPCA output followed by Z-score normalization) under
  `ITTRPCA_dataset_std/`.
- **`ITTRPCA_ITBHC.py`** — end-to-end pipeline. Takes **raw** modality matrices
  (default root `Raw_dataset/`), internally runs the HOGSVD-based irregular
  tensor robust PCA (IT-TRPCA, function `TSN1` in the bundled module
  `ITTRPCA_modeling.py`) to separate the low-rank shared component from sparse
  noise, Z-score standardizes the low-rank slices, and then performs ITBHC
  clustering. It shares the same CLI arguments as `ITBHC.py`:
   python ITTRPCA_ITBHC.py --dataset scGEM --beta 6 --code-lengths 8,8 --lambdas 0.3,1.3

## Arguments
--dataset: name of a built-in dataset (GSE, scGEM, PBMC3K, Sim700, Sim1000).
    Default: GSE. Mutually exclusive with --custom.
--custom: paths of two modality matrices and one label file (header-less CSVs).
--data_root: root folder of per-dataset subfolders. Default: ITTRPCA_dataset_std.
--beta: weight regulator beta. Default: 6.0.
--code-lengths: comma-separated binary code lengths b_v, one per omics layer.
    Default: 8,8. Keep b_v <= m_v; 128 is recommended when dimensionality permits.
--lambdas: comma-separated regularization parameters lambda^v. Default: 0.5,1.2.
--max-iter-out: maximum outer iterations. Default: 100.
--tol: convergence tolerance on the objective. Default: 1e-6.
--seed: random seed. Default: 1.
--verbose: print per-iteration diagnostics. Default: off.

## Data format
CSV, header=None. Inputs are the standardized low-rank slices produced by
IT-TRPCA preprocessing followed by Z-score normalization.
Structure per dataset (features x cells):
ITTRPCA_dataset_std/
└── <dataset>/
    ├── <dataset>_D1.csv     # modality 1 (e.g., RNA)
    ├── <dataset>_D2.csv     # modality 2 (e.g., ADT / ATAC)
    └── <dataset>_labels.csv # ground-truth labels, one per line (also fixes the cluster number c)
Processed datasets for all five benchmarks are available in the repository releases; 
raw-data provenance is described in the manuscript's Data Availability.

The directory structure of `Raw_dataset/` is the same as that of `ITTRPCA_dataset_std/`.
The former stores the raw modality matrices, while the latter stores the standardized low-rank slices processed by IT-TRPCA.

## License
This software is released under a custom Academic and Non-Commercial Use License. 
It is freely available for academic and non-commercial research purposes. 
Commercial use is strictly prohibited without prior written permission. 
Please see the [LICENSE](LICENSE) file for full details.
