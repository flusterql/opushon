# Opushon

> Representation of documentations for HTTP APIs.

## Abstract

This document extend HTTP by defining through the HTTP OPTIONS method a structure for response body that include information about the communication options, to determine the options and/or requirements associated with a resource.

## Feedback

Real world usage and feedback are welcome. Please feel free to use [GitHub Issues](https://github.com/cyril/opushon/issues) to make suggestions or fire up a Pull Request with changes to be reviewed. Thank you!

## About

This is a work in progress, largely influenced by [Zac Stewart](https://github.com/zacstewart)'s [The HTTP OPTIONS method and potential for self-describing RESTful APIs](http://zacstewart.com/2012/04/14/http-options-method.html) article.

## Status of This Document

Draft v0.2.2.

## Copyright Notice

Copyright (c) 2014 Cyril Wack.  All rights reserved.

***

## Introduction

HTTP OPTIONS [[RFC2616](https://www.ietf.org/rfc/rfc2616.txt)] header is sometimes not sufficient to convey enough information to allow the client to determine the options and/or requirements associated with a resource. While some header fields can indicate optional features implemented by the server and applicable to that resource (e.g., Allow), the client is not aware about its structure and/or constraints.

This specification defines simple JSON [[RFC7159](http://tools.ietf.org/html/rfc7159)] and YAML document formats to suit this purpose. They are designed to be reused by HTTP APIs, which can identify distinct "communication options" specific to their needs.

Thus, API clients can be informed of both the resource fields and the constraints of those fields, per method.

For example, consider a response that indicates the communication options of a user resource. If its Allow header field contains GET and POST, the body of the response should respectively inform the client about the fields that it is able to get, and the fields that it is able to post, with if needed some constraints such as the type of value.

## Requirements

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [[RFC2119](https://www.ietf.org/rfc/rfc2119.txt)].

## How to serve Opushon documents

Opushon has a media type for both the JSON and YAML variants, whos names are "application/opushon+json" and "application/opushon+yaml" respectively.

When serving Opushon over HTTP, the Content-Type of the response SHOULD contain the relevant media type name.

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
| `request`     | hash   | false        | `{}`          |
| `response`    | hash   | false        | `{}`          |

Where `request` is such as:

| Name           | Type | Nullifiable? | Default value |
| -------------- | ---- | ------------ | ------------- |
| `headers`      | hash | false        | `{}`          |
| `query_string` | hash | false        | `{}`          |
| `body`         | hash | false        | `{}`          |

And `response` is such as:

| Name      | Type | Nullifiable? | Default value |
| --------- | ---- | ------------ | ------------- |
| `headers` | hash | false        | `{}`          |
| `body`    | hash | false        | `{}`          |

***

## Parameters

The content of **headers**, **query string** and **body** params MUST be described with the keys below. When a key is missing, its default value is assigned.

| Name                | Type    | Nullifiable? | Default value |
| ------------------- | ------- | ------------ | ------------- |
| `title`             | string  | false        | `""`          |
| `description`       | string  | false        | `""`          |
| `type`              | string  | false        | `"string"`    |
| `nullifiable`       | boolean | false        | `true`        |
| `restricted_values` | array   | true         | `null`        |
| `example`           |         | true         | `null`        |

### Restricted values

If present, the value of `restricted_values` is an array where each item contains a hash with those params:

| Name          | Type   | Nullifiable? | Default value |
| ------------- | ------ | ------------ | ------------- |
| `title`       | string | false        | `""`          |
| `description` | string | false        | `""`          |
| `value`       |        | false        |               |

### Types of value

Allowed values for `type` are:

* `"string"`,
* `"number"`,
* `"boolean"`,
* `"array"`,
* `"hash"`.,
* `"file"`.

#### Constraints of value

##### String's length

| Name     | Type   | Nullifiable? | Default value |
| -------- | ------ | ------------ | ------------- |
| `minlen` | number | true         | `null`        |
| `maxlen` | number | true         | `null`        |

Constraint validation: if `minlen` and `maxlen` are present and their values are both not null, then the value of `minlen` MUST be less than the value of `maxlen`.

##### String's pattern

The pattern attribute specifies a regular expression against which the control's value.

If specified, the attribute's value MUST match the JavaScript Pattern production. [[ECMA262](http://www.ecma-international.org/ecma-262/5.1/)]

| Name      | Type   | Nullifiable? | Default value |
| --------- | ------ | ------------ | ------------- |
| `pattern` | string | true         | `null`        |

##### Number's range

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
        "request": {
            "headers": {
                "Auth-Token": {
                    "type": "string",
                    "description": "Authentication Token.",
                    "nullifiable": false,
                    "minlen": 32,
                    "example": null
                }
            },
            "query_string": {
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
            "body": {}
        },
        "response": {
            "headers": {},
            "query_string": {},
            "body": {
                "created_at": {
                    "type": "string",
                    "description": "The datetime that the resource was created at.",
                    "nullifiable": false,
                    "example": "2014-01-01T01:01:01Z"
                },
                "title": {
                    "type": "string",
                    "description": "The title of the resource.",
                    "nullifiable": false,
                    "example": "Found a bug"
                },
                "body": {
                    "type": "string",
                    "description": "The body of the resource.",
                    "nullifiable": true,
                    "example": "I'm having a problem with this."
                },
                "state": {
                    "type": "string",
                    "description": "Indicates the state of the issue.",
                    "nullifiable": false,
                    "example": "open"
                }
            }
        }
    },
    "POST": {
        "title": "Create an issue",
        "description": "Any user with pull access to a repository can create an issue.",
        "request": {
            "headers": {
                "Auth-Token": {
                    "type": "string",
                    "description": "Authentication Token.",
                    "nullifiable": false,
                    "minlen": 32,
                    "example": null
                }
            },
            "query_string": {},
            "body": {
                "title": {
                    "type": "string",
                    "description": "Issue title.",
                    "maxlen": 255,
                    "nullifiable": false,
                    "example": "Found a bug"
                },
                "body": {
                    "type": "string",
                    "description": "Issue body.",
                    "nullifiable": true,
                    "example": "I'm having a problem with this."
                },
                "labels": {
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
                    ],
                    "example": [
                        "label_1",
                        "label_2"
                    ]
                }
            }
        },
        "response": {
            "headers": {},
            "query_string": {},
            "body": {}
        }
    },
    "DELETE": {
        "title": "Delete issues",
        "description": "Remove every issues.",
        "request": {
            "headers": {
                "Auth-Token": {
                    "type": "string",
                    "description": "Authentication Token.",
                    "nullifiable": false,
                    "minlen": 32,
                    "example": null
                }
            },
            "query_string": {},
            "body": {}
        },
        "response": {
            "headers": {},
            "query_string": {},
            "body": {}
        }
    }
}
```

## References

* [RFC2119] Bradner, S., "Key words for use in RFCs to Indicate Requirement Levels", BCP 14, RFC 2119, March 1997.
* [RFC7159] Bray, T., "The JavaScript Object Notation (JSON) Data Interchange Format", RFC 7159, March 2014.
* [RFC2616] Fielding, R., Gettys, J., Mogul, J., Frystyk, H., Masinter, L., Leach, P., and T. Berners-Lee, "Hypertext Transfer Protocol -- HTTP/1.1", RFC 2616, June 1999.
* [YAML] Ben Kiki, O., Evans, C., and I. Net, "YAML Aint Markup Language", 2009, http://www.yaml.org/spec/1.2/spec.html.
