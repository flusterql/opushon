# Opushon

## Abstract

This document extend HTTP by defining through the HTTP OPTIONS method a structure for response body that include information about the communication options, to determine the options and/or requirements associated with a resource.

## Feedback

Real world usage and feedback are welcome. Please feel free to use [GitHub Issues](https://github.com/cyril/opushon/issues) to make suggestions or fire up a Pull Request with changes to be reviewed. Thank you!

## About

This is a work in progress, largely influenced by [Zac Stewart](https://github.com/zacstewart)'s [The HTTP OPTIONS method and potential for self-describing RESTful APIs](http://zacstewart.com/2012/04/14/http-options-method.html) article.

## Status of This Document

Draft.

## Copyright Notice

Copyright (c) 2014 Cyril Wack.  All rights reserved.

* * *

## Introduction

HTTP OPTIONS [[RFC2616](https://www.ietf.org/rfc/rfc2616.txt)] header is sometimes not sufficient to convey enough information to allow the client to determine the options and/or requirements associated with a resource. While some header fields can indicate optional features implemented by the server and applicable to that resource (e.g., Allow), the client is not aware about its structure and/or constraints.

This specification defines simple JSON [[RFC7159](http://tools.ietf.org/html/rfc7159)] and YAML document formats to suit this purpose. They are designed to be reused by HTTP APIs, which can identify distinct "communication options" specific to their needs.

Thus, API clients can be informed of both the resource fields and the constraints of those fields, per method.

For example, consider a response that indicates the communication options of a user resource. If its Allow header field contains GET and POST, the body of the response should respectivelly inform the client about the fields that it is able to get, and the fields that it is able to post, with if needed some constraints such as the type of value.

## Requirements

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [[RFC2119](https://www.ietf.org/rfc/rfc2119.txt)].

## The Options Structure JSON Object

The canonical model for problem details is a JSON [[RFC7159](http://tools.ietf.org/html/rfc7159)] object.

When serialised as a JSON document, that format SHOULD be identified with the "application/json" media type.

## Structure of the document

Considering a response having the following header:

```
Allow: OPTIONS,GET,HEAD,PATCH
```

The body structure MUST be a hash where each key is a HTTP method (in uppercase) and each value is a sub-hash, called an _option_ object.

Its body could look like:

```
{
  "GET": {
    "description": "Get the resource",
    "parameters": (list the gettable attributes (response body)),
    "example": (example of representation of such response body)
  },
  "PATCH": {
    "description": "Update the resource",
    "parameters": (list the patchable attributes (request body)),
    "example": (example of representation of such request body)
  }
}
```

An option object MUST have the following members:

| Name           | Type   | Nullifiable? | Default value |
| -------------- | ------ | ------------ | ------------- |
| `title`        | string | false        | `""`          |
| `description`  | hash   | false        | `""`          |
| `query_string` | hash   | false        | `{}`          |
| `parameters`   | hash   | false        | `{}`          |
| `example`      |        | true         | `null`        |

* * *

## Attributes

Both **query string** and **parameter** attributes MUST be described with the keys below. When a key is missing, its default value is assigned.

| Name           | Type    | Nullifiable? | Default value |
| -------------- | ------- | ------------ | ------------- |
| `nullifiable`  | boolean | false        | `true`        |
| `multiple`     | boolean | false        | `false`       |
| `type`         | string  | false        | `"string"`    |
| `description`  | string  | false        | `""`          |
| `examples`     | array   | false        | `[]`          |
| `strict`       | boolean | false        | `false`       |

Note 1: if the `multiple` key is true, more than one value can be included inside an array (where each item belongs to the specified type), unless the value is nullifiable and nullified.

Note 2: if the `strict` key is true, the value MUST be part of the listed examples (in `examples`).

Constraint validation: allowed values of `type` are:

* `"string"`,
* `"number"`,
* `"boolean"`,
* `"array"`,
* `"hash"`.

### String values

| Name      | Type   | Nullifiable? | Default value |
| --------- | ------ | ------------ | ------------- |
| `default` | string |              | `""`          |

#### String length

| Name     | Type   | Nullifiable? | Default value |
| -------- | ------ | ------------ | ------------- |
| `minlen` | number | true         | `null`        |
| `maxlen` | number | true         | `null`        |

Constraint validation: if minlen and maxlen are present and their values are both not null, then the value of minlen MUST be less than the value of maxlen.

#### String pattern

The pattern attribute specifies a regular expression against which the control's value, or, when the multiple attribute applies and is set, the control's values, are to be checked.

If specified, the attribute's value MUST match the JavaScript Pattern production. [[ECMA262](http://www.ecma-international.org/ecma-262/5.1/)]

| Name      | Type   | Nullifiable? | Default value |
| --------- | ------ | ------------ | ------------- |
| `pattern` | string | true         | `null`        |

### Number values

| Name      | Type   | Nullifiable? | Default value |
| --------- | ------ | ------------ | ------------- |
| `default` | number |              | `0`           |

#### Number range

| Name   | Type   | Nullifiable? | Default value |
| ------ | ------ | ------------ | ------------- |
| `min`  | number | true         | `null`        |
| `max`  | number | true         | `null`        |

### Boolean value

| Name      | Type    | Nullifiable? | Default value |
| --------- | ------- | ------------ | ------------- |
| `default` | boolean |              | `false`       |

### Array value

| Name      | Type  | Nullifiable? | Default value |
| --------- | ----- | ------------ | ------------- |
| `default` | array |              | `[]`          |

### Hash value

| Name      | Type | Nullifiable? | Default value |
| --------- | ---- | ------------ | ------------- |
| `default` | hash |              | `{}`          |

### Structure of `examples` key

Each example is an item that contains a hash with those params:

| Name          | Type   | Nullifiable? | Default value |
| ------------- | ------ | ------------ | ------------- |
| `title`       | string | true         | `null`        |
| `description` | string | true         | `null`        |
| `value`       |        | false        |               |

## Example

For example, a HTTP request like `OPTIONS /issues` could respond with:

```
HTTP/1.1 200 OK
Allow: OPTIONS,HEAD,GET,POST,DELETE
Content-Type: application/json
Content-Language: en
```

```json
{
    "GET": {
        "title": "List issues",
        "description": "List all issues across all the authenticated user's visible repositories.",
        "query_string": {
            "page": {
                "type": "number",
                "description": "Identify the page to return.",
                "min": 1,
                "max": null,
                "default": 1
            },
            "per_page": {
                "type": "number",
                "description": "Indicate the number of issues per page.",
                "min": 1,
                "max": 100,
                "default": 10
            },
            "state": {
                "description": "Indicates the state of the issues to return.",
                "examples": [
                    {
                        "value": "open",
                        "title": "Open"
                    },
                    {
                        "value": "closed",
                        "title": "Closed"
                    },
                    {
                        "value": "all",
                        "title": "All"
                    }
                ],
                "default": "open",
                "multiple": true,
                "nullifiable": true
            }
        },
        "parameters": {
            "created_at": {
                "type": "string",
                "description": "The datetime that the resource was created at.",
                "nullifiable": false
            },
            "title": {
                "type": "string",
                "description": "The title of the resource.",
                "maxlen": 255,
                "nullifiable": false
            },
            "body": {
                "type": "string",
                "description": "The body of the resource.",
                "maxlen": 10000,
                "nullifiable": true
            },
            "state": {
                "type": "string",
                "description": "Indicates the state of the issue.",
                "maxlen": 255,
                "examples": [
                    {
                        "value": "open",
                        "title": "Open"
                    },
                    {
                        "value": "closed",
                        "title": "Closed"
                    },
                    {
                        "value": "all",
                        "title": "All"
                    }
                ],
                "multiple": false,
                "nullifiable": false
            }
        },
        "example": [
            {
                "created_at": "2014-01-01T01:01:01Z",
                "title": "Found a bug",
                "body": "I'm having a problem with this.",
                "state": "open"
            }
        ]
    },
    "POST": {
        "title": "Create an issue",
        "description": "Any user with pull access to a repository can create an issue.",
        "query_string": {},
        "parameters": {
            "title": {
                "type": "string",
                "description": "Issue title.",
                "maxlen": 255,
                "nullifiable": false
            },
            "body": {
                "type": "string",
                "description": "Issue body.",
                "nullifiable": true
            },
            "labels": {
                "type": "string",
                "description": "Labels to associate with this issue.",
                "multiple": true,
                "nullifiable": true,
                "examples": [
                    {
                        "value": "label_1",
                        "title": "Java"
                    },
                    {
                        "value": "label_2",
                        "title": "Ruby"
                    },
                    {
                        "value": "label_3",
                        "title": "Elixir"
                    }
                ]
            }
        },
        "example": {
            "title": "Found a bug",
            "body": "I'm having a problem with this.",
            "labels": [
                "label_1",
                "label_2"
            ]
        }
    },
    "DELETE": {
        "title": "Delete issues",
        "description": "Remove every issues."
    }
}
```

## References

* [RFC2119] Bradner, S., "Key words for use in RFCs to Indicate Requirement Levels", BCP 14, RFC 2119, March 1997.
* [RFC7159] Bray, T., "The JavaScript Object Notation (JSON) Data Interchange Format", RFC 7159, March 2014.
* [RFC2616] Fielding, R., Gettys, J., Mogul, J., Frystyk, H., Masinter, L., Leach, P., and T. Berners-Lee, "Hypertext Transfer Protocol -- HTTP/1.1", RFC 2616, June 1999.
* [YAML] Ben Kiki, O., Evans, C., and I. Net, "YAML Aint Markup Language", 2009, http://www.yaml.org/spec/1.2/spec.html.
