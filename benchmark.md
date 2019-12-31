Benchmark
=========

The BinaryCIF format has been applied to encode macromolecular data stored using [mmCIF](http://mmcif.wwpdb.org/) 
data format (mmCIF is a schema of categories and fields that desribe a macromolecular structure
stored using the CIF format). The raw data for the benchmark are included in this repository.

- The "CoordinateServer" results were obtained using the corresponding query of the [CoordinateServer](https://webchemdev.ncbr.muni.cz/CoordinateServer).
- The "MMTF" results were obtained using [MMTF](https://mmtf.rcsb.org) version 1.0.

## HIV-1 Capsid size

Encoding the currenly largest structure in the PDB.org archive, the [HIV-1 Capsid](https://pdbe.org/3j3q)
with 2.44M atoms, BinaryCIF achieves very good results. 

![3j3q size](img/bench-3j3q.png)


## Whole PDB archive size

- (*) RCSB PDB: 122333 Entries, some 404'ed; recompressed using the same compression level as the other data.
- (**) reduced = alpha + phosphate trace + HET


![PDB Size](img/bench-pdb.png)

## Read and write performance

This is the performance of BinaryCIF implementation of the [CIFTools.js library](https://github.com/dsehnal/CIFTools.js),
[LiteMol](https://github.com/dsehnal/LiteMol), and the [CoordinateServer](https://webchemdev.ncbr.muni.cz/CoordinateServer).

![Parse](img/bench-perf-parse.png)

![Write](img/bench-perf-read.png)

