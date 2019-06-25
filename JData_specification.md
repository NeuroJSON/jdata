 JData: A general-purpose data storage and interchange format
============================================================

- **Status of this document**: This document is current under development.
- **Copyright**: (C) Qianqian Fang (2011, 2015, 2019) <q.fang at neu.edu>
- **License**: Apache License, Version 2.0
- **Version**: 0.6 (Draft 1)
- **Abstract**:

> JData is a general-purpose data interchange format aimed for portability,
readability and simplicity. It utilizes the JavaScript Object Notation 
(JSON) [RFC4627] and Universal Binary JSON (UBJSON) specifications to 
store complex hierarchical data in both text and binary formats. In this 
specification, we define a list of JSON-compatible constructs to store
a wide range of data structures, including scalars, arrays, structures, 
tables, hashes, linked lists, trees and graphs, and support for optional data
grouping and metadata for each data element. The generated data files are 
compatible with JSON/UBJSON specifications and can be readily processed by 
most existing parsers. Advanced features such as array compression, data 
links and anchors were supported to greatly enhance portability and 
scalability of the generated data files.



## Table of Content

- [Introduction](#introduction)
    * [Background](#background)
    * [JSON and UBJSON](#json-and-ubjson)
    * [JData specification overview](#jdata-specification-summary)
- [Grammar](#grammar)
    * [Text-based JData storage grammar](#text-based-jdata-storage-grammar)
    * [Binary JData storage grammar](#binary-jdata-storage-grammar)
- [Data Models](#data-models)
    * [Topological models](#topological-models)
    * [Semantic models](#semantic-models)
- [Data Annotation Keywords](#data-annotation-keywords)
    * [Data group keywords](#data-group-keywords)
        + [Data Group](#data-group)
        + [Dataset](#dataset)
        + [Data record](#data-record)
    * [Metadata](#metadata)
    * [Data storage keywords](#data-storage-keywords)
        + [Special constants](#special-constants)
        + [N-Dimensional array storage keywords](#n-dimensional-array-storage-keywords)
            - [Direct storage of N-D arrays](#direct-storage-of-n-d-arrays)
            - [Annotated storage of N-D arrays](#annotated-storage-of-n-d-arrays)
            - [Complex-valued arrays](#complex-valued-arrays)
            - [Sparse arrays](#sparse-arrays)
            - [Sparse complex-valued arrays](#sparse-complex-valued-arrays)
            - [Compressed array storage format](#compressed-array-storage-format)
        + [Associative arrays or maps](#associative-arrays-or-maps)
        + [Tables](#tables)
        + [Trees](#trees)
        + [Singly and doubly linked lists](#singly-and-doubly-linked-lists)
        + [Directed and undirected graphs](#directed-and-undirected-graphs)
        + [Generic byte-stream data](#generic-byte-stream-data)
- [Indexing and Accessing JData](#indexing-and-accessing-jdata)
    * [Index vector](#index-vector)
    * [Data query](#data-query)
- [Conversions Between JData Files](#conversions-between-jdata-files)
- [Data Referencing and Links](#data-referencing-and-links)
- [Recommended File Specifiers](#recommended-file-specifiers)
- [Summary](#summary)



Introduction
------------

### Background


Data are the digital representations of our world. Generating and processing 
data are the essential parts of our daily lives, and are at the very 
foundations of modern sciences, information technologies, businesses, and the 
interactions between our global societies.

Data take many different forms. Some data can be represented by simple scalars; 
some others have complex forms with hierarchical structures. An efficient 
representation of data also strongly depends on the application needs. In some 
cases, plain text files with white-space separated fields are sufficient; 
however, for performance-sensitive applications, using binary formats can 
significantly reduce loading and processing time. The abilities to store and 
parse complex data structures are particularly important to the scientific 
communities.

It is a challenging task to encapsulate wide varieties of data forms in a 
single data interchange format. There have been many previous efforts in 
designing a general-purpose data storage specification. Some of them have 
become popular choices in one or multiple categories of applications. 
Extensible Markup Language (XML), for example, is ubiquitously used as a 
data-exchange format, but the verbosity of the syntax, moderate complexity for 
parsing, impeded readability and inefficiency in expressing structured data 
suggested room for alternative formats. Comma Separated Value (CSV), a rather 
simple plain-text format, is used among some applications to exchange a tabular 
data structure (such as a spreadsheet); yet, its inability to encode more 
complex data forms, lack of flexibility and precision restrict it to specific 
applications.

Hierarchical Data Format (HDF) is a format targeting at the broad needs from 
the scientific communities. It has an extensible hierarchical data model with a 
large capacity to represent complex binary data. However, to effectively use 
HDF requires skillful implementation and in-depth understanding to the 
underlying data models. For small projects with non-critical performance needs, 
using an advanced data format such as HDF may requires additional development 
and maintenance efforts. Similar arguments can be made to the Common Data 
Format (CDF) or Network Common Data Format (netCDF) that are partly derived 
from HDF. In addition, the MATLAB mat-file format and Tecplot data format are 
also used among the research communities. Because of the requirement of 
propitiatory software or libraries, these formats have also found difficulties 
to achieve wide-spread use outside of the user communities of the associated 
software.

### JSON and UBJSON

The JavaScript Object Notation (JSON) format is a text-based data format that 
is known for its capability of storing complex data, excellent portability and 
human-readability. JSON is widely adopted in modern web applications, and is 
becoming popular among local/native applications. The key advantages of JSON 
include:

* **simplicity**: JSON data are composed by a list of `"name":value` pairs; 
  such a simple grammar greatly eases the use and parsing of the data file; free 
  JSON-encoders and decoders are widely available for most popular programming 
  languages;
* **human-readability**: the text-based nature of JSON and its clean, 
  easy-to-read format make it intuitively readable without in-depth knowledge of 
  the format itself;
* **hierarchical data support**: JSON has a tree-like data storage paradigm 
  which has the native capacity to support complex hierarchical data structures; 
  there is no inherent data size limit imposed by the format itself;
*  **web-readiness**: because JSON can be readily parsed by JavaScript, most 
  JSON-encoded data file can be directly invoked (inline or load from remote 
  site) from a JavaScript based web-application.

JSON also has limitations. JSON's `"value"` fields are weakly-typed. They only 
support strings, numbers, and Boolean types, but lack of the fine-granularity 
to represent various numerical types of different byte-lengths (in C-language, 
for example, short, int, long int, float, double, long double) and their signs 
(signed and unsigned). Because JSON is a text-based format, the size of the 
data file can be significantly larger than a binary format and requires 
additional conversion when used in an application. This introduces overhead in 
both storage and processing.

The Universal Binary JSON (UBJSON) is one of the binary counterparts to the JSON format. 
It specifically addresses the above mentioned limitations, yet, adheres to a 
simple grammar similar to the text-based JSON. As a trade-off, it loses the 
"human-readability" to a certain extend. Although the implemented parsers for 
UBJSON are not as abundant as JSON, due to the simplicity of the format itself, 
the development cost for implementing a parser for a new programming language 
is significantly lower than other more complex data formats.

With the ease-of-use, superior portability and parser availability, JSON and 
UBJSON have the potentials to serve as the main-stream data storage and 
interchange formats for general needs, especially for storage and interchange
scientific data. A combination of JSON and its binary counterpart offers features 
that are not currently available with the existing data storage schemes. Although 
they do not provide all the advanced features found in the more sophisticated 
formats, their greatly simplified encoding and decoding strategies permit 
efficient data sharing among the general audiences.

### JData specification overview

JData is a specification for storing, exchanging and processing general-purpose 
data that are commonly encountered in information technology (IT) industries and 
research communities. It has a text/UNICODE format derived from the JSON 
specification and a binary format derived from the UBJSON specification. JData 
is designed to represent the commonly used data structures including arrays, 
structures, trees and graphs. A round-trip conversion is defined between the 
text and binary versions of JData documents.

The purpose of this document is to define the text and binary JData format 
specifications. This is achieved through the definition of a semantic layer 
over the JSON/UBJSON data storage syntax to map various types of complex data 
structures. Such semantic layer includes

- a list of dedicated `"name"` fields, or keywords, that define the containers 
  of various data types that are commonly used in research,
- a list of dedicated `"name"` fields and formats to facilitate the grouping and 
  organization of hierarchical data,
- a list of format properties for the associated "value" field to store the 
  specific metadata of the data points, and, in addition,
- a set of conversion rules between the text and binary forms.

In the following sections, we will define the basic JData grammar, data models, 
followed by the keywords for data grouping and various data types, including 
scalars, N-dimensional arrays, sparse and complex arrays, structures, tables, 
hashes/associative arrays, trees and graphs. The expressions of these data 
structures in both text and binary forms are specified and exemplified, and 
their conversion rules are defined.

Grammar
------------------------

### Text-based JData Storage Grammar

The text-based JData grammar is identical to the JSON grammar defined in 
[RFC4627], except that JData also accepts the Concatenated JSON (CJSON), 
streaming format, defined in the following form:
```
    {
       "object1":{}
    }
    {
       "object2":{}
    }
    ...
```

In CJSON, multiple JSON objects can be included inside a single file or stream, 
separated by 0 or multiple permitted white spaces, namely

`LF (U+000A), CR (U+000D), tab (U+0009), or space (U+0020)`


### Binary JData Storage Grammar

The binary JData grammar is identical to the UBJSON grammar defined in the
[UBJSON Specification (Draft 12)](http://ubjson.org), with the following three exceptions

1. JData stores an `[N]` (`"no-op"`) record as `null` when saving the data in the text-format,
2. JData uses IEEE 754 binary form to store +/-Infinity, instead of converting to [Z], and
3. optimized array container header was extended to support N-dimensional dense arrays:
```
[[] [$] [type] [#] [[] [$] [nx type] [#] [ndim] [nx ny nz ...] [nx*ny*nz*...*sizeof(type)]
```
   or
```
[[] [$] [type] [#] [[] [nx type] [nx] [ny type] [ny] [nz type] [nz] ... []] [nx*ny*nz*...*sizeof(type) ]
```
where `ndim` is the number of dimensions, and `nx`, `ny`, and `nz` ... are 
all non-negative numbers specifying the dimensions of the N-dimensional array,
`nz/ny/nz/ndim` types must be one of the UBJSON integer types (`i,U,I,l,L`), . 
The binary data of the N-dimensional array is then serialized in the **row-major** format 
(similar to C, C++, Javascript or Python) order.

As a special note, all UBJSON integer types must be stored in the Big-Endian 
format, according to the specification; the storage to the floating point types 
(`d,D`) follows the IEEE 754 specification.


Data Models
----------------------

### Topological models

Topologically, we define different parts of a JData document using the below terminologies

* **leaflet** : an element with no child
* **compound leaf**, or **"leaf"**: a multi-element data unit completely made of leaflets, or empty
* **branch**: a multi-element data unit made of both leaves and leaflets.
* **node**: a leaflet, a leaf or a branch
* **composite node**: a leaf or a branch
* **root**: the top-most node of a JSON object
* **super-root**: the top level node in a JData document containing multiple 
  JSON/UBJSON objects in the form of a CJSON
* **named node**: a node in the form of `"name":{...}` or `"name":[]`, in the case 
  of named and index leaves and branches, respectively, or `"name":value` in the case
  of a leaflet
* **indexed node**: a node in the form of `{...}` or `[]`, in the case of named 
  and index leaves and branches, respectively, or a single `value` in the case
  of a leaflet
* **a structure**: a named or index node made of named sub-nodes, represented by `{...}`, 
  a structure can be empty
* **an array**: a named or index node made of indexed sub-nodes, represented by `[...]`,
  an array can be empty

A leaflet can only take one of the two possible forms: `value`, referred to as the
"index leaflet", and a `"name": value` pair, referred to as the "named leaflet". 
Here `value` should not contain any sub-element.

The topological elements of a JData document - "leaflet", "leaf", "branch", "root" 
and "super-root" - have specific meanings according to the definitions above.

### Semantic models

To efficiently process the represented data, we can semantically annotate the 
underlying data structure and organize them for fine granularity. The data grouping 
annotations supported in JData include

* **data record**: a "meaningful" datum, either in the form of a simple data point 
  (leaflet) or a complex structure (a composite node); a data record can be a leaflet, 
  a leaf, or a collection of leaflets or leaves. 
* **dataset**: a set of "logically connected" data records, such as the citation 
  information of a paper
* **data group**: a group of "logically connected" datasets
* **auxiliary data**: nodes that store "weakly-relevant" or "irrelevant" data to 
the "primary" data. the auxiliary data can appear at any level in the data annotation 
hierarchy.

The interpretations to the "meaningful datum", "logically connected data", 
"primary data" and "weakly-relevant or irrelevant data" are application dependent.

The data group annotations are ranked in the below order, from the highest 
level to the lowest level:

` super-root > root >  data group >= dataset >= data record >= leaf > leaflet `

for each annotation level, it can only contain elements annotated of the same 
level or lower levels, but not the upper level annotations. All data annotation
objects can be empty, i.e. do not contain any element. An example JData 
organization schematic is shown below

```
    super-root{
        root1{
            group1{
                dataset1{
                    dataset1.1{...}
                    ...
                    ... other auxiliary data ...
                }
                dataset2{...}
            },
            group2{
                dataset3{...}
            }
            group3{}
            dataset4{...}
            ... other auxiliary data ...
        }
        root2[
            ...
        ]
    }
```
A data group, dataset or data record can be topologically presented as a branch 
(including root) or a leaf, but not a leaflet.


Data Annotation Keywords
------------------------

All JData keywords are case sensitive. Data groups, datasets and data records 
can contain metadata to include additional information regarding the data themselves, 
such as name, create date and user-defined headers. However, metadata can 
also present in any branch or leaf.

Below is a short summary of the JData data annotation/storage keywords to be introduced

* **Data grouping**: `_DataGroup_`, `_Dataset_`, `_DataRecord_`
* **N-D Array**: `_ArrayType_`, `_ArraySize_`, `_ArrayIsComplex_`, `_ArrayIsSparse_`,
  `_ArrayData_`,`_ArrayZipType_`,`_ArrayZipSize_`, `_ArrayZipData_`, `_ArrayZipEndian_`, 
  `_ArrayZipLevel_`, `_ArrayZipOptions_`
* **Hash/Map**: `_MapData_`
* **Table**: `_TableData_`, `_TableCols_`, `_TableRows_`, `_TableRecords_`
* **Tree**: `_TreeData_`,`_TreeNode_`,`_TreeChildren_`
* **Linked List**: `_ListNode_`,`_ListNext_`,`_ListPrior_`,`_LinkedList_`
* **Graph**: `_GraphData_`,`_GraphNodes_`,`_GraphEdges_`,`_GraphMatrix_`
* **Byte-stream**: `_ByteStream_`
* **Inline metadata**: `"item_name::Property1=value1,Property2=value2,...": ...`
* **Metadata record**: `{"_DataInfo_":{...}}`
* **Data links and anchors**: `_DataLink_`, `_DataAnchor_`

### Data Group Keywords

The use of data grouping keywords is not mandatory in a JData document. 
Nonetheless, properly partitioning and grouping the data based on their semantic 
relationships and name the components accordingly can greatly enhance the 
portability and readability of the data, and thus, are strongly recommended.

#### Data Group

A data group can be stored in the form of a named node, either 

` "_DataGroup_": { ... }`

or

` "_DataGroup_": [ ... ]`

A data group annotation can have a name in the below form

` "_DataGroup_(a name string): {}`

or

` "_DataGroup_(a name string): []`

If multiple data groups are stored inside a structure, a unique name 
string, among the items under the same parent object, must be provided 
for each data group. For example
```
  {
      "_DataGroup_(name1)":{...},
      "_DataGroup_(name2)":{...},
      "_DataGroup_(name3)":[...],
      ...
  }
```
When defining one of the elements of an array object as a data group, one 
must convert that index node into a structure. For example, in the below
array
```
  [
      indexed_node1, indexed_node2, indexed_node3,...
  ]
```
if one needs to define the `indexed_node2` into a data group, one must define
```
  [
      indexed_node1,
      {
          "_DataGroup_(name1)": [index_node2]
      },
      index_node3,
      ...
  ]
```
where the name `"(name1)"` is optional.

#### Dataset

Similarly, a dataset is specified by

`"_Dataset_": { ... }`

or

` "_Dataset_": [ ... ]`

with an optional name parameter in the following format:

`"_Dataset_(a name string)": ...`

####  Data record

Also similarly, a dataset is specified by

`"_DataRecord_": { ... }`

or

` "_DataRecord_": [ ... ]`

with an optional name parameter in the following format:

`"_DataRecord_(a name string)"`


### Metadata

Metadata records can be associated with a data group, dataset or data record
(or any branch or leaf). It is optional. Metadata can have two forms

* **inline metadata**: metadata defined as part of the annotation tag string, and
* **metadata node**: a metadata structure added as the first element of a node (named or indexed)

The inline metadata is defined by attaching a series of comma-separated `property=value` 
strings to the data group keywords (`_DataGroup_`,`_Dataset_`,`_DataRecord_`), separated by 
two colons, i.e. `"::"`, for example

`"_{DataGroup,Dataset,DataRecord}_(name)::Property1=...,Property2=...,..."`

The `property` name should not contain space or comma. If comma appears inside the `value`,
it must be escaped using the form `"\,"`. 

An inline metadata can be attached to any named node, for example

`"name::Property1=...,Property2=...,..."`

One or more above permitted white spaces, see above, may be inserted before or after 
separators `"::"`, `","` and `"="` without changing the interpretations of the data.

The inline metadata is primarily used to store simple properties. When storage of more complex 
metadata is needed, a dedicated "metadata node" shall be inserted to the data object as the
first child of the annotated object. 

For example, if the data to be annotated is a structure, the metadata node is defined 
by a named leaf with a specific keyword `"_DataInfo_"`. An example can be 
found below:

```
  {
    "_DataInfo_": {
       "Property1": "...",
       "Property2": "...",
       "Property3": "...",
       ...
    },
    named_node1,
    named_node2,
    ...
  }
```
If the data to be annotated is an array, the metadata record is an indexed leaf as

```
  [
    {
       "_DataInfo_": {
         "Property1": "...",
         "Property2": "...",
         "Property3": "...",
         ...
       },
    }
    indexed_node1,
    indexed_node2,
    ...
  ]
```
The property names are user-defined. Recommended properties include but not limited
to the following list
```
  Version
  Author
  Comment
  UniqueID
  CreateTime
  ModifiedTime
```

### Data Storage Keywords

JData is designed to store wide varieties of data forms. The most common data structures
used in the scientific communities include scalars, constants, arrays, structures, 
tables, associative arrays, trees, and graphs.

#### Special constants

The following constants are supported by this version of the specification

* `NaN`: An `NaN` defined by the IEEE 754 standard shall be stored as `"_NaN_"` as a string 
  leaflet in text-based JData; in the binary JData, it should be stored in the IEEE 754 format
* `+/-Inf`: A `+infinity` or `-infinity` defined by the IEEE 754 standard shall be stored as a 
  string leaflets `"+_Inf_"` and `"-_Inf_"`, respectively, in the text-based JData ("+" sign
  can be omitted); in the binary JData, they should be stored in the IEEE 754 format
* logical `true`/`false`: A logical `true`/`false` should be represented by the JSON `true/false` 
  logical values, or `[T]`/`[F]` markers in UBJSON
* `Null`: a `Null` (empty) value shall be stored as `null` in JSON and `[Z]` in UBJSON


#### N-Dimensional Array Storage Keywords

An N-dimensional array is serialized using the **row-major** format (i.e. the fastest index
is the right-most index, similar to arrays in C, C++ or Python; in comparison, MATLAB and 
FORTRAN arrays are column-majored).

A solid (non-sparse) N-dimensional array can be represented in two forms in JData - the 
direct storage format and the annotated storage format.

##### Direct storage of N-D arrays

A solid N-D array can be stored directly using JSON/UBJSON nested array constructs. For example, 
a 1-D row vector shall be stored as

`[1,2,11,9,2.1,10,...]`

while a 1-D column-vector shall be stored as

```
[[1],[2],[11],[9],[2.1],[10],...]
```
in the JSON-formatted JData.

Below is an example of a 2x3x4 3-D array stored in JSON-formatted JData:

```
  [
      [
          [1,9,6,0],
          [2,9,3,1],
          [8,0,9,6]
      ],
      [
          [6,4,2,7],
          [8,5,1,2],
          [3,3,2,6]
      ]
  ]
```
The direct storage format of a solid N-D array does not have the ability to store information
regarding the type of the original binary data if using the JSON (text-based) format; however, 
such information can be stored when using the UBJSON format. 

Please be aware that the UBJSON format supported by JData includes an extended syntax 
to facilitate storage and loading of N-D arrays in the binary format. Please refer to the 
"Grammar" section for details.

##### Annotated storage of N-D arrays

In the annotated array storage format, one shall use a structure to store an N-D array. This
allows one to use additional fields to store additional information regarding the encoded 
array.

The annotated array format is shown below for a solid 2x3 array `a=[[1,2],[3,4],[5,6]]`
```
   {
       "_ArrayType_": "typename",
       "_ArraySize_": [N1,N2,N3,...],
       "_ArrayData_": [1,2,3,4,5,6]
   }
```
Here, the array annotation keywords are defined below:

* **`"_ArrayType_"`**: (required) a case-insensitive string value to specify the type of the data, see below
* **`"_ArraySize_"`**: (required) an integer-valued (see below) 1-D row vector, storing the dimensions 
  of the N-D array
* **`"_ArrayData_"`**: (required) a 1-D row vector storing the serialized array values, using the 
  **row-major** element order

To facilitate the pre-allocation of the buffer for storage of the array in the parser, 
it is required that the `"_ArrayType_"` and `"_ArraySize_"` nodes must appear before 
`"_ArrayData_"`.

The supported data types are similar to those supported by the UBJSON format, i.e.

* **uint8**: unsigned byte (8-bit), `[U]` in UBJSON
* **int8**: signed byte (8-bit), `[i]` in UBJSON
* **uint16**: unsigned short (16-bit),  no correspondance in UBJSON, map to `[I]`
* **int16**: signed short (16-bit), `[I]` in UBJSON
* **uint32**: unsigned integer (32-bit),  no correspondance in UBJSON, map to `[l]`
* **int32**: signed integer (32-bit), `[l]` in UBJSON
* **uint64**: unsigned long long integer (64-bit), no correspondance in UBJSON, map to `[L]`
* **int64**: signed long long integer (64-bit), `[L]` in UBJSON
* **single**: single-precision floating point (32-bit), `[d]` in UBJSON
* **double**: double-precision floating point (64-bit), `[D]` in UBJSON

The first 8 data types are considered "integer" types, and the last two types are considered 
"floating-point" types.


##### Complex-valued arrays

JData supports the storage of complex values or arrays using only the annotated N-D array storage
format. In this case, a named node `"_ArrayIsComplex_":true` must be added into the 
structure. In the meantime, `"_ArrayData_"` shall be defined as a 2-D array by concatenating the
serialized real and imaginary parts of the complex array as row vectors, with the real-part 
as the first row vector, and the imaginary-part in the 2nd row vector. The two row 
vectors must have the same length.

For example, a complex double-precision 1x3 row vector `a=[2+6*i, 4+3.2*i,  1.2+9.7*i]` shall be stored as

```
   {
       "_ArrayType_": "double",
       "_ArraySize_": [1, 3],
       "_ArrayIsComplex_": true,
       "_ArrayData_": [[2, 4, 1.2], [6, 3.2, 9.7]]
   }
```

The `"_ArrayIsComplex_"` node must be presented before `"_ArrayData_"`.

##### Sparse arrays

JData also supports the storage of N-D sparse arrays using the annotated N-D array storage
format. In this case, a named node `"_ArrayIsSparse_":true` must be added into 
the structure. In the meantime, the `"_ArrayData_"` shall be defined as a 2-D array by 
concatenating the N-tuple integer indices (left-most index first, and so on), each as a
row vector, followed by the row vector of the serialized non-zero array element 
values. For an N-D sparse array, each non-zero value requires an N-tuple index to specify 
its location.

For example, if a 3-D sparse array has the following non-zero element at the specified 
locations by a triplet `(i1, i2, i3)`:
```
  a: i1, i2, i3, value
      2   3   1   10.1
      3   1   1   9.0
      3   3   1   8.1
      5   1   2   17
      5   2   2   9.4
      2   2   3   20.5
```
it shall be stored in the following JSON format
```
   {
       "_ArrayType_": "double",
       "_ArraySize_": [5, 4, 3],
       "_ArrayIsSparse_": true,
       "_ArrayData_": [[2,3,3,5,5,2], [3,1,3,1,2,2], [1,1,1,2,2,3], [10.1,9.0,8.1,17,9.4,20.5]]
   }
```
The `"_ArrayIsSparse_"` node must be presented before `"_ArrayData_"`.

##### Sparse complex-valued arrays

Using the combination of `"_ArrayIsComplex_"` and `"_ArrayIsSparse_"`, one can store
a complex-valued sparse array using JData. In this case, both `"_ArrayIsComplex_":true` 
and `"_ArrayIsSparse_":true` must be presented in the structure, with `"_ArrayData_"` 
ordered by the N-tuple non-zero element indices (left-most index first, and so on), 
each as a row vector, followed by a row vector for the real-values of the non-zero elements, and 
lastly the row vector for the imaginary-values of the non-zero elements. All row vectors
of `"_ArrayData_"` must have the same length.

For example, if a 3-D sparse array has the following non-zero complex element at the 
specified indices `(i1, i2, i3)`
```
  a: i1, i2, i3,  complex values
      2   3   1   10.1+19.0*i
      3   1   1   9.0+11*i
      3   3   2   8.1+8.2*i
```
it shall be stored in the following JSON format
```
   {
       "_ArrayType_": "double",
       "_ArraySize_": [4, 3, 2],
       "_ArrayIsComplex_": true,
       "_ArrayIsSparse_": true,
       "_ArrayData_": [[2,3,3], [3,1,3], [1,1,2], [10.1,9.0,8.1], [19.0,11,8.2]]
   }
```
or the corresponding UBJSON equivalents.

##### Compressed array storage format

JData supports node-based data compression to enhance space-efficiency of the generated
files. Compressed data format is only supported in the annotated array storage format.

Four additional nodes are added to the annotated array structure

* **`_ArrayZipType_`**: (required) a string-valued leaflet to store the compression 
  method, for example, "zlib", "gzip" or "lzma"
* **`_ArrayZipSize_`**: (required) the dimensions of the pre-processed array, i.e. the data
  originally stored in `_ArrayData_` in the format specified by `"_ArrayType_"`, 
  before the array binary stream type-casted to byte stream and compression.
* **`_ArrayZipData_`**: (required) in addition, the `"_ArrayData_"` node is replaced by 
  `"_ArrayZipData_"`. In the case of JSON-formatted JData files, 
  `"_ArrayZipData_"` has a string value storing the "Base64" encoded compressed 
  byte-stream of the pre-processed array. In the case of UBJSON flavored JData, 
  `"_ArrayZipData_"` directly stores the compressed byte stream of the 
  pre-processed array without "Base64" encoding.

In addition, the following optional parameters may also be used

* **`_ArrayZipEndian_`**: (optional) a case-insensitive string, either "little" or "big",
  indicating the endianness of the byte-stream before compression; if missing, assume
  to be "little"
* **`_ArrayZipLevel_`**: (optional) a numerical value, typically an integer between
  0 and 9, specifying the level of the compression (interpretation is method/library-dependent)
* **`_ArrayZipOptions_`**: (optional) an array object allowing users to specify
  additional compression-method specific parameters (interpretation is method/library-dependent)

When a compressed array format is used, `"_ArrayZipType_"` and 
`"_ArrayZipSize_"` must appear before `"_ArrayZipData_"`.


#### Associative arrays or maps

In a map or an associative/hashed array, the element can be accessed using a unique
key. For example, the below pseudo code defines a 3-element associative array `@a` with
3 unique keys
```
   @a = {'Andy'->21, 'William'->21, 'Om'->22}
```
Such data structure can be conveniently represented using JSON/UBJSON as
```
   {
       "Andy":    21,
       "William": 21,
       "Om":      22
   }
```

To store maps with non-string valued keys, one can use a 2-D array enclosed inside 
a `"_MapData_"` construct, for example
```
   "_MapData_": [
       ["Andy",   21],
       ["William",21],
       ["Om",     22],
       [120, 30],
       [2.9, 45]
   ]
```
Each element of the `_MapData_` array may contain more than 2 elements; the additional 
elements shall be treated as auxiliary data.

#### Tables

A table data structure is defined by a 2-dimensional grid of data indexed by the table
columns (fields) and rows (records). For example

```
  Name    Age   Degree  Height
  ----  ------- ------  ------
  Andy    21     BS      69.2
  William 21     MS      71.0
  Om      22     BE      67.1
```
Such data can also be stored in JSON/UBJSON using 3 dedicated keywords
* **`"_TableCols_"`**: a 1-D array denoting the name and type for each of the column of the table, 
     can be empty if no name or types are associated with the columns
* **`"_TableRows_"`**: a 1-D array denoting the name and type for each of the row of the table
     can be empty if no name or types are associated with the rows
* **`"_TableRecords_"`**: a 2-D array storing each entry in the 2-D table.

For example, the above table can be serailized using the below format
```
    {
        "_TableCols_": ["Name", "Age", "Degree", "Height"],
	"_TableRows_": [],
	"_TableRecords_": [
	   ["Andy",    21, "BS", 69.2],
	   ["William", 21, "MS", 71.0],
	   ["Om",      22, "BS", 67.1]
	]
    }
```

The elements in the `"_TableRows_"` and `"_TableCols_"` can be a structure to accommodate 
additional metadata associated with the row or column. One can use `"DataName"` to specify 
the string name of the row or column, and `"DataType"` to specify their data types.

The supported `"DataType"` values are the same as those defined for `"_ArrayType_"` for 
numerical arrays, with the addition of the below types

* `"string"`: a UTF-8 encoded string
* `"bool"`: a boolean: either `true` or `false`
* `"blob"`: a binary byte-stream, the format of the `blob` record is described in the [Generic byte-stream data](#generic-byte-stream-data) below (without the `"_ByteStream_"` container)

For example, in the above example, one can define the columns as
```
    "_TableCols_": [
        {"DataName": "Name", "DataType": "string"}, 
        {"DataName": "Age", "DataType": "uint32"}, 
	{"DataName": "Degree", "DataType": "string"}, 
	{"DataName": "Height", "DataType": "single"}
    ],
    ...
```

In addition, a table can also be converted to an array of maps if a name is associated 
with each of the rows or columns.

* **Array of structures**: the table data can be serialized by grouping records in rows, such as

```
   [
       {"Name": "Andy",   "Age": 21, "Degree": "BS", "Height": 69.2},
       {"Name": "William","Age": 21, "Degree": "MS", "Height": 71.0},
       {"Name": "Om",     "Age": 22, "Degree": "BE", "Height": 67.1}
   ]
```

* **Structure of arrays**: the table data can also be organized in columns, where often 
  contains the same data types. Using the same above example, one can write the following 
  JData form
```
   {
       "Name":   ["Andy","William","Om"],
       "Age":    [21,21,22],
       "Degree": ["BS","MS","BE"],
       "Height": [69.2,71.0,67.1]
   }
```

The choice of the forms is user-dependent. Typically, the "structure of arrays" form 
leads to a smaller file size.

The above JData table can be enclosed inside an optional field `"_TableData_(table_name)":{...}` 
(if inside a structure) or `{"_TableData_":{...}}` (if inside an array) to inform the 
parser the start of the data structure.


#### Trees

A tree-like data structure can be conveniently represented by JSON/UBJSON formatted files
due to the similar underlying structures. In JData, we use two keywords to encapsulate a 
tree-like data structure. 

The tree-node name and the associated data are stored using a named node with a keyword
`_TreeNode_`:

`"_TreeNode_(node_name)": node_data`

If a tree-node contains children, we use a named array to store the children data:

```
"_TreeChildren_":[
     {"_TreeNode_(child1)":data1},
     {"_TreeNode_(child1)":data2},
     ...
]
```

For example, a tree data structure 
```
   root(data0)
     ├──── node1(data1)
     ├──── node2(data2)
     │       ├──── node2.1(data2.1)
     │       └──── node2.2(data2.2)
     └──── node3(data3)
```
can be represented by the below JSON structure
```
  {
     "_TreeNode_(root)": data0,
     "_TreeChildren_": [
         {"_TreeNode_(node1)": data1},
         {
             "_TreeNode_(node2)": data2,
             "_TreeChildren_": [
                 {"_TreeNode_(node2.1): data2.1},
                 {"_TreeNode_(node2.2): data2.2}
             ]
         },
         {"_TreeNode_(node3)": data3}
     ]
  }
```
and the corresponding UBJSON equivalents. The notations "data0", "data1" etc are parsed
node data in JData format according to the rules defined in this section, depending on
the type of the data.

One can add either inline or dedicated metadata record to store additional information
about the tree nodes, for example
```
  {
     "_TreeNode_(root)::NodeID=1,ParentID=0,Path=#root": data0,
     "_TreeChildren_": [
         {"_TreeNode_(node1)::NodeID=2,ParentID=1,Path=#root#node1": data1},
         {
             "_TreeNode_(node2)::NodeID=3,ParentID=1": data2,
             "_TreeChildren_": [
                 {"_TreeNode_(node2.1)::NodeID=4,ParentID=3": data2.1},
                 {"_TreeNode_(node2.2)::NodeID=5,ParentID=3": data2.2}
             ]
         },
         {"_TreeNode_(node3)::NodeID=6,ParentID=1": data3}
     ]
  }
```
Auxiliary data can be inserted to different levels of the above JData tree document
as named nodes, as long as the name of the auxiliary node does not conflict with
the `"_TreeNode_(name)"` and `"_TreeChildren_"` of the same level. Behaviors for parsing 
the auxiliary data are application dependent.

The above JData tree can be enclosed inside an optional field `"_TreeData_(tree_name)":{...}` 
(if inside a structure) or `{"_TreeData_":{...}}` (if inside an array) to inform the 
parser the start of the data structure.



#### Singly and doubly linked lists

Similar to the storage of trees, in JData, we use additional keywords to encapsulate a singly 
or doubly linked list inside a JSON/UBJSON array construct. The relevant keywords are

`"_ListNode_(unique_name)": node_data`

and

`"_ListNext_": "unique_name_of_next_node"`
`"_ListPrior_": "unique_name_of_prior_node"`

For example, the below linked list

```
   head(data0)->node1(data1)->node2(data2)->node3(data3)
```

shall be stored as
```
  [
     {
         "_ListNode_(head)": data0,
         "_ListNext_": "node1"
     },
     {
         "_ListNode_(node1)": data1,
         "_ListNext_": "node2"
     },
     {
         "_ListNode_(node2)": data2,
         "_ListNext_": "node3"
     },
     {
         "_ListNode_(node3)": data3,
         "_ListNext_": null
     }
  ]
```
If a node does not have a next or prior element, the `"_LinkNext_"` or `"_LinkPrior_"` value 
should be set to `null`.

The name label referred in the `"_LinkNext_"` or `"_LinkPrior_"` fields has a scope limited to 
the current list, i.e. the parent array one-level above the node. Multiple linked lists can 
share the same names if they are stored in a parallel or nested fashion.

The above linked list can be enclosed inside an optional field `"_LinkedList_(list_name)":[...]` 
(if inside a structure) or `{"_LinkedList_":[...]}` (if inside an array) to inform the parser 
the start of the data structure.


#### Directed and undirected graphs

In JData, we use the below keywords to encapsulate a graph data structure

* **`"_GraphNodes_"`**: a JData array object to store the serialized node data
* **`"_GraphEdges_"`**: a JData array object to store the connections between nodes. Each edge 
is represented by a 2 or 3-element array `["start node name", "end node name", (optional) edge data]`.

For example, if we modify the above linked list by adding a directed edge from `"node1"` to 
`"node3"` and from `"node3"` to `"node2"`, such as the data suggested by this diagram

```
                    ┌--------------------------┐
                    │                          V
   head(data0)->node1(data1)->node2(data2)->node3(data3)
                                 ^             │
                                 └-------------┘
```
we can then store this data structure as

```
    {
        "_GraphNodes_":{
              "head":  data0,
              "node1": data1,
              "node2": data2,
              "node3": data3
        },
        "_GraphEdges_":[
               ["head",  "node1", edgedata01],
               ["node1", "node2", edgedata12],
               ["node1", "node3", edgedata13],
               ["node2", "node3", edgedata23],
               ["node3", "node2", edgedata32]
        ]
    }
```
The data associated with each edge (edgedata) in this example is optional and can be any JData 
structure supported in this document. If the edge data is a scalar, it can be interpreted as 
weights in a weighted graph.

By default, the graph is assumed to be a directed graph. If a user intends to store undirected 
graph using the above format, one must use `"_GraphEdges_(false)"` or `"_GraphEdges_(0)"` to 
enclose the edge data.

The above graph data can be enclosed inside an optional field `"_GraphData_(graph_name)":{...}` 
(if inside a structure) or `{"_GraphData_":{...}}` (if inside an array) to inform the parser the 
start of the data structure.

When no data or metadata are associated with the edges, one can also use the graph's adjacency 
matrix to represent the connections between graph nodes. The adjacency matrix must only have 0 
and 1 values, with a 1 at A(i,j) indicating a directed edge from the i-th node in the 
`_GraphNode_` list to the j-th node in that list, and 0 indicating no connection.

For example, the adjacency matrix of the above graph can be written as
```
           head    node1   node2   node3
  head       0       1       0       0
  node1      0       0       1       1
  node2      0       0       0       1
  node3      0       0       1       0
```
In this case, instead of using `_GraphEdges_`, we use `_GraphMatrix_` to store the
adjacency matrix using the N-D array syntax above. For example, for the same graph, 
we can write this matrix using direct form

```
    {
        "_GraphNodes_":{
              "head":  data0,
              "node1": data1,
              "node2": data2,
              "node3": data3
        },
        "_GraphMatrix_":[
          [0,1,0,0],
          [0,0,1,1],
          [0,0,0,1],
          [0,0,1,0]
        ]
    }
```
Notice that the matrix is serialized in the **row-major** order in the above example. Alternatively, one
can also use the annotated array format to take advantage of the sparsity of the data:
```
    {
        "_GraphNodes_":{
              "head":  data0,
              "node1": data1,
              "node2": data2,
              "node3": data3
        },
        "_GraphMatrix_":{
             "_ArrayTye_": "uint8",
             "_ArraySize": [4,4],
             "_ArrayIsSparse": true,
             "_ArrayData_": [1,2,2,3,4, 2,3,4,4,2, 1,1,1,1,1]
        }
    }
```
One can also apply array compression, as explained above, to further reduce the file size
```
  {
    ...
	"_GraphMatrix_": {
		"_ArrayType_": "uint8",
		"_ArraySize_": [4,4],
		"_ArrayZipSize_": [1,16],
		"_ArrayZipType_": "zlib",
		"_ArrayZipEndian_": "little",
		"_ArrayZipData_": "eJxjYGQAAkYQyQhCAAA5AAY=="
	}
  }
```
here the `"_ArrayZipData_"` stores the row-major-serialized, byte-typecasted,
zlib-compressed and finally base64-encoded adjacency matrix.

In addition, a weighted graph can also be stored using the adjacency matrix by replacing
the "1"s in the matrix by the weight value (a numerical scalar).


#### Generic byte-stream data

A binary byte-stream of arbitrary length can be stored in the text-based JData using the below construct

`"_ByteStream_":"..."`

where the string value `"..."` shall be the Base64-encoded byte-stream binary data. For example, a string
valued byte-stream `"JData specification"` shall be stored as

`"_ByteStream_":"SkRhdGEgc3BlY2lmaWNhdGlvbg=="`

In the binary JData, a byte-stream shall be encoded using a similar `"name":value` pair where the 
`value` is reprented by an `[H]` marker. As specified in the 
[UBJSON Specification (Draft 12)](http://ubjson.org/type-reference/value-types/#numeric-gt-64bit), 
the `[H]` marker is immediately followed by the length of the byte stream, then followed by 
the raw binary values of the byte-stream **without Base64 encoding**. The same example above 
can be stored as

`[U][12][_ByteStream_][H][U][19][JData specification]`

The parsing and interpretation of the byte-stream data are application dependent. This is the most 
generic form of data storage but contains the least data semantic information. One can use this 
construct as containers to binary files, data segments, or encrypted data or files. In the case 
of data encryption, additional information related to the encryption and decryption, if needed, 
shall be stored as the metadata to the `"_ByteStream_"` object with a format specified in the 
[Metadata section](#metadata).


Indexing and Accessing JData
------------------------------

A JData-compliant parser should support the below programming interfaces for accessing 
and processing JData files

* **`JD_GetNode(root_node, index_vector, is_compact)`**: returns the JData node pointed by the index 
     vector, reference from the `root_node`
* **`JD_GetName(node)`**: returns the full name string of the specified node
* **`JD_GetData(node)`**: returns the value associated with the specified node
* **`JD_GetType(node)`**: returns the node type
* **`JD_GetLength(node)`**: returns the number of children of the specified node

### Index vector

Essentially, JData stores a serialized version of the complex data using collections 
of sequential or nested nodes, either in the named or indexed form. To access
any element (a leaflet, leaf or branch) of the JData document, one should use a vector 
of indices that points to the specific node. 

A JData-compliant parser must be able to retrieve JData elements via the below pseudo-code 
interface using a linear index vector
```
   JD_Node item = JD_GetNode(JD_Node root, [i1, i2, i3, i4, ...], is_compact)
```
where `i1` is the index of the data on the top-most level (relative to the root level of the 
`"root"` object), `i2`, is the index along the 2nd level, and so on. Each index is an 
integer, starting from 1, denoting the order
of the data element among the serialized elements of the same level. In order words, if
the current level is an array object, the index is the count of the elements before this data
element plus 1; if the current level is a structure, the index is the count of the named 
nodes appearing before this data plus 1. Using the tree data structure above, the linear index
of each node is listed below on the right side:

```
  {
     "_TreeNode_(root)": data0,                     <- [1]
     "_TreeChildren_": [                            <- [2]
         {"_TreeNode_(node1)": data1},              <- [2,1]
         {                                          <- [2,2]
             "_TreeNode_(node2)": data2,            <- [2,2,1]
             "_TreeChildren_": [                    <- [2,2,2]
                 {"_TreeNode_(node2.1): data2.1},   <- [2,2,2,1]
                 {"_TreeNode_(node2.2): data2.2}    <- [2,2,2,2]
             ]
         },
         {                                          <- [2,3]
             "_TreeNode_(node3)": data3             <- [2,3,1], or [[2,3]]
         }
     ]
  }
```
One can insert zeros to the right-side of the indexing vector if the array storing the vector
has a length longer than the depth of the assessed node. In this case, the first 0 scanning
from left to right of the indexing vector is considered the termination flag of the index.
In other words, index vectors `[2,2]`, `[2,2,0]` and `[2,2,0,0]` are equivalent. 

The third parameter, `is_compact` is a boolean flag. If set to `true`, `JD_GetNode` 
shall skip the index if any of the dimensions along the indexing vector is a singlet, 
i.e. the child count is 1. The compact indexing vector, enclosed by double-square-brackets 
as `[[...]]`, shall be passed to `JD_GetNode` as the 2nd input when `is_compact` is `true`.
Using the above example, both index vectors [[2,3]] and [2,3,1] refer to 
`"_TreeNode_(node3)": data3`. Please be aware that the compact indexing vector can not
distinguish between row and column vectors, as the column vector in JData has a trailing 
singlet dimension ([see N-D array section](#direct-storage-of-n-d-arrays)).

An optional alternative indexing vector definition allows to replace the index within a 
structure by a corresponding string, can be the name of the data item or a hashed version. 
For example, the item `{"_TreeNode_(node2.1): data2.1}` in the above may also be accessed 
via `["_TreeChildren_",2,"_TreeChildren_",1]`. This alternative indexing scheme is less 
sensitive to data serialization orders, but requires the parser to handle both string
and integer inputs.


### Data query

For each JData item identified via an indexing vector, a JData-compliant library must be able 
retrieve the `"name"` and `"data"` properties of the object via the below pseudo-code interface 
```
    string   name=JD_GetName(JDataNode item)
    JD_Node  data=JD_GetData(JDataNode item)
```
here `"name"` is a string variable recording the full item name, including the inline metadata
if present; `"data"` is the "value" of the object. When the inquired data object is an element 
in an array, the returned name must be empty.

A JData-compliant parser should also allow users to retrieve the type and the children count
of the node via the below interfaces
```
    JD_NodeType type = JD_GetType(JD_Node item)
```
The type must be able to distinguish the below 3 basic types 

1. a leaflet, 
2. a structure, 
3. an array

Combining with the `JD_GetName`, one can also query if the element is a named one or indexed one.

Additionally, JData-compliant library must provide the below interface to obtain the count of
the childrens for each data item. A leaflet, an empty structure or an empty array should return
a length of 0.
```
    integer length   = JD_GetLength(JD_Node item)
```


Conversions Between JData Files
---------------------------------

One can choose either the text or binary format to save the raw data into
a JData file, with the former following the JSON storage requirements and the 
latter following the UBJSON storage requirements. This specification permits
both lossless and lossy conversions between the raw data to JSON, raw data to 
UBJSON, and between JSON and UBJSON files. 

* **lossless conversion**: the input data are preserved after a save-load 
  round-trip conversion to and from one of the JData formats; type-casting 
  from low precision to high precision types is permitted
* **lossy conversion**: the input data may lose precision during the 
  save-load round-trip conversion but the loss of precision shall be limited
  to the level that is tolerable by the application.

For best practices, use of lossless conversion is highly recommended. Here are 
some general recommendations to best preserve data precision:

* Use binary-based JData to retain data type and binary information
* When using text-based JData, make sure to print sufficient decimals for 
  floating-point numerical data (typically 7 for single-precision and 17 for 
  double-precision values) to retain the full precision
* When saving numerical arrays to text-based JData, consider using annotated 
  array storage format, either with or without data compression to store 
  binary type information
* When parsing numerical arrays stored in the direct storage format in text-based
  JSON, one should consider using double-precision as the read-buffer to avoid
  truncation of input data and loss of precision.

In addition, if a transformation to the data does not alter the (full or compact) 
indexing vector to all leaflets in a JData document, it is referred to as an 
**"isometric transform"**, and is permitted. An example of an isometric transform 
is the conversion from a structure to an array as shown in the below example:
```
   {
      "a": {
         "name1": value1,      <- [1,1] or ["a","name1"]
         "name2": value2,      <- [1,2] or ["a","name2"]
         "name3": value3       <- [1,3] or ["a","name3"]
      },
      "b": "value4"            <- [2] or ["b"]
   }
```
to
```
   {
      "a": [
         {
             "name1": value1   <- [[1,1]] or [["a","name1"]]
         },
         {
             "name2": value2   <- [[1,2]] or [["a","name2"]]
         },
         {
             "name3": value3   <- [[1,3]] or [["a","name3"]]
         }
      ],
      "b": "value4"            <- [2] or ["b"]
   }
```
The only permitted "non-isometric transform" is the conversion between a direct
N-D array to an annotated N-D array.


Data Referencing and Links
---------------------------------

JData files support referencing and internal/external linking via the definitions of
data links and anchors. 

A link is defined by a named leaflet or leaf as shown in the below two styles
```
   "link_style1": {
       "_DataLink_": "path"
   },
   "link_style2": {
       "_DataLink_": {
           "URI": "path",
           "Parameters": [...],
           "MaxRecursion": 1
       }
   }
```
The `"path"` string specifies the Uniform Resource Identifier (URI) of the referenced 
JData data document, using a format compliant to the [RFC3986] specification, followed 
by the indexing vector string to point to a specific element of the referenced document. 
For example, the below link
```
   {
      "_DataLink_": "file:///space/test/jdfiles/tree.jdat:[1,2,2]"
   }
```
asks the parser to read a local file located at "/space/test/jdfiles/tree.jdat" and
load the node specified by indexing vector `[1,2,2]`, starting from the root (or super-root
if containing CJSON) to replace the `"_DataLink_"` node in the current document.

If using a `"_DataLink_"` structure, additional parameters can be specified via
user-defined parameters, such as `"Parameters"` and `"MaxRecursion"` to fine-tune
the linking behavior.

For easy referencing, JData permits the definitions of **named anchors** inside the 
"metadata" section for each node using the following format
```
    "obj::_DataAnchor_=a_unique_anchor_name": ...
```
or
```
   "obj": {
       "_DataInfo_"{
           ...
           "_DataAnchor_": "a_unique_anchor_name",
           ...
       }
       ...
   }
```
Then, the data object can be referenced as shown in the below example `"global_link1"`
```
   "global_link1": {
      "_DataLink_": "https://example.com:8080/space/test/jdfiles/tree.jdat#a_unique_anchor_name"
   },
   "local_link1": {
      "_DataLink_": "#a_unique_anchor_name"
   }
   "local_link2": {
      "_DataLink_": [1,2,2]
   }
   "local_compact_link3": {
      "_DataLink_": [[1,2,2]]
   }
```
A data link URI starting with "#" refers to the data anchor defined within the same document,
such as shown in the `"local_link1"` example above. Similarly, one can directly use the 
indexing vector, in the form of an array, in the value field of `"_DataLink_"` to cite 
a local node, as shown in the `"local_link2"` example above. A compact indexing vector 
(eliminating singlet dimensions) can be represented by a pair of double-square-brackets, 
i.e. 1-level nested vector, as shown in the `"local_compact_link3"` example above. The 
behaviors of other types of data link values are not specified.


Recommended File Specifiers
------------------------------

For the text-based JData file, the recommended file suffix is **`".jdat"`**; for 
the binary JData file, the recommended file suffix is **`".ubjd"`**.

The MIME type for the text-based JData document is 
**`"application/jdata-text"`**; that for the binary JData document is 
**`"application/jdata-binary"`**


Summary
----------

The major attractions of JSON and UBJSON are their simplicity and portability, which
are often missing from other alternatives. In this document, we aim to extend the ability
of JSON/UBJSON in storage and interchange complex data structures without needing to 
modify the language syntax, making the generated JData files readily usable for most 
existing JSON/UBJSON encoders and decoders.

Specifically, we defined JSON/UBJSON-based constructs to store N-D arrays, tables, trees,
linked lists, and graphs, and added the ability to associate metadata to any elements
of the JData document. In addition, we also define a set of core library interfaces to 
query and access the values and properties of the data units stored in a JData document.
To enhance space-efficiency and flexibility, we also introduced array data compression 
and data linking/referencing mechanisms. 

Although JData does not have all the sophisticated features as other advanced binary 
data exchange formats, such as HDF5, it is well suited for storage of small to medium
sized datasets generated in many scientific domains or IT applications. Combined with 
the excellent availability of parsers and web-friendliness, JData is expected to be 
easily adopted and extended in the future.
