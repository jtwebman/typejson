---
title: TypeJSON
subtitle: Strongly Typed JSON
comments: true
---
# What is this?

This is my thoughts on making a Strongly Typed JSON format. TypeJSON is like JSON Schema in that you define what the document is suppose to look like but in types. It is a way to make it less code on each end to wire it up to a typed language in your system. This is not meant to replace JSON Schema or to validate the content of the type but you could do that if you wanted with parameters talked about below.

Reasons for using TypeJSON vs JSON Schema:

- JSON Schema really came from JSON and the small amount of types supported. TypeJSON supports many different value types and allows you to build any type you want. This lets you map JSON directly to your types in your system.

- JSON Schema is missing a way to name types. You can use `id` properties and `$ref` but they are not required and are normally urls.

- TypeJSON forces you to declare types for everything. There is no generic object type or anything type.

- JSON Schema is missing union types. It does have anyOf or oneOf but that is only for arrays and not for basic single property types. TypeJSON has a Union type. This would allow you to define an endpoint that could return many different types.

TypeJSON is valid JSON so the parser can be just a simple parser on top of an already existing JSON parsers that will allow you to define functions to be called for each type or if your languages supports pattern matching you could use that. This will definitely depend on what language it is implemented in.

The number one goal of TypeJSON is to make it less work for you to map JSON to your types not to validate what is in the type is valid. Since validation almost always includes some business logic for the system I will leave that up to you to implement.

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
    "rating": "double"
  },
  "root": "customType"
}
```
TypeJSON is a flat structure where each property is a named type. The only requirement is that there is a root type defined. Names can't have `[`, `]`, `?`, or `|` in them. A name can have a `:` in them but it is used to set the type for the parameter and is not used for the name. This is talked about more below.

## Basic Types

JSON is missing many of the types that we enjoy in typed programming languages. Below is a list of the supported basic types in TypeJSON. Notice there is no object, or array type. I talk about them in their own sections below.

The basic types and what they need to be in JSON to not lose values:

- **int** (32-bit Signed Integer, can be a number or a string)
- **long** (64-bit Signed Integer, needs to be a string)
- **float** (32-bit Floating Point, can be a number or a string)
- **double** (64-bit Floating Point, can be a number or a string)
- **decimal** (Need to include precision and scale separated by `:`, needs to be a string)
- **byte** (8-bit Signed Integer, can be a number or a string)
- **short** (16-bit Signed Integer, can be a number or a string)
- **date** (YYYY-MM-DD format, needs to be a string)
- **time** (HH:MM:SS format with a 24 hour clock for hour, needs to be a string)
- **datetime** (ISO 8601 adjusted to UTC, no timezone, needs to be a string)
- **uuid** (UUID in canonical form, needs to be a string)
- **bool** (Normal JSON boolean)
- **string** (UTF-8 String)
- **base64** (This is a base64 string that will be converted to a byte array, allowing you to support binary data, though I might make another binary end-point with no json for that)
- **ref** (This is explained farther down for referencing other files)

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
    "modified": "datetime",
    "image": "base64"
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
  "modified": "2016-11-29T14:30:45Z",
  "image": "R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7"
}
```

The nice part about TypeJSON is this list isn't the limit. It is just the basic types supported. You can define any other type if you want by using a type like `"datetimetz": "string"` that could be a ISO 8601 date string with timezones supported.

Another example would be a type like `"id": "string?"`. It would be an id type and could use your own code to set what that is suppose to mean. The `?` is talked about down below as being a nullable type. This would allow you to have a Maybe ID so the create request could leave off the id but the response would return it with the id.

If you define your own type you can also include parameters after the name, like the decimal type, using `:` to separate each one. So if you wanted to make your own limited string length type you could define it this way `"lstring:int": "string"` and then when using it on a property like `"email": "lstring:255"` the parser would give you the parameter 255 as an `int` and you could make sure the string wasn't longer.

## Nulls

Nulls are handled a little different. Nothing by default supports null but any type can support it by ending the type name with a `?`. This makes it easier to map to Maybe's or Nullable depending on the language you are using.

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

Nullable or Maybe types also means it is not required to be present at all. So you could not even pass the property reason above. This in a sense makes `null` and `undefined` equal.

This would be valid JSON from the TypeJSON above:
```json
{
  "level": "Good"
}
```

If your type is a custom type or the decimal type and takes parameters make sure the `?` is after the parameters. To use the example above with a limited length string that could be null would look like this `"email": "lstring:255?"`.

## Arrays and Objects (Custom Types)

Arrays and Objects in TypeJSON need to have a type defined and everything in that array will need to be the same type, though we support union types, see below. When defining arrays you surround the type in `[]`. For objects you use the normal json `{}` structure.

Here is an example of an array of cities objects in TypeJSON with valid JSON:

TypeJSON (Notice the cities has `[]` around the city type to indicate it is an array):
```json
{
  "city": {
    "id": "uuid",
    "name": "string"
  },
  "cities": "[city]",
  "root": "cities"
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

Here is an example of an object that can have objects and arrays of objects in it. Notice how they are all defined in a flat structure and don't have to be in any order.

TypeJSON:
```json
{
  "user": {
    "id": "uuid?",
    "email": "string",
    "groups": "[group]?",
    "login": "credential?"
  },
  "group": {
    "id": "uuid?",
    "name": "string",
  },
  "credential": {
    "id": "uuid?",
    "username": "string",
    "password": "password?"
  },
  "password": "string",
  "root": "user"
}
```

Valid JSON (Though you should never pass back a hashed password in real life):
```json
{
  "id": "962ab988-b93d-11e6-80f5-76304dec7eb7",
  "email": "test@testsite.com",
  "groups": [
    {
      "id": "938a8db0-156e-4804-8f2a-aa3502ecd585",
      "name": "Admins"
    }
  ],
  "credential": {
    "id": "d623b9e1-6dfb-4051-a4d3-c566556d971e",
    "username": "testuser",
    "password": "2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824"
  }
}
```

Valid JSON (Another example from the same TypeJSON):
```json
{
  "email": "test@testsite.com"
}
```
All other types were nullable or maybes including the groups array. So you can define arrays that can be null or not set as well. You just have to add `?` on the outside of the `[]` brackets.

Arrays can also support nulls inside them as then each type that is returned will be a Maybe or Nullable. So if the type was `[group?]` then you could have an array with null in it like `[null, { "name": "Group Name" }]`. Just know all types coming from this array would be maybe's forcing you to handle that. So I might avoid doing that.

## Union types

Last TypeJSON supports union types. A union type is a type that can represent multiple types. To make them you just separate the types by a `|`. This lets you support the full power of using something like JSON and still staying type safe.

TypeJSON needs to be able to infer the type, so it will try and match the type in the order they are given. Say you had a type `long|string`. Since longs have to be strings in JSON to not lose values the TypeJSON parse will first try and make a long from the string. If it doesn't match it will then just leave it a string.

Here is an example using a union type in an array. It also defines `uuid` as type `id`, so you could then handle the `id` type in the parser.

TypeJSON:
```json
{
  "id": "uuid",
  "city": {
    "id": "id?",
    "city": "string"
  },
  "state": {
    "id": "id?",
    "state": "string"
  },
  "country": {
    "id": "id?",
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

Union types also support allowing null or maybe type. All you have to do is add a `?` to the type you want to be passed as a maybe or nullable. So if the union type can be a long, or a maybe string, then the type would be `long|string?`.

You need to be careful here when dealing with null values in the types in the union. Take this example:
```json
{
  "id": "int?",
  "globalId": "uuid?",
  "allId": "id|uuid"
}
```
The `id` and the `globalId` type can both be null but if nothing or null is passed for the type `allId` you will always get a Maybe or Nullable `id` type as they are matched in the order they are written in.

## Ref (Multiple Files)

Now you might want to define types in more then one file as well as reference types in multiple places so you don't have to keep defining them. If you want to reference a type in another file you will set the type to `ref:path/to/file:name` or `ref:path/to/file` for the root type.

The `ref` type is defined like this `ref:string|ref:string:string`. Basically it is a union type with a either one or two parameters as a string.

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
    "id": "ref:id:id",
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

All the files including the locations file could have also just had the root set and the parser would then use the file name or last path segment as the type name when parsing.

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
    "id": "int?",
    "email": "string",
    "firstName": "string?",
    "lastName": "string?"
  }
}
```

The client should also pass the TypeJSON header for a request with bodies like a PUT or POST but that isn't a hard requirement if the front-end client doesn't want to handle types the same way.

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

## TypeJSON in TypeJSON

If you were to define TypeJSON in TypeJSON this is what all the types would look like using just the basic `double`, `string`, and `bool` types supported in raw JSON:
```json
{
  "int": "double|string",
  "long": "string",
  "float": "double|string",
  "double": "double|string",
  "decimal:byte:byte": "string",
  "byte": "double|string",
  "short": "double|string",
  "date": "string",
  "time": "string",
  "datetime": "string",
  "uuid": "string",
  "base64": "string",
  "string": "string",
  "bool": "bool",
  "refRoot:string": "string",
  "refNamed:string:string": "string",
  "ref": "refRoot|refNamed"
}
```

## Libraries

Currently none as I want to run this by a bunch of programmers first to see if I am crazy to build it this way. I plan on writing a TypeScript, Elm, and PureScript implementation of this. If you want to help out and build a library in your favorite typed language go ahead and let me know so I can update this page. Keep in mind this is still a very new spec and we might have to make changes for edges cases I am not thinking about.

## Conclusion

What do you think? Please leave feedback in the comments below and see if we can't make a better strongly typed JSON format.
