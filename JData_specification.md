 JData: a general-purpose data storage and interchange format
============================================================

- **Status of This Document**: *This document is current under development.
It does not specify a standard of any kind.*
- **Copyright Notice**: Copyright (C) Qianqian Fang (2015, 2019).
- **License**: Apache License, Version 2.0
- **Version**: RFC.pre.0
- **Abstract**:

JData is a general-purpose data interchange format. It was derived
from the JavaScript Object Notation (JSON) [RFC4627] and Universal 
Binary JSON specifications (Draft 12). JData defines a list of dedicated "name" 
fields to represent a wide range of data structures, including 
scalars, arrays, tables, structures, hashes, links, trees and graphs.


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
single data interchange format. There has been many previous efforts in 
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
HDF require skillful implementation and in-depth understanding to the 
underlying data models. For small projects with non-critical performance needs, 
using and advanced data format such as HDF may requires additional development 
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
* **human readability**: the text-based nature of JSON and it's clean, 
easy-to-read format make it intuitively readable without in-depth knowledge of 
the format itself;
* **hierarchical data support**: JSON has a tree-like data storage paradigm 
which has the native capacity to support complex hierarchical data structures; 
there is no inherent data size limit imposed by the format itself;
*  **web ready**: because JSON can be readily parsed by JavaScript, most 
JSON-encoded data file can be directly invoked (inline or load from remote 
site) from a JavaScript based web-application.

JSON also has limitations. JSON's "value" fields are weakly-typed. They only 
support strings, numbers, and Boolean types, but lack of the fine-granularity 
to represent various numerical types of different byte-lengths (in C-language, 
for example, short, int, long int, float, double, long double) and their signs 
(signed and unsigned). Because JSON is a text-based format, the size of the 
data file can be significantly larger than a binary format and requires 
additional conversion when used in an application. This introduces overhead in 
both storage and processing.

The Universal Binary JSON (UBJSON) is a binary counterpart to the JSON format. 
It specifically addresses the above mentioned limitations, yet, adheres to a 
simple grammar similar to the text-based JSON. As a trade-off, it does lose the 
"human readability" to a certain extend. Although the implemented parsers for 
UBJSON are not as abundant as JSON, due to the simplicity of the format itself, 
the development cost for implementing a parser for a new programming language 
is significantly lower than other more complex data formats.

With the ease-of-use, superior portability and parser availability, JSON and 
UBJSON have the potentials to serve as the main-stream data storage and 
interchange formats for general needs, especially for the scientific 
communities. A combination of JSON and it's binary counterpart offers features 
that are not currently available with the existing data storage schemes. Yet 
they do not provide all the advanced features found in the more sophisticated 
formats, their greatly simplified encoding and decoding strategies permit 
efficient data sharing among the general audiences.

### JData Specification Summary

JData is a specification for storing, exchanging and processing general-purpose 
data that are commonly encountered in information technology industries and 
research communities. It has a text/UNICODE format derived from the JSON 
specification and a binary format derived from the UBJSON specification. JData 
is designed to represent the commonly used data structures including arrays, 
structures, trees and graphs. A round-trip conversion is defined between the 
text and binary versions of JData documents.

The purpose of this document is to define the text and binary JData format 
specifications. This is achieved through the definition of a semantic layer 
over the JSON/UBJSON data storage syntax to map various types of complex data 
structures. Such semantic layer includes

- a list of reserved "name" fields, or keywords, that define the containers 
of various data types that are commonly used in research,
- a list of reserved "name" fields and formats to facilitate the grouping and 
organization of hierarchical data,
- a list of format properties for the associated "value" field to store the 
specific metadata of the data points, and, in addition,
- a set of conversion rules between the text and binary forms.

In the following sections, we will define the basic JData grammar, data models, 
followed by the keywords for data grouping and various data types, including 
scalars, 1,2,3 and N-dimensional arrays, sparse and complex arrays, structures, 
hashes/associative arrays, trees and graphs. The expressions of these data 
structures in both text and binary forms are specified and exemplified, and 
their conversion rules are defined.

Grammar
------------------------

### Text-based JData Storage Grammar

The text-based JData grammar is identical to the JSON grammar defined in 
[RFC4627], except that JData also accepts the Concatenated JSON Object 
Collection (CJOC), also known as the Concatenated JSON streaming format, in the 
following form:
```
    {
       "object1":{}
    }
    {
       "object2":{}
    }
    ...
```

In CJOC, multiple JSON objects can be included inside a single file or stream, 
separated by 0 or multiple permitted white spaces, namely

`LF (U+000A), CR (U+000D), tab (U+0009), or space (U+0020)`

In the case where 
[JSONReference](http://tools.ietf.org/html/draft-pbryan-zyp-json-ref-03) is 
used, the document must have only one root object, and must not contain an CJOC.

### Binary JData Storage Grammar

The binary JData grammar is identical to the UBJSON grammar defined in 
[ubjson.org (Draft 12)](http://ubjson.org), with the array grammar extension to 
support N-dimensional arrays:

`[[] [$] [type] [#] [[] [$] [an integer type] [nx ny nz ...] []] [nx*ny*nz*...*sizeof(type) ] []]`

where the integer type can be one of the UBJSON integer types (i,U,I,l,L), and 
nx, ny, and nz ... are all non-negative numbers specifying the dimensions of 
the N-dimensional array. The binary data of the N-dimensional array is then 
serialized in the **column-major** format (similar to MATLAB or FORTRAN) order.

As a special note, all UBJSON integer types must be stored in the Big-Endian 
format, according to the specification; the storage to the floating point types 
(d,D) follows the IEEE 754 specification.

Data Models
----------------------

### Topological model

Topologically, we define different parts of a JData document using the below nomenclatures

* **"leaflet"** : an element with no child
* **"compound leaf"**, or **"leaf"**: a multi-element data unit completely made of leaflets, or empty
* **"branch"**: a multi-element data unit made of both leaves and leaflets.
* **node**: a leaflet, a leaf or a branch
* **composite node**: a leaf or a branch
* **root**: the top-most node of a JSON object
* **super-root**: the top level node in a JData document containing multiple 
JSON/UBJSON objects in the form of an CJOC
* **named node**: a node in the form of `"name":{...}` or `"name":[]`, in the case 
of named and index leaves and branches, respectively, or `"name":value` in the case
of a leaflet
* **index node**: a node in the form of `{...}` or `[]`, in the case of named 
and index leaves and branches, respectively, or a single `value` in the case
of a leaflet
* **a structure**: a named or index node made of named sub-nodes, represented by `{...}`, 
a structure can be empty
* **an array**: a named or index node made of index sub-nodes, represented by `[...]`,
an array can be empty

A leaflet can only take one of the two possible forms: `value`, referred to as the
"index leaflet", and a `"name": value` pair, referred to as the "named leaflet". 
Here `value` should not contain any sub-element.

The topological elements of a JData document - "leaflet", "leaf", "branch", "root" 
and "super-root" - have specific meanings according to the definitions above.

### Semantic model

To more efficiently process the represented data, we can semantically annotate the 
underying data stucture and organize them for finer granularity. The data grouping 
annotations are

* **"data record"**: a "meaningful" datum, either in the form of a simple data point 
(leaflet) or a complex structure (a composite node); a data record can be a leaflet, 
a leaf, or a collection of leaflets or leaves. 
* **"dataset"**: a set of "logically connected" data records, such as the citation 
information of a paper
* **"data group"**: a group of datasets of "logical connections" 
* **"auxiliary data"**: nodes that store "weakly-relevant" or "irrelevant" data to 
the processing. the auxiliary data can appear at any level in the data annotation hierarchy.

The interpretations to the "meaningful datum", "logically connected data", and 
"weakly-relevant or irrelevant data" are application dependent.

The data organization annotations are ranked in the below order, from the highest 
level to the lowest level:

` super-root > root >=  data group >= dataset >= data record >= leaf > leaflet `

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
        root2{A data object that does not have meta-data
            ...
        }
    }
```
A data group, dataset or data record can be topologically presented as a branch 
(including root) or a leaf, but not a leaflet.


Data Annotation Keywords
------------------------

All JData keywords are case sensitive. Data groups, datasets and data records 
can contain metadata to include additional information regarding the data, 
such as name, create date and user-defined headers. However, metadata can 
also present in any branch or leaf.


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
string must be provided for each data group. For example

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
where the name "(name1)" is optional.

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
strings to the data group keywords (_DataGroup_,_Dataset_,_DataRecord_), separated by 
two colons, i.e. "::", for example

`"_{DataGroup,Dataset,DataRecord}_(name)::Property1=...,Property2=...,..."`

The `property` name should not contain space or comma. If comma appears inside the `value`,
it must be escaped using the form `\,`. 

An inline metadata can be attached to any named node, for example

`"name::Property1=...,Property2=...,..."`

One or more above mentioned white spaces may be inserted before or after separators "::", "," 
and "=".

The inline metadata is primarily used to store simple properties. When storage of more complex 
metadata is needed, a dedicated "metadata node" shall be inserted to the data object as the
first element of the annotated object. 

For example, if the data to be annotated is a structure, the metadata node is defined 
by a named leaf with a specific keyword "\_MetaData\_". An example can be 
found below:

```
  {
    "_MetaData_": {
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
       "_MetaData_": {
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
  ModifiedTime
  CreateTime
```

### Data Storage Keywords

JData is designed to store wide varieties of data forms. The most common data strctures
used in the scientific communities include special constants, arrays, trees, and graphs.

#### Special constants

The following constants are supported by this version of JData specification

* NaN: An NaN defined by the IEEE 754 standard shall be stored as `"_NaN_"` as a string leaflet
* +/-Inf: A +infinity or -infinity defined by the IEEE 754 standard shall be stored as a 
string leaflet `"+_Inf_"` and `"-_Inf"`, respectively
* logical true/false: A logical true/false should be represented by the JSON true/false 
logical values, or [T]/[F] markers in UBJSON
* Null: a Null (empty) value can be stored as null in JSON and [Z] in UBJSON


#### N-Dimensional Array Storage Keywords

An N-dimensional is serialized using the column-major format (i.e. the fastest index
is the left-most index, similar to arrays in MATLAB and FORTRAN; in comparison, C arrays
are row-majored).

A solid (non-sparse) N-dimensional array can be represented in two forms in JData.

##### Direct storage of N-D arrays

A solid array can be stored directly using JSON/UBJSON nested array constructs. For example, 
a 1D column vector can be stored as

`[1,2,11,9,2.1,10,...]`

while a 1D row-vector can be stored as

```
[[1],[2],[11],[9],[2.1],[10],...]
```
in the JSON flavor of JData.

Below is an example of 4x3x2 3D array

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
regarding the type of the original binary data if using the JSON (text-based) format, however, 
such information can be stored when choosing the UBJSON format. 

Please be aware that the UBJSON format supported by JData includes an extended syntax 
to facilitate storage and loading of N-D arrays in the binary format. Please refer to the 
"Grammar" section for details.

##### Annotated storage of N-D arrays

In the annotated array storage format, one shall use a structure to store an N-D array. This
allows one to use additional fields to store additional information regarding the input array.

The annotated array format is shown below for a solid 2x3 array a=[[1,2],[3,4],[5,6]]
```
   {
       "_ArrayType_": "typename",
       "_ArraySize": [N1,N2,N3,...],
       "_ArrayData": [1,2,3,4,5,6]
   }
```
Here, the array annotation keywords are defined below:
* **"_ArrayType_"**: (required) a case-insensitive string value to specify the type of the data, see below
* **"_ArraySize_"**: (required) an integer-valued (see below) 1D column vector, storing the dimensions 
of the N-D array
* **"_ArrayData_"**: (required) a 1D column vector storing the serialized array values, using the 
column-major element order

To facilitate pre-allocation of the buffer for storage of the array, it is required that 
the "_ArrayType_" and "_ArraySize_" nodes must be presented before "_ArrayData_".

The supported data types are similar to those supported by the UBJSON format, i.e.

* **uint8**: unsigned byte (8-bit)
* **int8**: signed byte (8-bit)
* **uint16**: unsigned short (16-bit)
* **int16**: signed short (16-bit)
* **uint32**: unsigned integer (32-bit)
* **int32**: signed integer (32-bit)
* **uint64**: unsigned long long integer (64-bit)
* **int64**: signed long long integer (64-bit)
* **single**: single-precision floating point (32-bit)
* **double**: double-precision floating point (64-bit)

The first 8 data types are considered "integer" types, and the last two types are considered 
"floating-point" types.

##### Complex-valued arrays

JData supports the storage of complex values or arrays using only the annotated N-D array storage
format. In this case, a named node `"_ArrayIsComplex_":true` or `"_ArrayIsComplex_":1` 
is added into the structure. In the meantime, the "_ArrayData_" becomes the concatenation of the
serialized real and imaginary parts of the complex array, with the real-part in the first half, and
the imaginary-part in the 2nd half.

For example, a complex double-precision 1x3 row vector `a=[2+6*i, 4+3.2*i,  1.2+9.7*i]` can be stored as

```
   {
       "_ArrayType_": "double",
       "_ArraySize": [1, 3],
       "_ArrayIsComplex": true,
       "_ArrayData": [2,4,1.2,6,3.2,9.7]
   }
```

The "_ArrayIsComplex" node must be presented before "_ArrayData_".

##### Sparse arrays

JData also supports the storage of N-D sparse arrays using the annotated N-D array storage
format. In this case, a named node `"_ArrayIsSparse_":true` or `"_ArrayIsSparse_":1` 
is added into the structure. In the meantime, the "_ArrayData_" becomes the concatenation 
of the serialized N-element integer indices, followed by the non-zero array elements. For a
N-D sparse array, each non-zero value requires an N-integer index to specify its location.

For example, if a 3D sparse array has the following non-zero element at the specified locations:

```
  a: i1, i2, i3, value
      2   3   1   10.1
      3   1   1   9.0
      3   3   1   8.1
      5   1   2   17
      5   2   2   9.4
      2   2   3   20.5
```
it can be saved as the following JSON format
```
   {
       "_ArrayType_": "double",
       "_ArraySize": [5, 4, 3],
       "_ArrayIsSparse_": true,
       "_ArrayData": [2,3,3,5,5,2, 3,1,3,1,2,2, 1,1,1,2,2,3, 10.1,9.0,8.1,17,9.4,20.5]
   }
```

The "_ArrayIsSparse_" node must be presented before "_ArrayData_".

##### Sparse complex-valued arrays

Using the combination of `"_ArrayIsComplex_"` and `"_ArrayIsSparse_"`, one can store
a complex valued sparse array using JData. In this case, both `"_ArrayIsComplex_":true` 
and `"_ArrayIsSparse_":true` shall be presented in the structure, with "_ArrayData_" 
ordered by serialized non-zero element indices (left-most first, and so on), followed
by the serialized real-values of the 

For example, if a 3D sparse array has the following non-zero element at the specified locations:

```
  a: i1, i2, i3, value
      2   3   1   10.1+19.0*i
      3   1   1   9.0+11*i
      3   3   2   8.1+8.2*i
```
it can be saved as the following JSON format
```
   {
       "_ArrayType_": "double",
       "_ArraySize": [4, 3, 2],
       "_ArrayIsComplex": true,
       "_ArrayIsSparse_": true,
       "_ArrayData": [2,3,3, 3,1,3, 1,1,2, 10.1,9.0,8.1, 19.0,11,8.2]
   }
```

##### Compressed array storage format

JData supports in-file data compression to enhance space-efficiency of the generated
files. Compressed data format is only supported in the annotated array storage format.

Two additional nodes are added to the annotated array structure
* **_ArrayCompressionMethod_**: a string leaflet to store the compression method, for
example, "zlib", "gzip" or "lzma"
* **_ArrayCompressionSize_**: the pre-processed array size, in the format specified by
"_ArrayType_", before the array binary stream casted to byte stream and compression

In addition, the "_ArrayData_" node is replaced by "_ArrayCompressedData_". In the 
case of JSON flavored JData files, "_ArrayCompressedData_" has a string value storing
the "Base64" encoded compressed byte-stream of the pre-processed array. In the case of
UBJSON flavored JData, "_ArrayCompressedData_" directly stores the compressed byte 
stream of the pre-processed array without "Base64" encoding.

When a compressed array format is used, "_ArrayCompressionMethod_" and 
"_ArrayCompressionSize_" must appear before "_ArrayCompressedData_".


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
     |-----node1(data1)
     |-----node2(data2)
     |       |----node2.1(data2.1)
     |       |----node2.2(data2.2)
     |-----node3(data3)
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
         {"_TreeNode_(node3)": data3},
     ]
  }
```
and the corresponding UBJSON equivallents. The notations "data0", "data1" etc are parsed
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
         {"_TreeNode_(node3)::NodeID=6,ParentID=1": data3},
     ]
  }
```
Auxiliary data can be inserted to different levels of the above JData tree document
as named nodes, as long as the name of the auxiliary node does not conflict with
the "_TreeNode_(name)" and "_TreeChildren_" of the same level. Behaviors for parsing 
the auxiliary data are application dependent.

#### Singly and doubly linked lists

Similar to the storage of trees, in JData, we use additional keywords to encapsulate a singly 
or doubly linked list inside a JSON/UBJSON array construct. The relevant keywards are

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
  }
```
If a node does not have next or prior element, the "_LinkNext_" or "_LinkPrior_" value should be 
set to null.


Recommended File Specifiers
------------------------------

For the text-based JData file, the recommended file suffix is **".jdat"**; for 
the binary JData file, the recommended file suffix is **".ubjd"**.

The MIME type for the text-based JData document is 
**"application/jdata-text"**; that for the binary JData document is 
**"application/jdata-binary"**
