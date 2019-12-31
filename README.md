![Version 0.3.0](https://img.shields.io/badge/Version-0.3.0-blue.svg?style=flat)

![BinaryCIF](img/logo.png)

BinaryCIF is a data format that stores text based CIF files using 
a more efficient binary encoding. It enables both lossless and lossy
compression of the original CIF file. BinaryCIF is currently mainly used by RCSB PDB and PDBe and is supported by the [Mol*](https://github.com/molstar/molstar) and [LiteMol](https://github.com/dsehnal/LiteMol) viewers.

Some aspects of the BinaryCIF format, namely using [MessagePack](https://msgpack.org/) as the container
and the usage the fixed point, run length, delta, and integer packing encodings was
inspired by the [MMTF data format](http://mmtf.rcsb.org).

Table of contents
=================

* [Implementations](#implementations)
* [Principles](#principles)
* [Use Cases](#use-cases)
    - [CoordinateServer](#coordinateserver)
    - [DensityServer](#densityserver)

Implementations
=================

BinaryCIF is currently available as TypeScript (JavaScript) and Java.

- TypeScript implementation is part of the [Mol* project](https://github.com/molstar/molstar) (the original implementation of the BinaryCIF format is the [CIFTools.js library](https://github.com/dsehnal/CIFTools.js)).
- [Mol* ciftools](https://github.com/molstar/ciftools) are available as a standalone library/tools for conversion of text CIF to BinaryCIF.
- Java implementation is available at [rcsb/ciftools-java](https://github.com/rcsb/ciftools-java).

Principles
==========

* [Basic Principles](principle)
* [BinaryCIF Format](encoding)
* [Benchmark](benchmark)

Use Cases
=========

## CoordinateServer

BinaryCIF is supported by the [CoordinateServer](https://cs.litemol.org), a web service for 
delivering subsets of 3D macromolecular data stored in the [mmCIF format](http://mmcif.wwpdb.org/).

The server can return data both in the text and binary version of the CIF format,
with the binary representation being a lot more efficient (see the [benchmark](#benchmark)).

## DensityServer

BinaryCIF is supported by the [DensityServer](https://ds.litemol.org), a web service for accessing subsets of volumetric density data, that automatically downsamples the data depending on the volume of the requested region to reduce the bandwidth requirements and provide near-instant access to even the largest data sets. 

-------------------

## Contributing
Just open an issue or make a pull request. All contributions are welcome.

## Funding

Funding sources include but are not limited to:
* [RCSB PDB](https://www.rcsb.org) funding by a grant [DBI-1338415; PI: SK Burley] from the NSF, the NIH, and the US DoE
* [PDBe, EMBL-EBI](https://pdbe.org)
* [CEITEC](https://www.ceitec.eu/)