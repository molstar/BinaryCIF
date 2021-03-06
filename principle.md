Basic Principles
================

In this chapter the basic ideas behind the BinaryCIF will are discussed. 

[CIF](http://www.iucr.org/resources/cif/spec/version1.1/cifsyntax) is a text based format for storing tabular data. 
The data is stored row by row using this syntax:

```
loop_
_category.field1
_category.field2
...
_category.fieldK
value-1_1 value-1_2 ... value-1_K
...
value-N_1 value-N_2 ... value-N_K
```

For example, the table called ``atoms`` with columns ``type, id, element, x, y, z``

|type|id|element|x|y|z|
|:---:|---:|:---:|---:|---:|---:|
|ATOM|1|C|0|0|0|
|ATOM|2|C|1|0|0|
|ATOM|3|O|0|1|0|
|HETATM|4|Fe|0|0|1|

would be stored in CIF as 

```
loop_
_atoms.type
_atoms.id
_atoms.element
_atoms.x
_atoms.y
_atoms.z
ATOM 1 C 0 0 0
ATOM 2 C 1 0 0
ATOM 3 O 0 1 0
HETATM 4 Fe 0 0 1
```

If we want to compress the rows using a dictionary compression, it would identify 
the string ATOM as a repeated substring and represent the rows something along the lines of

```
A = ATOM

{A} 1 C 0 0 0
{A} 2 C 1 0 0
{A} 3 O 0 1 0
HETATM 4 Fe 0 0 1 
```

where ``{A}`` is a dictionary reference to the string ``ATOM``. At first, it would seem 
that this is an efficient solution. However, the problem with this data representation is that 
it is actually hard to compress because related data is not next to each other. 

Fortunately, we can do much better than this: we can transpose the tabular data 
and store them *per column* instead of *per row*:

```
_atoms.type:    ATOM ATOM ATOM HETATM
_atoms.id:      1 2 3 4
_atoms.element: C C O Fe
_atoms.x        0 1 0 0
_atoms.y        0 0 1 0
_atoms.z        0 0 0 1
```

Now, we can compress all the repeating ATOM values using a method called run-length encoding:

```
_atoms.type: {ATOM, 3} HETATM
```

Where ``{ATOM, 3}`` means *repeat the string* ``ATOM`` *3 times*. If the value ATOM repeats 
1 million times (which is quite common), this approach saves us a lot of space.

Similarly, we can apply different encoding schemes to other types of data. 
For example, the sequence

```
1 2 3 4 5 ... n
``` 

can be encoded using delta encoding as 

```
1 1 1 1 1 ...
```

meaning we start with 1, then add 1 to the previous value, ending up with 2, then add 1 to the
previous values as well getting 3, etc. At this point, we can use the run-length encoding
approach from the ATOM example and end up with 

```
{1, n}
```

to represent the original sequence of integers from 1 to n.

The final step is to use binary instead of text encoding to store our data to make it more 
space efficient. For example, storing the number 1234 as text requires 4 bytes:

```
"1"  "2"  "3"  "4"

0x31 0x32 0x33 0x34
```

However, storing the number as a 16-bit integer, we required only 2 bytes:

```
4 * 256  +   210 

  0x04      0xD2
```

Applying the different encoding methods, the representation of our ``atoms`` table becomes

```
_atoms.type:    {ATOM, 3} HETATM
_atoms.id:      {1, 4}
_atoms.element: {C, 2} O Fe
_atoms.x        0x00 0x01 0x00 0x00
_atoms.y        0x00 0x00 0x01 0x00
_atoms.z        0x00 0x00 0x00 0x01 
```