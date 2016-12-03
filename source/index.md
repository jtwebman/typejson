---
title: TypeJSON
subtitle: Strongly Typed JSON
comments: true
---
# TypeJSON - What is this?

This is my thoughts on making a Strongly Typed JSON format. I plan on writing a TypeScript, Elm, and PureScript implementation of this. If you want to help out and build a library in your favorite typed language go ahead and let me know so I can update this page.

All values in TypeJSON have a type. The advantage is it is all built in standard JSON format. No special property names.

The following JSON:
```json
{
  "name": "This is my string",
  "valid": true,
  "rating": 3.4
}
```
And this TypeJSON for the same thing.
```json
{
  "type": {
    "customType": {
      "name": "string",
      "valid": "bool",
      "rating": "float"
    }
  },
  "customType": {
    "name": "This is my string",
    "valid": true,
    "rating": 3.4
  }
}
```
TypeJSON you define all the types ahead of time in a JSON object. Everything is still in a normal JSON format, though many of the new supported basic types have to be strings to not lose values.

## Basic Types

The basic types to map better with other typed languages are as follows:

- **int** (32-bit Signed Integer, can be a number or a string)
- **long** (64-bit Signed Integer, needs to be a string)
- **float** (32-bit Floating Point, can be a number or a string)
- **decimal** (Need to include precision and scale separated by `:`, needs to be a string)
- **byte** (8-bit Signed Integer, can be a number or a string)
- **short** (16-bit Signed Integer, can be a number or a string)
- **date** (YYYY-MM-DD format, needs to be a string)
- **time** (HH:MM:SS format with a 24 hour clock for hour, needs to be a string)
- **datetime** (ISO 8601 adjusted to UTC, no timezone, needs to be a string)
- **uuid** (UUID in canonical form, needs to be a string)
- **bool** (Normal JSON boolean)
- **string** (UTF-8 String)

That list isn't the limit just the basic types supported. You can define any other types if you want. You can also alias types with a name.

Here is an example of all the other basic types in TypeJSON format:
```json
{
  "type": {
    "example": {
      "id": "uuid",
      "version": "int",
      "name": "string",
      "watched": "long",
      "status": "byte",
      "favoritecolor": "short",
      "valid": "bool",
      "rating": "float",
      "starpower": "double",
      "cash": "decimal:19:4",
      "started": "date",
      "meeting": "time",
      "modified": "datetime"
    }
  },
  "example": {
    "id": "962ab988-b93d-11e6-80f5-76304dec7eb7",
    "version": 20,
    "name": "This is my string",
    "watched": "9223372036854775807",
    "status": 127,
    "permissions": 32767,
    "valid": true,
    "rating": 3.4,
    "starpower": "9007199254740992",
    "cash": "9999999999999.0000",
    "started": "2016-12-03",
    "meeting": "16:00:00",
    "modified": "2016-11-29T14:30:45Z"
  }
}
```

## Nulls

Nulls are handled a little different. Nothing by default supports null but any type can support it by ending the type alias with a `?`. This make it easier to map to Maybe's or Nullable depending on the language you are using.

Here is an example allowing nulls in TypeJSON:
```json
{
  "type": {
    "status": {
      "level": "string",
      "reason": "string?"
    }
  },
  "status": {
    "level": "Good",
    "reason": null
  }
}
```

## Arrays and Objects (Custom Types)

Arrays and Objects in TypeJSON need to have a type defined and everything in that array will need to be the same type, though we support union types, see below. This does mean something that could be parsed in JSON could error in TypeJSON.

Here is an example of an array of cities objects in TypeJSON:
```json
{
  "type": {
    "city": {
      "id": "uuid",
      "name": "string"
    },
    "cities": "[city]"
  },
  "cities": [
    {
      "id": "cce56e92-b946-11e6-80f5-76304dec7eb7",
      "name": "Portland"
    },
    {
      "id": "e3bf98d6-b946-11e6-80f5-76304dec7eb7",
      "name": "New York"
    },
    {
      "id": "edaf5534-b946-11e6-80f5-76304dec7eb7",
      "name": "San Francisco"
    }
  ]
}
```
Notice you have to define the top level type `cities` in this example. Also notice to make it an array type it was surrounded buy `[]`.

## Union types

Last TypeJSON supports union types. To make them you just separate the types by a `|`. This lets you support the full power of using something like JSON and still staying type safe. A union type is a type that can represent multiple types but since the parser needs to be able to infer them they can only be object types and each type needs to have different property names. We don't support basic types in union types but you could build a simple object type to get around this.

Here is an example using a union type:
```json
{
  "type": {
    "id": "uuid",
    "city": {
      "id": "id",
      "city": "string"
    },
    "state": {
      "id": "id",
      "state": "string"
    },
    "country": {
      "id": "id",
      "country": "string"
    },
    "locations": "[city|state|country]"
  },
  "locations": [
    {
      "id": "cce56e92-b946-11e6-80f5-76304dec7eb7",
      "city": "Portland"
    },
    {
      "id": "e3bf98d6-b946-11e6-80f5-76304dec7eb7",
      "state": "Oregon"
    },
    {
      "id": "edaf5534-b946-11e6-80f5-76304dec7eb7",
      "country": "USA"
    }
  ]
}
```

## Type Section

All types need to be setup in this section. They don't have to be in any order and they can be used in other types in the same section.

Here is an example with some custom types. We alias id as a `uuid` and use it in the `user` type:
```json
{
  "type": {
    "id": "uuid",
    "tag": "string",
    "user": {
      "id": "id",
      "firstName": "string?",
      "lastName": "string?",
      "email": "string",
      "tags": "[tag]"
    }
  },
  "user": {
    "id": "962ab988-b93d-11e6-80f5-76304dec7eb6",
    "firstName": null,
    "lastName": "Turner",
    "email": "jt@typejson.org",
    "tags": ["nerd", "starwars", "programmer"]
  }
}
```

## Top Level Object

TypeJSON top level object can only have two properties. The first is `type` with at least one type defined in it. The other property needs to be a name defined in the type section. So if the top level object is an array you have to define that type as well and use the name. See example above with `Arrays and Objects`.

## Libraries

Currently none as I want to run this by a bunch of programmers first to see if I am crazy to build it this way. What do you think? Please leave feedback in the comments below and see if we can't make a better strongly typed JSON format.
