# TLV Python Parser - Version 0.6.0

A **Tag-Length-Value** (also known as **Type-Length-Value**) is an encoding scheme used for many protocols.

The _tag_ is fixed in size (can be set from 1 to 4 bytes).
The _length_ is automatically determined by how many bytes it would take to represent the size of the _value_ by default, 
but could be set to a fixed size (from 1 to 4 bytes). The _value_ field is of variable size.

The fields are:

* __Tag__: An alphanumeric code that represents the kind of field the object represents;
* __Length__: Size of the value field (in bytes);
* __Value__: Variable-sized series of bytes which contains data for this field object.

Advantages of using TLV:

*  Sequences are usually easy to parse;
*  Unknown tags or elements can be skipped or ignored, so new versions can be added without a problem;
*  Elements can be placed in any order;
*  New elements can be created without breaking the protocol itself or the parsing function.

For more information, you can see: https://en.wikipedia.org/wiki/Type-length-value

## Installation

You can install directly from PyPI:

```
  pip install uttlv
```

Or download the source code and install using pip:
```
  pip install .
```

## How to use

To start using this package, just import the package and create an object

```python
  from uttlv import TLV

  # Create object
  t = TLV()
```

To add a tag to object, do it like a dict value:

```python
  # A tag of int value
  t[0x01] = 10
  # A tag of string value
  t[0x02] = 'test'
  # A tag of an array of bytes
  t[0x03] = bytes([1, 2, 3])
  # Or another TLV object
  another_one = TLV()
  another_one[0x05] = 234
  t[0x04] = another_one
```

A tag can only be _int_, _str_, _bytes_ or a _TLV_ itself. Any other type will raise a _TypeError_ exception.
If a tag is inserted and another object with same tag value already exists on the object, the tag will be overriden with the new value.

To get the underlying array, just call `to_byte_array()` method:

```python
  arr = t.to_byte_array()
  print('TLV:', arr)
```


## Parse

To parse an array, just call the method `parse_array()`:

```python
  # create object
  t = TLV()
  # parse from object
  data = bytes([0x03, 0x00, 0x04, 0x00, 0x00, 0x00, 0x0A])
  t.parse_array(data)
```


## Pretty print

If you call `tree()`, the object will create a string with a _tree-like_ structure to print:

```python
  from prtlv import TLV

  # Create object
  t = TLV()
  # Add value
  t[0x01] = 10
  # Print it
  print('Value:\n', t.tree())
  ## <output>
  ## Value: 
  ## 1: 10
  ##
```

## _Tag_ map

You can also add a dictionary to map a tag to its underline class type, so it's showed as correct type
instead of a bytearray.

The dictionay must have all keys as the tag values and its respective values as the class type of the 
tag:

```python
  config = {
      0x01: {TLV.Config.Type: int, TLV.Config.Name: 'NUM_POINTS'},
      0x02: {TLV.Config.Type: int, TLV.Config.Name: 'IDLE_PERIOD'},
      0x03: {TLV.Config.Type: str, TLV.Config.Name: 'NAME'},
      0x04: {TLV.Config.Type: str, TLV.Config.Name: 'CITY'},
      0x05: {TLV.Config.Type: bytes, TLV.Config.Name: 'VERSION'},
      0x06: {TLV.Config.Type: bytes, TLV.Config.Name: 'DATA'},
      0x07: {TLV.Config.Type: TLV, TLV.Config.Name: 'RELATED'},
      0x08: {TLV.Config.Type: TLV, TLV.Config.Name: 'COMMENT'},
      0x09: {TLV.Config.Type: TLV, TLV.Config.Name: 'Empty'}
  }

  # Set map
  TLV.set_tag_map(config)
```

For now, only 'int', 'str', 'bytes' and 'TLV' are accepted as valid classes. Any other class will raise
AttributeError.

If a tag map is configured, one can use the tag name to access its value:

```python
 t = TLV()
 t['NUM_POINTS'] = 10
 print(t['NUM_POINTS'])
```

And also can print it with all tag names instead of values:

```python
 t.tree(use_names=True)
 ## <output>
 ## NUM_POINTS: 10
```

You can access also the tags directly:

```python
 t = TLV()
 t['NUM_POINTS'] = 10
 print(t.NUM_POINTS)
```

By default, a field defined as type str in the tag map would be encoded or decoded as utf-8. The encoder can be replaced
to use utf16, utf32 or ascii by setting it in 

```python
 uttlv.tlv.ALLOWED_TYPES[str] = uttlv.tlv.encoder.Utf16Encoder
```
or 
```python
 uttlv.tlv.ALLOWED_TYPES[str] = uttlv.tlv.encoder.Utf32Encoder
```
or 
```python
 uttlv.tlv.ALLOWED_TYPES[str] = uttlv.tlv.encoder.AsciiEncoder
```
respectively.

## Iterator

You can iterate through the available tags inside a TLV object by using `iter()`:

```
t = TLV()
t.parse_array(bytes(command_data))

for command in t:
  pass
```