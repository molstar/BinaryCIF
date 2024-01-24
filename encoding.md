BinaryCIF Format
================

## Data Layout

A [CIF file](http://www.iucr.org/resources/cif/spec/version1.1/cifsyntax) ([example](https://www.ebi.ac.uk/pdbe/static/entry/1tqn_updated.cif)) contains:

* One or more data blocks
* Each data block has one or more category
* Each category has one or more field
* Each field contains *data*

To represent this hierarchy, the basic shape of the BinaryCIF file defines the following 
interfaces:

```
File {
    version: string
    encoder: string
    dataBlocks: DataBlock[]
}

DataBlock {
    header: string
    categories: Category[]
}

Category {
    name: string
    rowCount: number
    columns: Column[]
}

Column {
    name: string
    data: Data
    mask: Data
}

Data {
    data: Uint8Array
    encoding: Encoding[]
}
```

The most interesting part is the ``Data`` interface where the actual data is stored. 
The interface has two properties: ``data`` which is just an array of bytes (the binary data) 
and an array of *encodings* that describes the transformations that were applied to the 
source data to obtain the final binary result stored in the ``data`` field.

Additionally, the ``Column`` interface defines a ``mask`` property used to determine
if a certain value is present, not present (``.`` token in CIF), or unknown (``?`` token in CIF). 

Currently, BinaryCIF supports these encoding methods:

```
type Encoding = 
    | ByteArray 
    | FixedPoint
    | IntervalQuantization 
    | RunLength 
    | Delta 
    | IntegerPacking 
    | StringArray
``` 

## Encoding Methods

### Byte Array

Encodes an array of numbers of specified types as raw bytes.

```
ByteArray {
    kind = "ByteArray"
    type: Int8 | Int16 | Int32 | Uint8 | Uint16 | Uint32 | Float32 | Float64
}
```

### Fixed Point

Converts an array of floating point numbers to an array of 32-bit integers multiplied 
by a given factor. 

```
FixedPoint {
    kind = "FixedPoint"
    factor: number
    srcType: Float32 | Float64
}
```

#### Example

```
[1.2, 1.23, 0.123] 
---FixedPoint---> 
{ factor = 100 } [120, 123, 12] 
``` 

### Interval Quantization

Converts an array of floating point numbers to an array of 32-bit integers where 
the values are quantized within a given interval into specified number of
discrete steps. Values lower than the minimum value or greater than the 
maximum are reprented by the respective boundary values.

```
IntervalQuantization {
    kind = "IntervalQuantization"
    min: number,
    max: number,
    numSteps: number,
    srcType: Float32 | Float64
}
```

#### Example

```
[0.5, 1, 1.5, 2, 3, 1.345 ] 
---IntervalQuantization---> 
{ min = 1, max = 2, numSteps = 3 } [0, 0, 1, 2, 2, 1] 
``` 

### Run Length

Represents each integer value in the input as a pair of ``(value, number of repeats)``
and stores the result sequentially as an array of 32-bit integers. Additionally,
stores the size of the original array to make decoding easier.

```
RunLength {
    kind = "RunLength"
    srcType: int[]
    srcSize: number
}
```

#### Example

```
[1, 1, 1, 2, 3, 3] 
---RunLength---> 
{ srcSize = 6 } [1, 3, 2, 1, 3, 2]
```

### Delta

Stores the input integer array as an array of consecutive differences. 

```
Delta {
    kind = "Delta"
    origin: number
    srcType: int[]
}
```

Because delta encoding is often used in conjuction with integer packing,
the ``origin`` property is present. This is to optimize the case
where the first value is large, but the differences are small. 

#### Example

```
[1000, 1003, 1005, 1006] 
---Delta---> 
{ origin = 1000, srcType = Int32 } [0, 3, 2, 1]
```

### Integer Packing

Stores a 32-bit integer array using 8- or 16-bit values. Includes the size 
of the input array for easier decoding. The encoding is more effective 
when only unsigned values are privided. 

```
IntegerPacking {
    kind = "IntegerPacking"
    byteCount: number
    srcSize: number
    isUnsigned: boolean
}
```

#### Example 

```
[1, 2, -3, 128] 
---IntgerPacking---> 
{ byteCount = 1, srcSize = 4, isUnsigned = false } [1, 2, -3, 127, 1]
```

### String Array

Stores an array of strings as a concatenation of all unique strings, an array of offsets
describing substrings, and indices into the offset array. 
indices to corresponding substrings.

```
StringArray {
    kind = "StringArray"
    dataEncoding: Encoding[]
    stringData: string
    offsetEncoding: Encoding[]
    offsets: Uint8Array
}
```

#### Example

```
['a','AB','a'] 
---StringArray---> 
{ stringData = 'aAB', offsets = [0, 1, 3] } [0, 1, 0]
```

Encoding Process
----------------

To encode the data, a sequence of encoding transformations needs to be specified
for each input column. For example, to encode the ``_atoms.id`` column
from the background section, we could specify the encoding as ``[Delta, RunLength, IntegerPacking]``:

```
[1, 2, 3, 4]
---Delta--->
{ srcType = Int32 } [1, 1, 1, 1]
---RunLength--->
{ srcSize = 4 } [1, 4]
---IntegerPacking--->
{ byteCount = 1, srcSize = 2 } [1, 4]
```

**Little endian** is used everywhere to encode the data.

Once each column has been encoded and the ``File`` data structure built, the 
[MessagePack](https://msgpack.org/) format (which is more or less a binary encoding of the standard
JSON format) is used to produce the final binary result. 

Optionally, the data can be compressed using standard methods such as Gzip to achieve
further compression.

Decoding Process
----------------

To decode the BinaryCIF data, first the MessagePack data are decoded and then 
for each column, the binary data are decoded applying inverses of the transformations
specified in the ``encoding`` array backwards. So to decode the encoding specified by
``[Delta, RunLength, IntegerPacking]`` we would first apply the decoding
of ``IntegerPacking``, then ``RunLength``, and finally ``Delta``.
