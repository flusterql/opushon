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

***

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
Allow: OPTIONS,HEAD,GET,PATCH
```

The body structure MUST be a hash where each key is a HTTP method (in uppercase) and each value is a sub-hash, called an _option_ object.

Its body could look like:

```
{
  "GET":   ( options and/or requirements associated with the resource ),
  "PATCH": ( options and/or requirements associated with the resource )
}
```

An option object MUST have the following members:

| Name          | Type   | Nullifiable? | Default value |
| ------------- | ------ | ------------ | ------------- |
| `title`       | string | false        | `""`          |
| `description` | string | false        | `""`          |
| `parameters`  | hash   | false        | `{}`          |
| `examples`    | hash   | false        | `{}`          |

Where `parameters` is such as:

| Name     | Type | Nullifiable? | Default value |
| -------- | ---- | ------------ | ------------- |
| `input`  | hash | true         | `null`        |
| `output` | hash | true         | `null`        |

And where `examples` is such as:

| Name     | Type | Nullifiable? | Default value |
| -------- | ---- | ------------ | ------------- |
| `input`  |      | true         | `null`        |
| `output` |      | true         | `null`        |

***

## Parameters

Both **query string** and **body** params MUST be described with the keys below. When a key is missing, its default value is assigned.

| Name                | Type    | Nullifiable? | Default value |
| ------------------- | ------- | ------------ | ------------- |
| `title`             | string  | false        | `""`          |
| `description`       | string  | false        | `""`          |
| `type`              | string  | false        | `"string"`    |
| `nullifiable`       | boolean | false        | `true`        |

Constraint validation: allowed values of `type` are:

* `"string"`,
* `"number"`,
* `"boolean"`,
* `"array"`,
* `"hash"`.

### Input Constraints

| Name                | Type    | Nullifiable? | Default value |
| ------------------- | ------- | ------------ | ------------- |
| `query_string`      | boolean | false        | `true`        |
| `restricted_values` | array   | true         | `null`        |

#### Structure of `restricted_values` key

If present, the value of `restricted_values` is an array where each item contains a hash with those params:

| Name          | Type   | Nullifiable? | Default value |
| ------------- | ------ | ------------ | ------------- |
| `title`       | string | false        | `""`          |
| `description` | string | false        | `""`          |
| `value`       |        | false        |               |

#### String length

| Name     | Type   | Nullifiable? | Default value |
| -------- | ------ | ------------ | ------------- |
| `minlen` | number | true         | `null`        |
| `maxlen` | number | true         | `null`        |

Constraint validation: if minlen and maxlen are present and their values are both not null, then the value of minlen MUST be less than the value of maxlen.

#### String pattern

The pattern attribute specifies a regular expression against which the control's value.

If specified, the attribute's value MUST match the JavaScript Pattern production. [[ECMA262](http://www.ecma-international.org/ecma-262/5.1/)]

| Name      | Type   | Nullifiable? | Default value |
| --------- | ------ | ------------ | ------------- |
| `pattern` | string | true         | `null`        |

#### Number range

| Name   | Type   | Nullifiable? | Default value |
| ------ | ------ | ------------ | ------------- |
| `min`  | number | true         | `null`        |
| `max`  | number | true         | `null`        |

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
        "parameters": {
            "input": {
                "page": {
                    "type": "number",
                    "description": "Identify the page to return.",
                    "min": 1,
                    "max": null
                },
                "per_page": {
                    "type": "number",
                    "description": "Indicate the number of issues per page.",
                    "min": 1,
                    "max": 100
                },
                "state": {
                    "description": "Indicates the state of the issues to return.",
                    "restricted_values": [
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
                    "nullifiable": true
                }
            },
            "output": {
                "created_at": {
                    "type": "string",
                    "description": "The datetime that the resource was created at.",
                    "nullifiable": false
                },
                "title": {
                    "type": "string",
                    "description": "The title of the resource.",
                    "nullifiable": false
                },
                "body": {
                    "type": "string",
                    "description": "The body of the resource.",
                    "nullifiable": true
                },
                "state": {
                    "type": "string",
                    "description": "Indicates the state of the issue.",
                    "nullifiable": false
                }
            }
        },
        "examples": {
            "input": null,
            "output": [
                {
                    "created_at": "2014-01-01T01:01:01Z",
                    "title": "Found a bug",
                    "body": "I'm having a problem with this.",
                    "state": "open"
                }
            ]
        }
    },
    "POST": {
        "title": "Create an issue",
        "description": "Any user with pull access to a repository can create an issue.",
        "parameters": {
            "input": {
                "title": {
                    "query_string": false,
                    "type": "string",
                    "description": "Issue title.",
                    "maxlen": 255,
                    "nullifiable": false
                },
                "body": {
                    "query_string": false,
                    "type": "string",
                    "description": "Issue body.",
                    "nullifiable": true
                },
                "labels": {
                    "query_string": false,
                    "type": "string",
                    "description": "Labels to associate with this issue.",
                    "nullifiable": true,
                    "restricted_values": [
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
            "output": null
        },
        "examples": {
            "input": {
                "title": "Found a bug",
                "body": "I'm having a problem with this.",
                "labels": [
                    "label_1",
                    "label_2"
                ]
            },
            "output": null
        }
    },
    "DELETE": {
        "title": "Delete issues",
        "description": "Remove every issues.",
        "parameters": {
            "input": null,
            "output": null
        },
        "examples": {
            "input": null,
            "output": null
        }
    }
}
```

## Limitations

In input parameters, a query string parameter and a body parameter cannot have the same name because of the structure of data (which is a hash).

## References

* [RFC2119] Bradner, S., "Key words for use in RFCs to Indicate Requirement Levels", BCP 14, RFC 2119, March 1997.
* [RFC7159] Bray, T., "The JavaScript Object Notation (JSON) Data Interchange Format", RFC 7159, March 2014.
* [RFC2616] Fielding, R., Gettys, J., Mogul, J., Frystyk, H., Masinter, L., Leach, P., and T. Berners-Lee, "Hypertext Transfer Protocol -- HTTP/1.1", RFC 2616, June 1999.
* [YAML] Ben Kiki, O., Evans, C., and I. Net, "YAML Aint Markup Language", 2009, http://www.yaml.org/spec/1.2/spec.html.
