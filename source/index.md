---
title: TypeJSON
subtitle: Strongly Typed JSON
comments: true
---
# What is this?

This is my thoughts on making a Strongly Typed JSON format. This is like JSON Schema in that you define what the document is suppose to look like but this is for validating the types and making less code on the each end to wire it up to real types in your system. This is not meant to replace JSON Schema.

Reasons for using TypeJSON vs JSON Schema:

- JSON Schema really came from JSON and the small amount of types supported. TypeJSON supports many different value types and allows you to build any type you want.

- JSON Schema is missing a way to name types. You can use `id` properties and `$ref` but they are not required and are normally urls. TypeJSON forces you to declare types for everything. There is no generic object type or anything type.

- JSON Schema is missing union types. It does have anyOf or oneOf but that is only for arrays and not for basic single property types. TypeJSON has a Union type. This would allow you to define a endpoint that could return many different types.

The TypeJSON parser can be just a simple parser on top of the already existing JSON parsers that will allow you to define functions to be called for each type or if your languages supports pattern matching you could use that. This will definitely depend on what language it is implemented in.

The number one goal of TypeJSON is to make it less work for you to map JSON to your types not to validate what is in that type is valid.

# TypeJSON Explained

TypeJSON is a separate file like JSON Schema. Here is a basic example.

The following JSON:
```json
{
  "name": "This is my string",
  "valid": true,
  "rating": 3.4
}
```
And this TypeJSON to describe the types.
```json
{
  "customType": {
    "name": "string",
    "valid": "bool",
    "rating": "float"
  },
  "root": "customType"
}
```
TypeJSON is a flat structor where each property is a named type. The only requirement is that there is a root type defined state the type for the root object.

## Basic Types

JSON is missing many of the types that we are use to in typed programming languages. Below is a list of the supported basic types in TypeJSON. Notice there is no object, or array type. I talk about them in their own sections below.

The basic types and what they need to be in JSON to not lose values:

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

That list isn't the limit just the basic types supported. You can define any other types if you want. You can also alias these types with a name to better handle them in your code.

Here is an example of all the basic types in a JSON and TypeJSON file.

TypeJSON:
```json
{
  "example": {
    "id": "uuid",
    "version": "int",
    "name": "string",
    "watched": "long",
    "status": "byte",
    "favoriteColor": "short",
    "valid": "bool",
    "rating": "float",
    "starPower": "double",
    "cash": "decimal:19:4",
    "started": "date",
    "meeting": "time",
    "modified": "datetime"
  },
  "root": "example"
}
```

Valid JSON:
```json
{
  "id": "962ab988-b93d-11e6-80f5-76304dec7eb7",
  "version": 20,
  "name": "This is my string",
  "watched": "9223372036854775807",
  "status": 127,
  "favoriteColor": 32767,
  "valid": true,
  "rating": 3.4,
  "starPower": "9007199254740992",
  "cash": "9999999999999.0000",
  "started": "2016-12-03",
  "meeting": "16:00:00",
  "modified": "2016-11-29T14:30:45Z"
}
```

## Nulls

Nulls are handled a little different. Nothing by default supports null but any type can support it by ending the type alias with a `?`. This make it easier to map to Maybe's or Nullable depending on the language you are using.

Here is an example allowing nulls in TypeJSON.

TypeJSON (Notice the reason property has `?` on the end of the type string):
```json
{
  "status": {
    "level": "string",
    "reason": "string?"
  },
  "root": "status"
}
```

JSON:
```json
{
  "level": "Good",
  "reason": null
}
```

Nullable or Maybe types also means it is not required to be present at all. So you could not even pass the property reason above. This in a sense makes null and undefined equal.

This would be valid JSON from the TypeJSON above:
```json
{
  "level": "Good"
}
```

## Arrays and Objects (Custom Types)

Arrays and Objects in TypeJSON need to have a type defined and everything in that array will need to be the same type, though we support union types, see below. This does mean something that could be parsed in JSON could error in TypeJSON.

Here is an example of an array of cities objects in JSON and TypeJSON:

TypeJSON (Notice the root has `[]` around to root type to indicate it is an array):
```json
{
  "city": {
    "id": "uuid",
    "name": "string"
  },
  "root": "[city]"
},
```

Valid JSON:
```json
[
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
```

## Union types

Last TypeJSON supports union types. A union type is a type that can represent multiple types. To make them you just separate the types by a `|`. This lets you support the full power of using something like JSON and still staying type safe.

TypeJSON needs to be able to infer the type so it will try and match the type in the order they are given. Say you had a type `long|string`. Since longs have to be strings in JSON to not lose values the TypeJSON parse will first try and make a long from the string. If it doesn't match it will then just leave it a string.

Here is an example using a union type in an array. It also alias `uuid` as a type id. You could then handle the `id` type in the parser.

TypeJSON:
```json
{
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
  "root": "[city|state|country]"
}
```

Valid JSON:
```json
[
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
```

## Multiple Files

Now you might want to define types in one file and reference in multiple places so you don't have to keep defining them. If you want to reference a type in another file you will set the type to `ref:path/to/file#name`.

It will work a lot like how JSON Schema uses `$ref`. A few differences is you don't have to add the `#name` on the end if you are going to use the root type.

If the original document was loaded from a http url it will use that path to load any references. If you load the file via the file system it will use that and you don't have to add the extension if the file is a .json file. Both can be full urls or paths or relative paths.

Lets take the union type example above. I could make a file for each type all in the same folder or url path:

id.json:
```json
{
  "id": "uuid",
  "root": "id"
}
```

city.json (Notice `id` references the id.json):
```json
{
  "city": {
    "id": "ref:id",
    "city": "string"
  },
  "root": "city"
}
```

state.json (Notice the `id` reference here actually use the type in the doc but isn't needed as it is used for the root):
```json
{
  "state": {
    "id": "ref:id#id",
    "state": "string"
  },
  "root": "state"
}
```

country.json:
```json
{
  "country": {
    "id": "ref:id",
    "country": "string"
  },
  "root": "country"
}
```

locations.json (This references all 3 types above as a union type):
```json
{
  "locations": "[ref:city|ref:state|ref:country]",
  "root": "locations"
}
```

The above locations file could have also just had the root set and the parser would then use the file name or last path segment as the type name when parsing.

locations.json (With just root):
```json
{
  "root": "[ref:city|ref:state|ref:country]"
}
```

## Adding TypeJSON support to Your API

TypeJSON content-type is `application/json` but if your API supports TypeJSON you will need to implement a few things to make it easier for the consumer to know the types.

First you will need to set another http header `TypeJSON` that will be set to the uri of the root type for all responses that have a body.

Here is an example of a basic get user http request and response with headers.

API Request:
```http
GET /api/v1/users/1
Host: testsite.com
Content-Type: application/json
Cache-Control: no-cache
```

API Response (Notice the TypeJSON header):
```http
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 94
TypeJSON: /api/v1/types/user

{
  "id": "1",
  "email": "test@testsite.com",
  "firstName": "Test",
  "lastName": "User"
}
```

TypeJSON Request:
```http
GET /api/v1/types/user
Host: testsite.com
Content-Type: application/json
Cache-Control: no-cache
```

TypeJSON Response:
```http
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 114

{
  "root": {
    "id": "int",
    "email": "string",
    "firstName": "string?",
    "lastName": "string?"
  }
}
```

The new TypeJSON header is useful but it is more of an after the fact thing. To support TypeJSON in full you need to also support the HTTP Option request. This will return all the allowed request types for the end-point. It will also have a body that is JSON and has a type set for each request type that has a type in the request or response body. If it is not set then that request or response doesn't have a body.

Lets take the example above of `/api/v1/user/{id}` end point.

HTTP Options Request:
```http
OPTIONS /api/v1/users/1
Host: testsite.com
Content-Type: application/json
Cache-Control: no-cache
```

HTTP Options Response:
```http
HTTP/1.1 200 OK
Allow: GET,HEAD,PUT,DELETE
Content-Type: application/json
Content-Length: 117

{
  "GET": {
    "response": "ref:/api/v1/types/user"
  },
  "PUT": {
    "request": "ref:/api/v1/types/user"
  }
}
```

This tells you that the end-point supports GET, HEAD, PUT, and DELETE. If you do a `get` request the response will be a `user`. If you are making `put` request you need to have a `user` in the request. All other requests and responses have no body to be parsed.

## Libraries

Currently none as I want to run this by a bunch of programmers first to see if I am crazy to build it this way. I plan on writing a TypeScript, Elm, and PureScript implementation of this. If you want to help out and build a library in your favorite typed language go ahead and let me know so I can update this page. Keep in mind this is still a very new spec and we might have to make changes for edges cases I am not thinking about.

## Conclusion

What do you think? Please leave feedback in the comments below and see if we can't make a better strongly typed JSON format.
