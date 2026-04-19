JData: A general-purpose data annotation and interchange format
============================================================

- **Copyright**: (C) Qianqian Fang (2011, 2015-2026) <q.fang at neu.edu>
- **License**: Apache License, Version 2.0
- **Version**: V1 (Draft-4)
- **URL**: https://neurojson.org/jdata/draft4
- **Status**: Under development
- **Development**: https://github.com/NeuroJSON/jdata
- **Acknowledgment**: This project is supported by US National Institute of Health (NIH)
  grant [U24-NS124027 (NeuroJSON)](https://neurojson.org)
- **Abstract**:

> JData is a general-purpose data interchange format aimed for portability,
readability and simplicity. It utilizes the JavaScript Object Notation 
(JSON) [RFC4627] and binary JSON formats to store complex hierarchical
data in both text and binary formats. In this specification, we define
a list of JSON-compatible annotations to encode a wide range of commonly
used data structures, including scalars, arrays, structures, tables,
hashes, linked lists, trees and graphs, and support optional annotations
for data grouping and metadata for each data element. The generated data files are 
compatible with JSON/binary JSON specifications and can be readily processed by 
most existing parsers. Advanced features such as array compression, data 
linking and anchoring are supported to enhance portability and 
scalability of the generated data files.



## Table of Content

- [Introduction](#introduction)
    * [Background](#background)
    * [JSON and Binarh JData](#json-and-binary-jdata)
    * [JData specification overview](#jdata-specification-overview)
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
            - [Special matrices](#special-matrices)
            - [Complex-valued special matrices](#complex-valued-special-matrices)
            - [Compressed array storage format](#compressed-array-storage-format)
        + [Associative arrays or maps](#associative-arrays-or-maps)
        + [Tables](#tables)
        + [Enumerations](#enumerations)
        + [Trees](#trees)
        + [Singly and doubly linked lists](#singly-and-doubly-linked-lists)
        + [Directed and undirected graphs](#directed-and-undirected-graphs)
        + [Generic byte-stream data](#generic-byte-stream-data)
- [Indexing and Accessing JData](#indexing-and-accessing-jdata)
    * [Index vector](#index-vector)
    * [Data query](#data-query)
- [Converting Between JData Files](#converting-between-jdata-files)
- [Data Referencing and Links](#data-referencing-and-links)
- [Recommended File Specifiers](#recommended-file-specifiers)
- [Summary](#summary)



Introduction
------------

#### Background

Open sharing of scientific data has become a cornerstone of modern research,
driven by community standards such as the FAIR principles -- Findability,
Accessibility, Interoperability, and Reusability -- and mandated by an
increasing number of funding agencies and journals.  Yet while open-source
software development has converged on widely accepted, human-readable
"source-code" formats -- plain text files under version control, universally
readable without specialized tools -- the sharing of complex scientific
datasets lacks an equivalent convention.  Shared datasets today are commonly
distributed in domain-specific binary formats whose readability and
long-term usability depend entirely on the continued availability of
specialized parsers and libraries.

This tight coupling between data and its parser creates a fundamental
fragility.  Binary formats encode data in opaque byte sequences whose
interpretation requires an external schema -- often embedded only in the
parser source code or a separate specification document.  As formats
evolve, parsers are updated or retired, and the risk of datasets becoming
unreadable grows with time.  The scientific record is consequently
vulnerable: data carefully collected and shared today may be effectively
inaccessible to researchers a decade from now, undermining the very
reproducibility that data sharing is meant to support.  A durable solution
requires a format in which the data structure is self-describing and
human-readable -- analogous to source code -- rather than opaque and
parser-dependent.

#### JData as the source-code format for scientific data

JData is designed to serve as the ``source-code'' format for scientific data
exchange -- a representation that is simultaneously machine-processable and
directly interpretable by humans, without requiring domain-specific tooling.
By encoding data structures using self-explanatory, standardized annotation
keywords embedded within the data file itself, JData makes the organization
and meaning of a dataset transparent at the point of storage.  This
human-readability also makes JData inherently compatible with modern
artificial intelligence (AI) tools: large language models and AI-assisted
data pipelines can parse, query, and reason about JData documents without
bespoke adapters, enabling sophisticated data interoperation, automated
processing, and broad reuse in ways that opaque binary formats preclude.

In this specification, we define a set of JSON-compatible annotation
keywords to unambiguously map the data structures most commonly encountered
in research -- N-dimensional (N-D) arrays, associative arrays (maps),
tables, trees, linked lists, directed and undirected graphs, and binary
large objects (blobs) -- into standardized, self-describing JSON and binary
JSON wrappers.  Built-in support for internal data compression is also
provided, allowing JData files to remain compact without sacrificing
readability or portability.

#### Building on the JSON ecosystem

JData is built upon the JavaScript Object Notation (JSON) format [RFC4627]
-- an internationally standardized (ECMA-404, ISO21778:2017), text-based
data-exchange format that has become one of the most widely adopted
serialization standards across web and native applications.  By anchoring
JData to JSON, the specification immediately inherits a vast and mature
ecosystem of free parsers available for virtually every programming language,
as well as a suite of powerful complementary technologies: JSON Schema for
automated data validation, JSONPath for structured data queries, JSON-LD for
semantic data linking and referencing, and NoSQL database engines -- such as
CouchDB and MongoDB -- that natively ingest JSON documents for scalable,
searchable data storage.  This ecosystem transforms JData from a static file
format into an active data platform capable of supporting automated
pipelines, cross-dataset queries, and large-scale data integration.

#### Binary JData for performance-sensitive applications

For applications where storage efficiency and parsing speed are critical,
JData provides a binary representation through the Binary JData (BJData)
format, derived from the Universal Binary JSON (UBJSON) specification.
BJData retains a grammar closely aligned with JSON while addressing its core
limitations for scientific use: it supports strongly typed numerical data
across a full range of integer and floating-point precisions, enables
storage of N-dimensional packed arrays in an optimized container format, and
lifts the per-record and total-file size restrictions imposed by competing
binary formats such as MessagePack and BSON.  A distinguishing property of
BJData is that all semantic elements -- record name tags and data-type
markers -- remain human-readable strings, placing it in a unique
``quasi-human-readable'' category absent from almost all other binary
formats.  Conversion between text JData and binary JData is lossless and
fully specified, so the choice between the two representations is purely
a performance trade-off with no loss of data fidelity or self-description.

#### Scope of this document

The remainder of this specification defines the text and binary JData
grammar, the topological and semantic data models, and the complete set of
data annotation keywords.  For each supported data structure, both the text
and binary representations are specified and exemplified, and their mutual
conversion rules are defined.  We first describe the basic JData grammar,
then the data annotation keywords for data grouping, N-D arrays, maps,
tables, trees, linked lists, graphs, and byte-stream data, followed by
indexing and query conventions, data referencing and linking, and a
summary of recommended file specifiers.

Grammar
------------------------

### Text-based JData Storage Grammar

The text-based JData grammar is identical to the JSON grammar defined in 
[RFC4627], with the exception that JData also accepts the Concatenated JSON (CJSON)
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

The Binary JData (BJData) format used in this specification is based on 
[BJData Specification (Draft 4)](https://neurojson.org/bjdata/), 
which is extended from the widely used [UBJSON Specification (Draft 12)](https://ubjson.org), 
with the addition of the following features

1. BJData supports 5 new data type markers: `[u]: uint16`, `[m]:  uint32`, `[M]: uint64`, `[h]: float16` and `[B]: byte`
2. BJData allows extension of custom binary data types via the new Extended data type `[E]`
3. BJData uses little-Endian (LE) as the default numeric (integers and floating-point numbers) data byte order
as oppose to the big-Endinan (BE) byte order used by UBJSON
4. BJData uses the respective IEEE 754 binary form to store +/-Infinity and NaN instead 
of converting to [Z]
5. optimized array container header was extended to support N-dimensional packed arrays by
attaching a 1-D integer array construct following the `#` (count) marker, for example

```
[[] [$] [type] [#] [[] [$] [nx type] [#] [ndim type] [ndim] [nx ny nz ...] [nx*ny*nz*...*sizeof(type)]
```
   or
```
[[] [$] [type] [#] [[] [nx type] [nx] [ny type] [ny] [nz type] [nz] ... []] [nx*ny*nz*...*sizeof(type) ]
```
where `ndim` is the number of dimensions, and `nx`, `ny`, and `nz` ... are 
all non-negative numbers specifying the dimensions of the N-dimensional array.
`nz/ny/nz/ndim` types must be one of the BJData integer types (`i,U,I,u,l,m,L,M`). 
The binary data of the N-dimensional array is then serialized in the **row-major** format 
(similar to C, C++, Javascript or Python) order, or, in the **column-major** format
as in MATLAB for FORTRAN if using the following BJData Draft-3 (or newer) syntax

```
[[] [$] [type] [#] [[] [[] [$] [nx type] [#] [ndim type] [ndim] [nx ny nz ...] []] [nx*ny*nz*...*sizeof(type)]
```
6. optimized container for packed object storage via structure-of-array (SOA) introduced
in Draft-4.

As a special note, all BJData/UBJSON integer and floating-point types must be
stored in the **Little-Endian** format, according to 
[BJData Specification (Draft-4)](https://neurojson.org/bjdata/).


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
  JSON/binary-JSON objects in the form of a CJSON
* **named node**: a node in the form of `"name":{...}` or `"name":[]` in the case 
  of named and index leaves and branches, respectively, or `"name":value` in the case
  of a leaflet
* **indexed node**: a node in the form of `{...}` or `[]` in the case of named 
  and index leaves and branches, respectively, or a single `value` in the case
  of a leaflet
* **a structure**: a named or index node made of named sub-nodes, represented by `{...}`; 
  a structure can be empty
* **an array**: a named or index node made of indexed sub-nodes, represented by `[...]`;
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
the "primary" data. The auxiliary data can appear at any level in the data annotation 
hierarchy.

The interpretations for "meaningful datum", "logically connected data", 
"primary data" and "weakly-relevant or irrelevant data" are application dependent.

The data group annotations are ranked in the below order, from highest 
 to lowest level:

` super-root > root >  data group >= dataset >= data record >= leaf > leaflet `

Each annotation level only contain annotated  elements of the same 
or lower levels, but not upper level annotations. All data annotation
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
also be present in any branch or leaf.

Below is a short summary of the JData data annotation/storage keywords that can be introduced

* **Data grouping**: `_DataGroup_`, `_Dataset_`, `_DataRecord_`
* **N-D Array**: `_ArrayType_`, `_ArraySize_`, `_ArrayIsComplex_`, `_ArrayIsSparse_`,
  `_ArrayData_`, `_ArrayLabel_`, `_ArrayCoords_`, `_ArrayUnits_`, `_ArrayFillValue_`,
  `_ArrayShape_`, `_ArrayOrder_`, `_ArrayChunks_`, `_ArrayZipType_`,
  `_ArrayZipSize_`, `_ArrayZipData_`, `_ArrayZipEndian_`, `_ArrayZipLevel_`,
  `_ArrayZipOptions_`, `_ArrayShuffle_`
* **Hash/Map**: `_MapData_`
* **Table**: `_TableData_`, `_TableCols_`, `_TableRows_`, `_TableRecords_`,
  `_TableIndex_`, `_TableSortOrder_`
* **Enumeration**: `_EnumKey_`, `_EnumValue_`, `_EnumOrdered_`
* **Tree**: `_TreeData_`,`_TreeNode_`,`_TreeChildren_`
* **Linked List**: `_ListNode_`,`_ListNext_`,`_ListPrior_`,`_LinkedList_`,`_ListLength_`
* **Graph**: `_GraphData_`,`_GraphNodes_`,`_GraphEdges_`,`_GraphEdges0_`,`_GraphMatrix_`
* **Byte-stream**: `_ByteStream_`
* **Inline metadata**: `"item_name::Property1=value1,Property2=value2,...": ...`
* **Metadata record**: `{"_DataInfo_":{...}}`, `{"_DataSchema_":{...}}`
* **Data links and anchors**: `_DataLink_`, `_DataAnchor_`

### Data Group Keywords

The use of data grouping keywords is not mandatory in a JData document. 
Nonetheless, properly partitioning and grouping the data based on their semantic 
relationships and naming the components accordingly can greatly enhance the 
portability and readability of the data, and thus are strongly recommended.

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
string, among items under the same parent object, must be provided 
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
if one needs to define the `indexed_node2` as a data group, one must define
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

Also similarly, a data record is specified by

`"_DataRecord_": { ... }`

or

` "_DataRecord_": [ ... ]`

with an optional name parameter in the following format:

`"_DataRecord_(a name string)"`


### Metadata

Metadata records can be associated with a data group, dataset or data record
(or any branch or leaf). It is optional. Metadata can have two forms

* **inline metadata**: metadata defined as part of the annotation tag string
* **metadata node**: a metadata structure added as the first element of a node (named or indexed)

The inline metadata is defined by attaching a series of comma-separated `property=value` 
strings to the data group keywords (`_DataGroup_`,`_Dataset_`,`_DataRecord_`), separated by 
two colons, i.e. `"::"`. For example

`"_{DataGroup,Dataset,DataRecord}_(name)::Property1=...,Property2=...,..."`

The `property` name should not contain spaces or commas. If commas appear inside the `value`,
it must be escaped using the form `"\,"`. 

Inline metadata can be attached to any named node, for example

`"name::Property1=...,Property2=...,..."`

One or more permitted white spaces, see above, may be inserted before or after 
separators `"::"`, `","` and `"="` without changing the interpretation of the data.

The inline metadata is primarily used to store simple properties. When storage of more complex 
metadata is needed, a dedicated "metadata node" should be inserted to the data object as the
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
The property names are user-defined. Recommended properties include, but are not limited
to, the following list
```
  Version       - version string of the data or the generating software
  Author        - name(s) of the data creator(s)
  Comment       - free-text description
  UniqueID      - a globally unique identifier (e.g., UUID v4)
  CreateTime    - ISO 8601 creation timestamp (e.g., "2024-03-15T14:32:00Z")
  ModifiedTime  - ISO 8601 last-modification timestamp
  License       - SPDX license identifier (e.g., "CC-BY-4.0", "Apache-2.0")
  GeneratedBy   - name and version of the software that produced the data
  DerivedFrom   - URI or identifier of the source data from which this was derived
  SourceFormat  - original file format before conversion (e.g., "NIfTI-1", "HDF5")
```

In software environments where leading-underscore key names cannot be defined at the
top level -- such as the root of a document in
[Apache CouchDB](https://couchdb.apache.org/) -- the alternative names `".datainfo"`
and `".dataschema"` may be used in place of `"_DataInfo_"` and `"_DataSchema_"`
(see below), respectively.

A complementary keyword `"_DataSchema_"` may optionally be added to any branch or leaf
to associate a [JSON Schema](https://json-schema.org/) document that formally describes
the structure and data types of the annotated node.  The value of `"_DataSchema_"` must
be either a valid JSON Schema object or a URI string pointing to an external schema
document.  `"_DataSchema_"` supplements `"_DataInfo_"` and the two may coexist.  For
example:
```
  {
    "_DataInfo_":   { "Version": "1.0", "Author": "Jane Doe" },
    "_DataSchema_": {
      "$schema": "https://json-schema.org/draft/2020-12/schema",
      "type": "object",
      "properties": { "Age": { "type": "integer", "minimum": 0 } }
    },
    "Age": 30
  }
```
Parsers that do not recognize `"_DataSchema_"` must silently ignore it; the keyword
carries no data payload and does not affect decoding of sibling fields.

### Data Storage Keywords

JData is designed to store a wide variety of data forms. The most common data structures
used in the scientific communities include scalars, constants, arrays, structures, 
tables, associative arrays, trees, and graphs.

#### Special constants

The following constants are supported by this version of the specification

* `NaN`: An (integer or floating-point) `NaN` defined by the [IEEE 754 standard](https://en.wikipedia.org/wiki/NaN) 
  shall be stored as a `"_NaN_"` string leaflet in text-based JData, unless it is part of an 
  array stored in the [annotated N-D array format](#annotated-storage-of-n-d-arrays); 
  in binary JData, it should be stored in the respective (integer or floating-point) IEEE 754 format
* `+/-Inf`: A `+infinity` or `-infinity` defined by the IEEE 754 standard shall be stored as 
  string leaflets `"+_Inf_"` and `"-_Inf_"`, respectively, in the text-based JData ("+" sign
  can be omitted), unless it is part of an array stored in the [annotated N-D array format](#annotated-storage-of-n-d-arrays); 
  in the binary JData, they should be stored in the respective (integer or floating-point) IEEE 754 format
* logical `true`/`false`: A logical `true`/`false` should be represented by the JSON `true/false` 
  logical values, or `[T]`/`[F]` markers in BJData/UBJSON
* `Null`: a `Null` (empty) value shall be stored as `null` in JSON and `[Z]` in BJData/UBJSON


#### N-Dimensional Array Storage Keywords

An N-dimensional array is serialized using the **row-major** format (i.e. the fastest index
is the right-most index, similar to arrays in C, C++ or Python; in comparison, MATLAB and 
FORTRAN arrays are column-majored).

A solid (non-sparse) N-dimensional array can be represented in two forms in JData - the 
direct storage format and the annotated storage format.

##### Direct storage of N-D arrays

A solid N-D array can be stored directly using JSON/binary-JSON nested array constructs. For example, 
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
such information can be stored when using the BJData/UBJSON format. 

Please be aware that the BJData format supports an extended syntax compared to UBJSON
to facilitate storage and loading of N-D arrays in the binary format. Please refer to the 
"Grammar" section for details.

##### Annotated storage of N-D arrays

In the annotated array storage format, one shall use a structure to store an N-D array. This
allows one to use additional fields to store additional information regarding the encoded 
array.

The annotated array format is shown below for a solid 3x2 array `a=[[1,2],[3,4],[5,6]]`
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
* **`"_ArrayOrder_"`**: (optional) a case-insensitive string value denoting the array serialization 
  order: `"r"` or `"row"` for "row-major" order (as in C/C++/Python) and "c", "col" or "column" for "column-major" 
  order (as in MATLAB/FORTRAN); if missing, JData assumes the **row-major** order as default.
* **`"_ArrayData_"`**: (required) a 1-D row vector (or a rectangular array, see below) storing the serialized 
  array values, assuming the **row-major** element order if `"_ArrayOrder_"` is not specified.
* **`"_ArrayLabel_"`**: (optional) must be a 1-D array with elements equal or less than the dimensions
  of the array, which is the length of `"_ArraySize"`. The first element defines the name or label, in
  the form of a string, for the 1st dimension, the 2nd element defines the name for the 2nd dimension,
  and so on. If the label of a dimension is an empty string `""`, it is undefined. If any of the element
  is an array, it further defines the names/labels for the array indices along this dimension. This array
  must be in the form `[["label_1", column_start_1, column_width_1], ["label_2", column_start_2, ...]]`,
  where optional integers `column_start_i` and `column_width_i` define the start and width, respectively,
  of the array indices that are associated with this label.
* **`"_ArrayCoords_"`**: (optional) a 1-D JSON array whose length equals the length of
  `"_ArrayLabel_"` (which must be present when `"_ArrayCoords_"` is used).  Element `i`
  is the coordinate vector for the `i`-th dimension (in `"_ArrayLabel_"` order) and must
  have a length equal to `"_ArraySize_"[i]`.  Each element may be either:
  1. a plain 1-D JSON array of numeric or string values; or
  2. an annotated JData array object (with `"_ArrayType_"`, `"_ArraySize_"`, etc.).
     When `"_ArrayShape_":"range"` is used, `"_ArrayData_"` must be a 2-element vector
     `[start, end]` (inclusive on both ends) and `"_ArraySize_"` must equal the
     dimension size.

  For example:
  ```
    "_ArrayLabel_": ["x", "t"],
    "_ArrayCoords_": [
        [0.0, 0.5, 1.0, 1.5, 2.0],
        ["2024-01-01", "2024-01-02", "2024-01-03"]
    ]
  ```
* **`"_ArrayUnits_"`**: (optional) a single string, or a 1-D array of strings with length
  equal to the number of dimensions (i.e., `length(_ArraySize_)`).  Each string specifies
  the physical unit of the corresponding dimension following the UDUNITS-2 convention
  (e.g., `"mm"`, `"ms"`, `"kg/m^3"`).  A single string applies uniformly to all
  dimensions.  An empty string `""` denotes a dimensionless or undefined unit.
* **`"_ArrayFillValue_"`**: (optional) a scalar of the same numeric type as `"_ArrayType_"`
  representing missing or invalid entries.  Parsers should treat array elements equal to
  this value as missing data.  This supplements the IEEE 754 `NaN` convention for
  integer-typed arrays where `NaN` cannot be represented (e.g., a sentinel value of
  `65535` for a `uint16` array).  If both `"_ArrayFillValue_"` and `NaN` entries coexist
  in a floating-point array, both shall be treated as missing.
* **`"_ArrayChunks_"`**: (optional) a 1-D integer array specifying the tile (chunk) shape
  for partitioning the **pre-processed array** -- that is, the rectangular data buffer that
  would otherwise be stored in `"_ArrayData_"` after any sparse, complex, or shape
  transformations have been applied (see the [Compressed array storage format](#compressed-array-storage-format)
  section for the definition of the pre-processed array).  The length of `"_ArrayChunks_"`
  must equal the number of dimensions of the pre-processed array (1, 2, or 3), not
  necessarily the number of logical dimensions in `"_ArraySize_"`.

  Chunking at the pre-processed level ensures that each chunk is a self-contained
  rectangular block that can be decompressed independently, without requiring knowledge
  of `"_ArrayIsComplex_"`, `"_ArrayIsSparse_"`, or `"_ArrayShape_"`.  Those flags apply
  only after all chunks have been reassembled into the full pre-processed array.

  When `"_ArrayChunks_"` is present, `"_ArrayData_"` (or `"_ArrayZipData_"` for
  compressed data) becomes a 1-D JSON array whose elements are the individual chunk
  payloads, ordered in row-major (C) sequence across the pre-processed array dimensions.
  The last chunk along any dimension may be smaller than the declared chunk shape if the
  array extent is not evenly divisible.  `"_ArrayZipType_"`, when present, applies
  uniformly to every chunk.  `"_ArrayZipSize_"` **must** be present alongside
  `"_ArrayChunks_"` and stores the shape of the **full pre-processed array** (not the
  chunk shape); the decoder uses it together with `"_ArrayChunks_"` to determine the
  number of chunks per dimension (`ceil(_ArrayZipSize_ / _ArrayChunks_)`) and the
  actual shape of each tile, including partial boundary tiles.  For example, a
  100x100 array stored in 32x32 tiles with zlib compression:
  ```
    {
        "_ArrayType_":    "single",
        "_ArraySize_":    [100, 100],
        "_ArrayChunks_":  [32, 32],
        "_ArrayZipType_": "zlib",
        "_ArrayZipSize_": [100, 100],
        "_ArrayZipData_": [
            "<base64-chunk-0-0>", "<base64-chunk-0-1>", "<base64-chunk-0-2>",
            "<base64-chunk-0-3>", "<base64-chunk-1-0>", ...
        ]
    }
  ```
  The total number of chunks is `prod(ceil(_ArrayZipSize_ ./ _ArrayChunks_))` = 16
  (4×4 tiles); the boundary tiles in the last row and column each have extent 4
  (`100 - 3×32`) along the respective dimension.

To facilitate the pre-allocation of the buffer for storage of the array in the parser, when
an ordered object or map is used to store an array, it is recommended that the `"_ArrayType_"`,
`"_ArraySize_"` and `"_ArrayOrder_"` (if present) nodes must appear before `"_ArrayData_"`.

In the case of complex-valued or sparse arrays or the presence of `"_ArrayShape_"` (see below), 
`"_ArrayData_"` may contain a 2-D rectangular array to store the pre-processed array elements. 
One can further serialize such 2-D `"_ArrayData_"` into a 1-D vector using the **row-major** order 
and then add `"_ArrayZipSize_": [Nx, Ny]` to store the 2-D `"_ArrayData_"` dimensions before
1-D serialization.

The supported data types are similar to those supported by the [BJData/UBJSON format](https://github.com/NeuroJSON/bjdata/blob/master/Binary_JData_Specification.md#type_summary), i.e.

* **uint8**: unsigned byte (8-bit), `[U]` in BJData/UBJSON
* **int8**: signed byte (8-bit), `[i]` in BJData/UBJSON
* **uint16**: unsigned short (16-bit), `[u]` in BJData, no correspondence in UBJSON
* **int16**: signed short (16-bit), `[I]` in BJData/UBJSON
* **uint32**: unsigned integer (32-bit), `[m]` in BJData, no correspondence in UBJSON
* **int32**: signed integer (32-bit), `[l]` in BJData/UBJSON
* **uint64**: unsigned long long integer (64-bit), `[M]` in BJData, no correspondence in UBJSON
* **int64**: signed long long integer (64-bit), `[L]` in UBJSON
* **half**: half-precision floating point (16-bit), `[h]` in BJData, no correspondence in UBJSON
* **single**: single-precision floating point (32-bit), `[d]` in BJData/UBJSON
* **double**: double-precision floating point (64-bit), `[D]` in BJData/UBJSON

The first 8 data types are considered "integer" types, and the last three types are considered 
"floating-point" types.

In addition, the below types can be used as aliases in an annotated array format:

* **byte**: byte arrays (8-bit), `[B]` in BJData, no correspondence in UBJSON
* **char**: character arrays (8-bit), `[C]` in BJData/UBJSON
* **logical**: logical arrays (8-bit), `[T]`/`[F]` in BJData/UBJSON

If the above aliases are used, a parser may optionally convert the enclosed
`uint8` data to byte, character or logical arrays (1-byte).

The following aliases are accepted for the three floating-point types to improve
interoperability with Apache Arrow, NumPy, and related toolchains:

* **float16**: alias for `half`   (16-bit floating point)
* **float32**: alias for `single` (32-bit floating point)
* **float64**: alias for `double` (64-bit floating point)

A parser encountering `float16`, `float32`, or `float64` must treat them identically
to `half`, `single`, and `double`, respectively.  The canonical names `half`, `single`,
and `double` remain preferred in newly generated JData files.

The following `"_ArrayLabel_"` example defines a 4D array with the first dimension with a name of `"x"` and length of 5;
the 2nd dimension is named as `"y"` of length 5, the 3rd dimension as `"z"` of length 6, and the 1st column of the 4th 
dimension is `"t"`, and the 2nd column of the 4th dimension is `"v"`.
```
   {
       "_ArrayType_": "double",
       "_ArraySize_": [5, 5, 6, 2],
       "_ArrayLabel_": ["x", "y", "z", [["t", 1], ["v", 2]]]
       "_ArrayData_": [...]
   }
```

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

The `"_ArrayIsComplex_"` node is recommended to appear before `"_ArrayData_"` when an ordered 
object or map is used to store such data.

##### Sparse arrays

JData also supports the storage of N-D sparse arrays using the annotated N-D array storage
format. In this case, a named node `"_ArrayIsSparse_":true` must be added into 
the structure. In the meantime, the `"_ArrayData_"` shall be defined as a 2-D array by 
concatenating the N-tuple integer indices (left-most index first, and so on), each as a
row vector, followed by the row vector of the serialized non-zero array element 
values. For an N-D sparse array, each non-zero value requires an N-tuple index to specify 
its location.

For example, if a 3-D sparse array has the following non-zero elements at the specified 
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
each as a row vector, followed by a row vector for the real-values of the non-zero elements, and ended
lastly with the row vector for imaginary-values of the non-zero elements. All row vectors
of `"_ArrayData_"` must have the same length.

For example, if a 3-D sparse array has the following non-zero complex elements at the 
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
or the corresponding BJData equivalents.


##### Special matrices

JData can efficiently store a list of special matrices via the `"_ArrayShape_"` descriptor.

The `"_ArrayShape_"` descriptor can be used in conjunction with `"_ArrayIsComplex_"`,
but it shall not be used when `"_ArrayIsSparse_"` is set to `true`. The `"_ArrayShape_"` 
node is recommended to appear before `"_ArrayData_"` if present.

The `"_ArrayShape_"` field shall be either in the form of

`"_ArrayShape_": "shapeid"`

or

`"_ArrayShape_": ["shapeid", param1, param2, ...]`

Here, the `"shapeid"` tag is a case-insensitive string specifying the type of the
special matrix. The currently supported `"shapeid"` values include

* `"diag"`: a diagonal matrix, can be a non-square matrix (optional `param1` 
  defines the length of the diagonal elements, must not be more than the smallest 
  value in the `_ArraySize_` vector)
* `"upper"`: an upper triangular (square) matrix (2-D only)
* `"lower"`: a lower triangular (square) matrix (2-D only)
* `"uppersymm"`: a symmetric (square) matrix, only storing the upper triangle (2-D only)
* `"lowersymm"`: a symmetric (square) matrix, only storing the lower triangle (2-D only)

For the above array types, the array entries falling under the mask of the prescribed shape,
referred to as the "effective elements", are serialized as a single vector in the row-major 
order in `"_ArrayData_"`. For example, a 3x3 upper triangular matrix:
```
   a11  a12  a13
    0   a22  a23
    0    0   a33
```
can be stored as 
```
 {
   ...
   "_ArrayShape_": "upper",
   "_ArrayData_": [a11, a12, a13, a22, a23, a33]
 }
```

In addition, band matrices are supported via the below `"shapeid"` tags

* `"upperband"`: an upper-band matrix (2-D only, optional `param1` defines the 
    number of super-diagonals of the matrix)
* `"lowerband"`: a lower-band matrix (2-D only, optional `param1` defines the 
    number of sub-diagonals of the matrix)
* `"uppersymmband"`: a symmetric band-matrix by storing only the upper-band 
   (2-D only, optional `param1` defines the number of super-diagonals of the matrix)
* `"lowersymmband"`: a symmetric band-matrix by storing only the lower-band 
   (2-D only, optional `param1` defines the number of sub-diagonals of the matrix)
* `"band"`: a band matrix (2-D only, optionally, if only `param1` presents, it defines 
    both the upper and lower bandwidths while  if both `param1` and `param2` are given,
    they define the numbers of super-diagonals and sub-diagonals, respectively)

The [LAPACK-styled band-matrix storage method](https://www.netlib.org/lapack/lug/node124.html) 
is used here. In other words, for the superdiagonal band (the band above the diagonal), 
the band data are converted into a 2-D matrix with one diagonal line per row 
(front-padding arbitrary values if the element is outside of the matrix); a 
sub-diagonal band is converted to a 2-D matrix with one diagonal per row 
(rear-padding zeros if the element is outside of the matrix).

For example, the below band matrix has 2 super-diagonals, and 1 sub-diagonal, 
and 1 diagonal (thus, a total bandwidth of 2+1+1=4)

```
    a11 a12 a13  0   0   0
    a21 a22 a23 a24  0   0
     0  a32 a33 a34 a35  0
     0   0  a43 a44 a45 a46
     0   0   0  a54 a55 a56
     0   0   0   0  a65 a66
```
the above matrix can be stored as

```
 {
   ...
   "_ArraySize_": [6, 6],
   "_ArrayShape_": ["band", 2, 1],
   "_ArrayData_": [
      [0   0   a13 a24 a35 a46],
      [0   a12 a23 a34 a45 a56],
      [a11 a22 a33 a44 a55 a66],
      [a21 a32 a43 a54 a65  0 ]
   ]
 }
```

where the "0" entries are elements outside of the matrix and can have an arbitrary 
value. The dimensions of the `"_ArrayData_"` (as a 2-D array) are
`(total bandwidth)`-by-`(maximum array dimension)`, where 
   * `total bandwidth` = `param1`+ `param2` + 1
   * `maximum array dimension` = the largest number in the `"_ArraySize_"` vector

Moreover, Toeplitz and related structured matrices, as well as compact range arrays,
are supported via the below `"shapeid"` values:

* `"toeplitz"`: a Toeplitz matrix by storing only the first row and column
   (padding zeros to have the same length); if the optional `param1` is present, it
   shall denote `max(#super-diagonals+1, #sub-diagonal+1)`
* `"circulant"`: a circulant matrix, fully determined by its first row; `"_ArrayData_"`
   stores only the first row as a 1-D vector.  The `k`-th row is a cyclic right-shift
   of the first row by `k` positions.  Only square matrices are supported.
* `"hankel"`: a Hankel matrix defined by its first row and last column;
   `"_ArrayData_"` stores a 2-D array whose first row is the first row of the matrix
   and whose second row is the last column, both padded with zeros at the rear to the
   same length.  The first element of the first row and the last element of the second
   row must coincide (they share the corner element).
* `"identity"`: an identity (or scaled-identity) matrix; `"_ArrayData_"` must be a
   scalar equal to the diagonal value (1 for a standard identity matrix).  The full
   matrix is reconstructed as `scalar * I`.  Only square matrices are supported.
* `"zero"`: a zero matrix (all entries are zero); `"_ArrayData_"` must be `[0]` or
   may be omitted entirely.  No data storage is required beyond `"_ArraySize_"`.
* `"range"`: a uniformly spaced array (or N-D grid of independent axes), stored as
   a compact `[start, end]` pair per dimension rather than materializing all element
   values.  The boundaries are **inclusive** on both ends (MATLAB convention), so
   the `i`-th reconstructed value along a dimension of size `N` is
   `start + (end - start) * i / (N - 1)` for `i = 0 ... N-1`.
   The element count `N` along each dimension is given by the corresponding entry in
   `"_ArraySize_"`, which remains required.

   For a 1-D range, `"_ArrayData_"` is a 2-element vector `[start, end]`:
   ```
     {
         "_ArrayType_":  "double",
         "_ArraySize_":  [101],
         "_ArrayShape_": "range",
         "_ArrayData_":  [0.0, 100.0]
     }
   ```
   This encodes the 101-element vector `[0, 1, 2, ..., 100]`.

   For an N-D range (separable grid), `"_ArrayData_"` is an `ndim x 2` matrix
   where row `i` is the `[start, end]` pair for the `i`-th dimension, matching
   the order of `"_ArraySize_"`.  For example, a 5x3 2-D grid spanning
   x in [0, 4] and y in [10, 12] is stored as:
   ```
     {
         "_ArrayType_":  "double",
         "_ArraySize_":  [5, 3],
         "_ArrayShape_": "range",
         "_ArrayData_":  [[0.0, 4.0], [10.0, 12.0]]
     }
   ```
   This encodes a grid whose x-axis is `[0, 1, 2, 3, 4]` and y-axis is
   `[10, 11, 12]`.  Each dimension is reconstructed independently.

   When `"_ArraySize_"[i]` equals 1, `start` and `end` must be equal for that
   dimension (a degenerate single-point axis), and the step is defined as zero.

   `"range"` must not be combined with `"_ArrayIsComplex_"` or `"_ArrayIsSparse_"`.
   It may be combined with `"_ArrayLabel_"`, `"_ArrayCoords_"`, `"_ArrayUnits_"`,
   and `"_ArrayFillValue_"` in the usual way.

For example, the below non-square (5-by-6) Toeplitz matrix with 3 effective values
in the row `[a11 a12 a13]` and 2 effective values in the column `[a11 a21]`
```
    a11 a12 a13  0   0   0
    a21 a11 a12 a13  0   0
     0  a21 a11 a12 a13  0
     0   0  a21 a11 a12 a13
     0   0   0  a21 a11 a12
```
shall be stored as
```
 {
   ...
   "_ArraySize_": [5, 6],
   "_ArrayShape_": ["toeplitz", 3],
   "_ArrayData_": [
      [a11, a12, a13],
      [a11, a21, 0]
   ]
 }
```
Notice that the first element of the row and the column vectors must be identical. The 2nd vector
in `"_ArrayData_"` is padded in the rear to match the length of the first vector, keeping
`"_ArrayData_"` in a rectangular shape.

The `param1` in the above example is optional, similar to in the band-matrix
cases. If it is not included, the parser shall read the first sub-vector in 
`"_ArrayData_"` to determine the effective element number.

##### Complex-valued special matrices

The `"_ArrayShape_"` descriptor can be used with `"_ArrayIsComplex":true` to store 
complex-valued special matrices. In such cases, the data stored in `"_ArrayData_"` 
shall add an extra left-most dimension (must be 2), and store the real and imaginary 
parts of the effective elements in the 1st and 2nd sub-matrices (of matching size), 
respectively.

For example, a complex-valued Toeplitz matrix of the same shape shall be stored as
```
 {
   ...
   "_ArraySize_": [5, 6],
   "_ArrayIsComplex_": true,
   "_ArrayShape_": ["toeplitz", 3],
   "_ArrayData_": [
      [
          [r11, r12, r13],
          [r11, r21, 0]
      ],
      [
          [i11, i12, i13],
          [i11, i21, 0]
      ]
   ]
 }
```
where `r` and `i` values denote the real and imaginary parts of the effective values
in the Toeplitz matrix.

In addition, when `"_ArrayIsComplex":true` is present, the `"_ArrayShape_"` `"shapeid"`
can take the below additional values to represent "Hermitian matrices"

* `"upperherm"` (similar to `"uppersymm"`)
* `"lowerherm"` (similar to `"lowersymm"`)
* `"upperhermband"` (similar to `"uppersymmband"`)
* `"lowerhermband"` (similar to `"lowersymmband"`)

where the mirrored elements across the diagonal shall have opposite signs for the 
imaginary parts when decoding the array.


##### Compressed array storage format

JData supports node-based data compression to enhance the space-efficiency of generated
files. The compressed data format is only supported in the annotated array storage format.

For all array types supported in this specification, we want to point out that the 
encoded data stored in the `"_ArrayData_"` container is always rectangular (1-D, 2-D, 
or 3-D) in shape. This allows us to efficiently store and parse the data payload and
apply compression. We refer to the data stored in `"_ArrayData_"` as the 
**"pre-processed" array**.

Four additional nodes are added to the annotated array structure

* **`_ArrayZipType_`**: (required) a case-insensitive string-valued leaflet identifying
  the compression codec.  To ensure unambiguous codec identification across parsers,
  the normative identifiers follow the
  [Numcodecs codec registry](https://numcodecs.readthedocs.io/en/stable/), which is
  also adopted by Zarr.  The recommended identifiers are:

  | Identifier          | Description                                        |
  |---------------------|----------------------------------------------------|
  | `"zlib"`            | zlib/DEFLATE stream (RFC 1950)                     |
  | `"gzip"`            | gzip file format (RFC 1952)                        |
  | `"bz2"`             | bzip2 compression                                  |
  | `"lzma"`            | LZMA / XZ compression                              |
  | `"zstd"`            | Zstandard compression (RFC 8878)                   |
  | `"lz4"`             | LZ4 block compression                              |
  | `"blosc2"`          | Blosc2 meta-compressor (default internal codec)    |
  | `"blosc2lz4"`       | Blosc2 with LZ4 internal codec                     |
  | `"blosc2lz4hc"`     | Blosc2 with LZ4-HC internal codec                  |
  | `"blosc2blosclz"`   | Blosc2 with BloscLZ internal codec                 |
  | `"blosc2zstd"`      | Blosc2 with Zstandard internal codec               |
  | `"blosc2zlib"`      | Blosc2 with zlib internal codec                    |
  | `"base64"`          | Base64 encoding only (no compression)              |

  Note: `"zlib"` and `"gzip"` are distinct formats sharing the DEFLATE algorithm
  but differing in header/trailer structure; parsers must not treat them as synonyms.
  JData supports only Blosc2 (not the older Blosc v1 library).  When the Blosc2
  internal codec is unspecified, `"blosc2"` defaults to the BloscLZ codec.  The
  `"_ArrayZipOptions_"` field may be used to pass additional Blosc2 parameters such
  as `typesize`, `clevel`, `shuffle`, and `nthreads`.  Additional codecs not listed
  above may be used; their identifiers should follow the Numcodecs naming convention
* **`_ArrayZipSize_`**: (required) the dimensions of the **pre-processed array**, i.e. the data
  originally stored in `_ArrayData_` in the format specified by `"_ArrayType_"`, 
  before the array binary stream is type-casted to byte stream and compressed.
* **`_ArrayZipData_`**: (required) in addition, the `"_ArrayData_"` node is replaced by 
  `"_ArrayZipData_"`. In the case of JSON-formatted JData files, 
  `"_ArrayZipData_"` has a string value storing the "Base64" encoded compressed 
  byte-stream of the pre-processed array. In the case of binary flavored JData, 
  `"_ArrayZipData_"` directly stores the compressed byte stream of the 
  pre-processed array without "Base64" encoding.

In addition, the following optional parameters may also be used

* **`_ArrayZipEndian_`**: (optional) a case-insensitive string, either "little" or "big",
  indicating the endianness of the byte-stream before compression; if missing, assumed
  to be "little"
* **`_ArrayZipLevel_`**: (optional) a numerical value, typically an integer between
  0 and 9, specifying the level of the compression (interpretation is method/library-dependent)
* **`_ArrayZipOptions_`**: (optional) an array object allowing users to specify
  additional compression-method specific parameters (interpretation is method/library-dependent)
* **`"_ArrayShuffle_"`**: (optional) if present, must be a non-zero integer; a positive
  integer specifying the number of bytes at which the serialized raw data buffer is
  shuffled before compression.  For example, `"_ArrayShuffle_": 4` reorders a
  byte stream `[1,2,3,4,5,6,7,8,9,10,11,12]` into
  `[1,5,9,2,6,10,3,7,11,4,8,12]`.  A negative integer specifies a bit-wise shuffle
  with the absolute value as the bit-wise shuffle spacing.  `"_ArrayShuffle_"` must
  be applied, if present, **before** compression/encoding; an unshuffle operation must
  be applied upon decompression/decoding.

When a compressed array format is encoded in an ordered object, `"_ArrayZipType_"` and 
`"_ArrayZipSize_"` is recommended to appear before `"_ArrayZipData_"`.

`"_ArrayZipData_"` and `"_ArrayData_"` can not appear under the same parent node.

#### Associative arrays or maps

In a map or an associative/hashed array, the element can be accessed using a unique
key. For example, the below pseudo code defines a 3-element associative array `@a` with
3 unique keys
```
   @a = {'Andy'->21, 'William'->21, 'Om'->22}
```
Such data structure can be conveniently represented using JSON/binary JSON as
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
Such data can also be stored in JSON/binary JSON using 3 dedicated keywords
* **`"_TableCols_"`**: a 1-D array denoting the name and type for each of the column of the table, 
     can be empty if no name or types are associated with the columns
* **`"_TableRows_"`**: a 1-D array denoting the name and type for each of the row of the table
     can be empty if no name or types are associated with the rows
* **`"_TableRecords_"`**: a 2-D array storing each entry in the 2-D table.

For example, the above table can be serialized using the below format
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

* `"string"`: a UTF-8 encoded string (NFC normalization recommended per Unicode Annex #15)
* `"bool"`: a boolean: either `true` or `false`
* `"blob"`: a binary byte-stream; the format of the `blob` record is described in the
  [Generic byte-stream data](#generic-byte-stream-data) section (without the
  `"_ByteStream_"` container)
* `"datetime"`: an ISO 8601 date-time string (e.g., `"2024-03-15T14:32:00Z"`); parsers
  should interpret values without an explicit time-zone offset as UTC

Two additional optional keywords are defined for tables:

* **`"_TableIndex_"`**: (optional) a string or 1-D array of strings naming one or more
  columns in `"_TableCols_"` that together form the unique row index (analogous to a
  primary key in SQL or `DataFrame.index` in pandas).  For example,
  `"_TableIndex_": "Name"` or `"_TableIndex_": ["LastName", "FirstName"]`.
  If absent, the implicit row index is the 1-based row position.
* **`"_TableSortOrder_"`**: (optional) a 1-D array of column name strings declaring the
  sort order of the stored records.  A column name prefixed with `"-"` denotes
  descending order (e.g., `"_TableSortOrder_": ["Age", "-Height"]` declares ascending
  `Age`, then descending `Height`).  Parsers may rely on this annotation to skip
  re-sorting for binary-search or merge operations.

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

The choice of forms is user-dependent. Typically, the "structure of arrays" form 
leads to a smaller file size.

The above JData table can be enclosed inside an optional field `"_TableData_(table_name)":{...}` 
(if inside a structure) or `{"_TableData_":{...}}` (if inside an array) to inform the 
parser about the start of the data structure.

#### Enumerations

An enumeration data structure is spatially effective when storing array data containing
repeated values. For example, the following array
```
"gender": ["M","M","F","F","F","M",...,"F"]
```
contains only two possible values, `"M"` or `"F"`, each repeating many times. We use the following keywords to represent such data:

* `"_EnumKey_"`: an array containing the unique values (categories) that appear in the
  data; values may have mixed data types.
* `"_EnumValue_"`: an integer array with each entry denoting the **1-based** index of
  the corresponding value in `"_EnumKey_"` (the first entry of `"_EnumKey_"` has index 1);
  `"_EnumValue_"` can be an N-D array.  Note that this 1-based convention differs from
  the 0-based indexing used by JSONPath and Apache Arrow `DictionaryArray`; parsers
  converting to those formats must subtract 1 from each `"_EnumValue_"` entry.
* `"_EnumOrdered_"`: (optional) a boolean flag; when `true`, the categories in
  `"_EnumKey_"` are treated as an **ordered** (ordinal) sequence, where the position in
  `"_EnumKey_"` defines the ranking (first entry is lowest).  This is equivalent to
  `ordered=True` in a pandas `CategoricalDtype` or an R ordered factor.  If absent or
  `false`, the enumeration is unordered (nominal).

With the above construct, we can store the above `"gender"` example as
```
"gender": {
  "_EnumKey_": ["M","F"],
  "_EnumValue_": [1,1,2,2,2,1,...,2]
}
```
An ordered categorical (e.g., severity levels) is stored as
```
"severity": {
  "_EnumKey_":     ["low", "medium", "high"],
  "_EnumOrdered_": true,
  "_EnumValue_":   [1,3,2,1,...,3]
}
```
Because `"_EnumValue_"` is an integer array, one can further use the
[N-D array annotation format](#compressed-array-storage-format) to compress the data
and reduce the storage overhead.


#### Trees

A tree-like data structure can be conveniently represented by JSON/binary-JSON formatted files
due to the similar underlying structures. In JData, we use two keywords to encapsulate a 
tree-like data structure. 

The tree-node name and the associated data are stored using a named node with a keyword
`_TreeNode_`:

`"_TreeNode_(node_name)": node_data`

If a tree-node contains children, we use a named array to store the children data:

```
"_TreeChildren_":[
     {"_TreeNode_(child1)": data1},
     {"_TreeNode_(child1)": data2},
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
and the corresponding binary JSON equivalents. The notations "data0", "data1", etc. are parsed
node data in JData format according to the rules defined in this section and depend on
the type of the data.

One can add either inline or dedicated metadata records to store additional information
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
as named nodes as long as the name of the auxiliary node does not conflict with
the `"_TreeNode_(name)"` and `"_TreeChildren_"` of the same level. Behaviors for parsing 
the auxiliary data are application dependent.

The above JData tree can be enclosed inside an optional field `"_TreeData_(tree_name)":{...}` 
(if inside a structure) or `{"_TreeData_":{...}}` (if inside an array) to inform the 
parser of the start of the data structure.



#### Singly and doubly linked lists

Similar to the storage of trees, we use additional keywords to encapsulate a singly 
or doubly linked list inside a JSON/binary-JSON array construct. The relevant keywords are

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
If a node does not have a next or prior element, the `"_ListNext_"` or `"_ListPrior_"` value
should be set to `null`.

An optional `"Length"` integer annotation may be added as inline metadata on the
`"_LinkedList_"` container to declare the total number of nodes, enabling parsers to
pre-allocate the appropriate buffer before traversal.  For example:
```
"_LinkedList_(measurements)::Length=4": [...]
```

The name label referred to in the `"_ListNext_"` or `"_ListPrior_"` fields has a scope
limited to the current list, i.e. the parent array one-level above the node. Multiple linked lists can 
share the same names if they are stored in a parallel or nested fashion.

The above linked list can be enclosed inside an optional field, `"_LinkedList_(list_name)":[...]` 
(if inside a structure) or `{"_LinkedList_":[...]}` (if inside an array), to inform the parser of 
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

By default, the graph is assumed to be a directed graph. If a user intends to store an
undirected graph, the edge list must be enclosed in `"_GraphEdges0_"` instead of
`"_GraphEdges_"`.  When `"_GraphEdges0_"` is used, each edge `[A, B, ...]` implies the
reverse edge `[B, A, ...]`; parsers must not double-store the edge data.  The adjacency
matrix stored in `"_GraphMatrix_"` is interpreted as symmetric when paired with
`"_GraphEdges0_"`.

The above graph data can be enclosed inside an optional field, `"_GraphData_(graph_name)_":{...}` 
(if inside a structure) or `{"_GraphData_":{...}}` (if inside an array), to inform the parser of the 
start of the data structure.

When no data or metadata are associated with the edges, one can also use the graph's adjacency 
matrix to represent the connections between graph nodes. The adjacency matrix must only have 0 
and 1 values, with a 1 at A(i,j) indicating a directed edge from the i-th node in the 
`_GraphNode_` list to the j-th node in that list and a 0 indicating no connection.

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
we can write this matrix using the direct form

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
             "_ArrayType_": "uint8",
             "_ArraySize_": [4,4],
             "_ArrayIsSparse_": true,
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
zlib-compressed and finally base64-encoded (if stored in JSON, for binary JSON, 
base64 is not needed) adjacency matrix.

In addition, a weighted graph can also be stored using the adjacency matrix by replacing
the "1"s in the matrix by the weight values (a numerical scalar).


#### Generic byte-stream data

A binary byte-stream of arbitrary length can be stored in the text-based JData using the below construct

`"_ByteStream_":"..."`

where the string value `"..."` shall be the Base64-encoded byte-stream binary data. For example, a string
valued byte-stream `"JData specification"` shall be stored as

`"_ByteStream_":"SkRhdGEgc3BlY2lmaWNhdGlvbg=="`

In the binary JData, a byte-stream shall be encoded using a similar `"name":value` pair where the 
`value` is represented by an `[H]` marker. As specified in the 
[BJData Specification (Draft 3)](https://github.com/NeuroJSON/bjdata/blob/master/Binary_JData_Specification.md#high-precision), 
the `[H]` marker is immediately followed by the length of the byte stream, then followed by 
the raw binary values of the byte-stream **without Base64 encoding**. The same example above 
can be stored as

`[U][12][_ByteStream_][H][U][19][JData specification]`

The parsing and interpretation of the byte-stream data are application dependent. This is the most 
generic form of data storage but contains the least data semantic information. One can use this 
construct as a container to binary files, data segments, or encrypted data or files. In the case 
of data encryption, additional information related to the encryption and decryption, if needed, 
shall be stored as metadata to the `"_ByteStream_"` object with a format specified in the 
[Metadata section](#metadata).

An example `_ByteStream_` record that includes additional information in the metadata section is
shown below:

```
"_ByteStream_":{
       "_DataInfo_":
           "MediaType": "text/plain",
           "Encoding": ["zlib","base64"],
           "ByteLength": 1250,
       },
       "Data": "..."
}
```
where `MediaType` specifies the **original** data MIME type - in a format compatible to those 
used in common Internet standards (such as *RFC2045*, *RFC2046* etc), additional
encoding steps (specified by `"Encoding"`), in the order or processing, that converts the 
original data to the byte stream stored in the `"Data"` subfield, data length etc. 
For example, in the above `_ByteStream_` object, the `"Data"` subfield stores a plain-text 
object, by first apply a zlib-based compression to the text stream, and then apply "base64"
encoding. The total converted "Data" byte length is 1250-byte, as shown in `ByteLength`.


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

### Element Referencing

Essentially, JData stores a serialized version of complex data using collections 
of sequential or nested nodes, either in the named or indexed form. To access
any element (a leaflet, leaf or branch) of the JData document, one should use a vector 
of indices that points to the specific node. 

A JData-compliant parser must be able to retrieve JData elements via the below pseudo-code 
interface using an index vector or reference string
```
   JD_Node item = JD_GetNode(JD_Node root, [i1, i2, i3, i4, ...], is_compact)
      or
   JD_Node item = JD_GetNode(JD_Node root, "key")
```
where `i1` is the index of the data on the top-most level (relative to the root level of the 
`"root"` object), `i2`, is the index along the 2nd level, and so on. Each index is an 
integer, starting from 1, denoting the order
of the data element among the serialized elements of the same level. In order words, if
the current level is an array object, the index is the count of the elements before this data
element plus 1; if the current level is a structure, the index is the count of the named 
nodes appearing before this data plus 1. 

In some programming environments, such as C++ and Python, object keys may be unordered.
As a result, referencing using integer based index vector may not be reliable. An alternative
is [JSONPath](https://goessner.net/articles/JsonPath/). JSONPath uses a series of
dot-separated names to locate an element inside a JSON data tree, such as
```
$.name1.name2.name3[0].name4
```
in this notation, `$` refers to the root of the JSON object, `$.name1` refers to the root-level
sub-element `name1`; `$.name1.name2.name3[0]` specifies the `name2` child of `$.name1` has a child
named `.name3`, which is an array; using `.name3[0]` means taking the first element of the
array. Lastly, the first element of the `name3` array has a child named `.name4`.
When the object names contain `.` `[`, or `]`, they must be escaped by inserting a
`\` before. For example, `$.file.test\.json` specifies a key named `test.json` under the `$.file` object.

It is worth mentioning that JSONPath supports deep-scan (`..`) and filtering operators, 
although this specification does not require the parsers to fully support all JSONPath
operators.

Using the tree data structure above, the linear index
of each node is listed below on the right side:

```
  {
     "_TreeNode_(root)": data0,                     <- [1] or $._TreeNode_(root)
     "_TreeChildren_": [                            <- [2] or $._TreeChildren_
         {"_TreeNode_(node1)": data1},              <- [2,1] or $._TreeChildren_[0]
         {                                          <- [2,2] or $._TreeChildren_[1]
             "_TreeNode_(node2)": data2,            <- [2,2,1] or $._TreeChildren_[0]._TreeNode_(node2)
             "_TreeChildren_": [                    <- [2,2,2] or $._TreeChildren_[0].__TreeChildren_
                 {"_TreeNode_(node2.1): data2.1},   <- [2,2,2,1] or $._TreeChildren_[0].__TreeChildren_[0]
                 {"_TreeNode_(node2.2): data2.2}    <- [2,2,2,2] or $._TreeChildren_[0].__TreeChildren_[1]
             ]
         },
         {                                          <- [2,3] or $._TreeChildren_[2]
             "_TreeNode_(node3)": data3             <- [2,3,1], or [[2,3]] or $._TreeChildren_[2]._TreeNode_(node3)
         }
     ]
  }
```

The third parameter, `is_compact`, is a boolean flag. If set to `true`, `JD_GetNode` 
shall skip the index if any of the dimensions along the indexing vector is a singlet, 
i.e. the child count is 1. The compact indexing vector, enclosed by double-square-brackets 
as `[[...]]`, shall be passed to `JD_GetNode` as the 2nd input when `is_compact` is `true`.
Using the above example, both index vectors `[[2,3]]` and `[2,3,1]` refer to 
`"_TreeNode_(node3)": data3`. Please be aware that the compact indexing vector can not
distinguish between row and column vectors as the column vector in JData has a trailing 
singlet dimension ([see N-D array section](#direct-storage-of-n-d-arrays)).

An optional alternative indexing vector definition allows the replacement of the index within a 
structure by a corresponding string, which can be the name of the data item or a hashed version of it. 
For example, the item `{"_TreeNode_(node2.1): data2.1}` in the above may also be accessed 
via `["_TreeChildren_",2,"_TreeChildren_",1]`. This alternative indexing scheme is less 
sensitive to data serialization orders, but requires the parser to handle both string
and integer inputs.

The `JD_GetNode` API may also optionally provide supports to the widely used 
[JSON Path Notations](https://github.com/json-path/JsonPath) to query individual elements
in a JData/JSON based data structure.

### Data query

For each JData item identified via an indexing vector, a JData-compliant library must be able to
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

Combined with the `JD_GetName`, one can also query if the element is a named or indexed one.

Additionally, a JData-compliant library must provide the below interface to obtain the count of
the children for each data item. A leaflet, an empty structure or an empty array should return
a length of 0.
```
    integer length   = JD_GetLength(JD_Node item)
```


Converting Between JData Files
---------------------------------

One can choose either the text or binary format to save the raw data into
a JData file, with the former following JSON storage requirements and the 
latter following BJData/BJSON storage requirements. This specification permits
both lossless and lossy conversions between the raw data to JSON, raw data to 
BJData/UBJSON, and JSON and BJData/UBJSON files. 

* **lossless conversion**: the input data are preserved after a save-load 
  round-trip conversion to and from one of the JData formats; type-casting 
  from low precision to high precision types is permitted
* **lossy conversion**: the input data may lose precision during the 
  save-load round-trip conversion but the loss of precision shall be limited
  to a level that is tolerable by the application.

For best practices, the use of lossless conversion is highly recommended. Here are 
some general recommendations to best preserve data precision:

* Use binary-based JData to retain data type and binary information
* When using text-based JData, make sure to print sufficient decimals for 
  floating-point numerical data (typically 7 for single-precision and 17 for 
  double-precision values) to retain the full precision
* When saving numerical arrays to text-based JData, consider using annotated 
  array storage format, either with or without data compression, to store 
  binary type information
* When parsing numerical arrays stored in the direct storage format in text-based
  JSON, one should consider using double-precision as the read-buffer to avoid
  truncation of input data and loss of precision.

In addition, if a transformation of the data does not alter the (full or compact) 
indexing vector of all leaflets in a JData document, it is referred to as an 
**"isometric transform"** and is permitted. An example of an isometric transform 
is the conversion from a structure to an array as shown in the below example:
```
   {
      "a": {
         "name1": value1,      <- [1,1] or ["a","name1"] or $.a.name1
         "name2": value2,      <- [1,2] or ["a","name2"] or $.a.name2
         "name3": value3       <- [1,3] or ["a","name3"] or $.a.name3
      },
      "b": "value4"            <- [2] or ["b"] or $.b
   }
```
to
```
   {
      "a": [
         {
             "name1": value1   <- [[1,1]] or [["a","name1"]] or $.a[0].name1
         },
         {
             "name2": value2   <- [[1,2]] or [["a","name2"]] or $.a[1].name2
         },
         {
             "name3": value3   <- [[1,3]] or [["a","name3"]] or $.a[2].name3
         }
      ],
      "b": "value4"            <- [2] or ["b"] or $.b
   }
```
The only permitted "non-isometric transform" is the conversion between a direct
N-D array to an annotated N-D array.


Data Referencing and Links
---------------------------------

JData files support referencing and internal/external linking via the definition of
data links and anchors. 

A link is defined by a named leaflet or leaf as shown in the below two styles
```
   "link_style1": {
       "_DataLink_": "path"
   },
   "link_style2": {
       "_DataLink_": {
           "URI": "path",
           "Parameters": "$.key1.key2...",
           "MaxRecursion": 1
       }
   }
```
The `"path"` string specifies the Uniform Resource Identifier (URI) of the referenced 
JData data document using a format compliant to the [RFC3986] specification, followed 
by the indexing vector string to point to a specific element of the referenced document. 
For example, the below link
```
   {
      "_DataLink_": "file:///space/test/jdfiles/tree.jdat:$.key1.key2..."
   }
```
asks the parser to read a local file located at "/space/test/jdfiles/tree.jdat" and
load the node specified by JSONPath string `$.key1.key2...`, starting from the root (or super-root
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
      "_DataLink_": "$.key1.key2..."
   }
   "local_compact_link3": {
      "_DataLink_": [1,2,2]
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

For the text-based JData file, the recommended file suffix is **`".jdt"`**; for 
the binary-JSON based JData file, the recommended file suffix is **`".jdb"`**.

The MIME type for the text-based JData document is 
**`"application/jdata-text"`**; that for the binary JData document is 
**`"application/jdata-binary"`**


Summary
----------

The main appeals of JSON and binary-JSON are their simplicity and portability, which
are often missing from other alternatives. In this document, we aim to extend the ability
of JSON/binary-JSON to store and interchange complex data structures without needing to 
modify the language syntax, making the generated JData files readily usable for most 
existing JSON/binary-JSON encoders and decoders.

Specifically, we defined JSON/binary-JSON-based constructs to store N-D arrays, tables, trees,
linked lists, and graphs, and added the ability to associate metadata to any element
of the JData document. In addition, we also defined a set of core library interfaces to 
query and access the values and properties of the data units stored in a JData document.
To enhance space-efficiency and flexibility, we also introduced array data compression 
and data linking/referencing mechanisms. 

Although JData does not have all the sophisticated features of other advanced binary 
data exchange formats, such as HDF5, it is well suited for the storage of small to medium
sized datasets generated in many scientific domains or IT applications. Combined with 
the excellent availability of parsers and web-friendliness, JData is expected to be 
easily adoptable and extended in the future.

