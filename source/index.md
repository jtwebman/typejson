---
title: TypeJSON
subtitle: Strongly Typed JSON
---
# What is this?

This is my thoughts on making a Strongly Typed JSON format. I plan on writing
a TypeScript, Elm, and PureScript implementation of this. If you want to help
out and build a library in your favorite typed language go ahead and let me
know so I can update this page.

# TypeJSON Explained

All values in TypeJSON have a type. They are either implicit basic types already
support by JSON like string, boolean, and number (which just alias a 32-bit
floating point number not a 64-bit float). Notice null, object, and array are
not types. We will talk about them later.

The following JSON:
```json
{
  "name": "This is my string",
  "valid": true,
  "rating": 3.4
}
```
And this TypeJSON inline for the same thing.
```json
{
  "name:string": "This is my string",
  "valid:bool": true,
  "rating:float": 3.4
}
```

There are a few more basic types to map better with other typed languages. This
includes int (32-bit Signed Integer), long (64-bit Signed Integer), float
(32-bit Floating Point), decimal, byte (8-bit Signed Integer ), short
(16-bit Signed Integer), date, time, datetime (ISO 8601 adjusted to UTC), and
uuid. That list isn't the limit just the bare basic. You can define any other
types if you want. You can also alias types with a name.

Here is an example of all the other basic types in TypeJSON inline format:
```json
{
  "id:uuid": "962ab988-b93d-11e6-80f5-76304dec7eb7",
  "version:int": 20,
  "name:string": "This is my string",
  "watched:long": "9223372036854775807",
  "status:byte": 127,
  "permissions:short": 32767,
  "valid:bool": true,
  "rating:float": 3.4,
  "starpower:double": "1.1",
  "cash:decimal:19:4": "9999999999999.0000",
  "started:date": "2016-12-03",
  "meeting:time": "16:00:00",
  "modified:datetime": "2016-11-29T14:30:45Z"
}
```

Nulls are handle a little different. Nothing by default supports null but any
type can support it by ending the type alias with a '?'.

Here is an example allowing nulls:
```json
{
  "canthavenulls:string": "This property can not be null",
  "nullable:string?": null
}
```

Arrays and objects in TypeJSON need to have a type defined and everything in
that array will need to be the same type. This does mean something that could be
parsed in JSON could error in TypeJSON.
