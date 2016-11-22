---
title: API Reference

language_tabs:
  - shell

search: true
---

# Getting Started

The DeskPRO API is a REST-based API that runs over HTTP(S). All API requests are made to a URL that begins with
`http://example.com/api/v2/`.

Since all API requests are simply HTTP requests, it's easy to use in any programming
language. For the purposes of demonstration, we'll use cURL in our examples here.

Here's an exmaple call made to the `helpdesk/discover` endpoint:

```shell
curl http://example.com/api/v2/helpdesk/discover
```

> Example response:

```json
{
    "data": {
        "is_deskpro": true,
        "helpdesk_url": "https:\/\/example.com\/",
        "base_api_url": "https:\/\/example.com\/api\/v2\/",
        "build": "1477995658"
    },
    "meta": {},
    "linked": {}
}
```

## Authentication

The above example is one of the few API endpoints that is public. Almost every API endpoint in DeskPRO that
is actually useful requires authentication. The most common way to use the API is via an API key. An admin can
create keys from Admin > Apps > API Keys.

To authenticate an API request, you send an Authorization header like `Authorization: key YOURKEY`. Your API key
will be an integer, followed by a colon, followed by a long random string of characters.

> Example request to the /me endpoint to see info about the agent the key is assigned to:

```shell
curl -H "Authorization: key 2:YWK2PGCS8CNW62SR8TBTBKWMY" http://example.com/api/v2/me
```

> Example response:

```json
{
    "data": {
        "auth_method": "api_key",
        "app_id": null,
        "person_id": 1,
        "person": {
            "id": 1,
            "...": "..."
        }
    },
    "meta": {},
    "linked": {}
}
```

## Request Format

The DeskPRO API is completely JSON.

* You SHOULD send a `Accept: application/json` to confirm that your app is expecting JSON back from the API.
* You MUST send `POST/PUT` requests with a `Content-Type: application/json` header and send a well-formed JSON payload.

### HTTP Verbs

The HTTP verb you use is meaningful. Here's a summary of the HTTP verbs you can use with the DeskPRO API and what they mean:

| Verb   | Description                         | Example                         |
|--------|-------------------------------------|---------------------------------|
| GET    | Get a single resource or collection | GET /tickets or GET /tickets/13 |
| DELETE | Delete a single resource            | DELETE /ticket/13               |
| PUT    | Create something new                | PUT /tickets                    |
| POST   | Update an existing resource         | POST /tickets/13                |

## Response Format

When you perform a `GET` request to load a resource or a collection, you'll get a response like this. 

```json
{
    "data": {},
    "links": {},
    "meta": {}
}
```

* `data` is the JSON representation of the resource requested, or an array of objects if you requested a collection
* `links` optional; when specified, it is an array of useful links. Resource collections always include links for “next”, “prev”, “first”, “last”.
* `meta` optional; misc information that varies based on endpoint. Resource collections should include “count”, “total_count*, “page”, “total_pages”.

### Collection Responses

During a resource collection request, you can specify the following optional query parameters:

* `count` the number of results to return
* `page `the page you are requesting

Collection responses are always an _array_  of objects inside of `data`.

`links` will contain some useful links to help you paginate:

* `next` URL to the next page of results
* `prev` URL to the previous page of results
* `self` URL to the page you requested
* `first` URL to the first page of results
* `last` URL to the last page of results

`meta` will contain some useful data:

* `total` is the total count (all matching resources)
* `page` is current page number
* `per_page` is a count of the number of resources returned to you on this page
* `total_pages` is the total number of pages for this request

> Example request to /tickets which is a collection resource:

```shell
curl -H "Authorization: key 2:YWK2PGCS8CNW62SR8TBTBKWMY" http://example.com/api/v2/tickets
```

> Example response of a collection resource

```json
{
   "links": {
        "first": "/tickets",
        "next": "/tickets?page=2",
        "prev": null,
        "last": "/tickets?page=106"
   },
   "data": {
      {
        "id": 1,
        "person_id": 4,
        "title": "Foo Bar"
      },
      {
        "id": 2,
        "person_id": 12,
        "title": "Fizz Buzz"
      }
   },
   "meta": {
      "total": 5445,
      "page": 1,
      "per_page": 10,
      "total_pages": 545
   }
}
```

### Creating and Updating Content

When you update a resource with `POST`, the API will simply return a `204 No Content` empty response to ackknowledge that
the request succeeded. When you create a resource with a `PUT` request, the API will return a `201 Created` response.

In both cases, a `Location` header will be supplied with the full URL to the API endpoint where
you can `GET` the resource.

TIP: If you would rather your `POST/PUT` request return the full resource automatically (i.e. you want to use it right away),
you can submit your request with the `?follow_location=1` query string parameter. This will cause the API
to return the resource as if you did the GET yourself.

### Error Responses

Error responses are also JSON and follow the shape described here.

```
{
    "status": 400
    "code": "invalid_json_body",
    "message": "Invalid JSON body"
}
```

* The `status` will be the HTTP status code that most closely matches the error.
* The `code` is a DeskPRO specific string to represent the specific error that happened. You may
wish to use this value in your code in an if/switch etc to base your error handling on.
* The `message` is a human-readable English string that describes what went wrong. This value is typcially
only useful for developers.

#### Details Errors

In some cases, detailed error messages are available, such as when there are form validation errors.

```json
{
    "status": 400,
    "code": "invalid_input",
    "message": "Request input is invalid.",
    "errors": {
        "errors": [
            {
                "code": "extra_fields",
                "message": "Unexpected field names: name"
            }
        ],
        "fields": {
            "email": {
                "errors": [
                    {
                        "code": "required",
                        "message": "This value should not be blank."
                    }
                ]
            },
            "date": {
                "errors": [
                  {
                    "code": "invalid_date",
                    "message": "This date is invalid"
                  }
                ],
                "fields": {
                  "day": {
                      "errors": [
                        {
                          "code": "out_of_range",
                          "message": "Must be a number between 1 and 31"
                        }
                      ]
                  },
                  "year": {
                    "errors": [
                      {
                        "code": "required",
                        "message": "This value is required"
                      }
                    ]
                  }
                }
            }
        }
    }
}
```

The root `errors` property is an object that has two properties:

* `errors.errors` is a JSON array of error objects that happened on a global level (not specific to any one input).
* `errors.fields` is a JSON object that has a property for every expected input name that had an error.
  Inside of each field is a property named “errors” which is an array of error objects (code/message).

### Common HTTP Status Codes

The HTTP status codes that the API returns is significant. Here is a list of common return codes.

| Code   | Verbs | Example                         |
|--------|-------------------------------------|---------------------------------|
|200 |GET |found and returned|response is described above                                                          |
|404 |Any |resource not found|                                                                                     |
|201 |POST|created resource  |the created entity is in the response (including “self” and “data” links)            |
|204 |PUT |updated resource  |no body will be in the response                                                      |
|400 |Any |Bad Request       |malformed request or invalid input                                                   |
|401 |Any |unauthorized      |you are not authenticated                                                            |
|403 |Any |forbidden         |we accept your token, but you can’t perform this action, no resource has been updated|
|405 |Any |method not allowed|                                                                                     |
|429 |Any |too many requests |throttle/rate-limit hit                                                              |
|423 | Any | Locked | Special code we use when the helpdesk is offline for maintenance |

## API Key Limits

You can specify API limits on every API key you create. For example, it is not uncommon to set an upper limit on
an API key to prevent some kind of mistake in it's use (such as a flood or infinite loop situation). You can define
these limits in terms of an hourly limit or a daily limit.

For cloud customers, there is also default global limit to prevent abuse. You can raise this on request by contacting
 us at support@deskpro.com.
 
When you hit a rate limit, requests will begin to fail with a HTTP status code `429 Too Many Requests` error.

# API Token Exchange

You can also use the API to exchange an agent login for an API token which can be used to authenticate requests.
An API token is similar to an API key except that it doesn't need to be created by an admin. It's created
by the system when given a successful set of credentials.

A token is used in the same way except the Authorzation header looks like `Authorization: token YOURTOKEN`. This is
the same as the API key method except it starts with the word "token" instead of "key".

> Example request sending login credentials

```shell
curl -H "Content-Type: application/json" \
     -X POST \
     -d '{"email": "user@example.com", "password": "my_password"}'
     http://example.com/api/v2/api_tokens
```

> Example response with an API token

```json
{
    "data": {
        "person_id": 1,
        "token": "18:BZWWGQ58QDX8H5B4W4RAJ978Q"
    },
    "meta": {},
    "linked": {}
}
```

> The /me endpoint returns info about the agent account the token/api key is for

```shell
curl -H "Authorization: token 18:BZWWGQ58QDX8H5B4W4RAJ978Q" http://example.com/api/v2/me
```

> Example response:

```json
{
    "data": {
        "auth_method": "api_token",
        "app_id": null,
        "person_id": 1,
        "person": {
            "id": 1,
            "...": "..."
        }
    },
    "meta": {},
    "linked": {}
}
```

## External Authentication Sources

The above example works for DeskPRO-based logins or form-based logins where a username/email address
and password can be gathered from the user and then passed directly to the API for validation. But it does
NOT work for other types of authentication apps such as JWT, SAML, oAuth, etc.

For these types of apps, you need to render a website with a special URL for the user to log in to. When the user
finishes logging in, they are redirect back to DeskPRO which can then communicate the changes up to your app
through Javascript.

### Get a list of usersources

Typically, the first step in your app will be to get a list of usersources that exist so you can render
some kind of button.

> Request user sources used for agents:

```curl
curl http://example.com/api/v2/api_tokens/user_sources/agent
```

> Example response including a Google+ user source:

```json
{
    "data": [
        {
            "id": 5,
            "title": "Google+ Sign-In",
            "type": "agent",
            "source_type": "googleplus",
            "display_type": "social",
            "display_order": 0,
            "display_options": {
                "login_url": "http:\/\/master.dp.devput.com:8080\/api\/v2\/api_tokens\/user_sources\/5\/login?format=default"
            },
            "is_enabled": true
        }
    ],
    "meta": {},
    "linked": {}
}
```

### Handling the login

In your app, you would render a button that loaded the URL in the `data.display_options.login_url` field. In a web app,
this might be rendering an `<a>` tag for example. In a desktop or mobile application, you would initialise a new
web view to accept te login.

After the user logs in, they are returned to DeskPRO which renders a special page like this:

```html
<script>sendPayload({
    "token": "18:BZWWGQ58QDX8H5B4W4RAJ978Q",
    "person_id": 1
})</script>
```

Your app will need to make sure the web view implements this special `sendPayload` function so it can accept the
Javascript object.

# Batch Requests

Sometimes you want to perform multiple queries all at once. For example, maybe you need to load a few collections
to populate a form. Normally you would need to submit multiple HTTP requests which could be slow.

> Example request using a key-value map of requests to execute in a batch:

```shell
curl -H "Content-Type: application/json" \
     -X POST \
     -d '
 {
     "t": {
         "method": "POST",
         "url": "/api/tickets",
         "payload": {
             "title": "new-ticket"
         }
    },
    "john": "/api/people/1",
    "img": "/api/people/1/image",
    "parts": "/api/tickets/5/participants"
 }
'
     http://example.com/api/v2/batch
```

> Example response where each response uses the same key you sent in the request:

```json
{
    "t": {
      "headers": {
          "response_code": 201,

      },
      "data": {
         "title": "new-ticket"
         "...": "..."
      }
    },
    "john": {
      "headers": {
        "response_code": 404
      },
      "data": {
        "...": "..."
      }
    },
    "...": "..."
}
```

You POST a map to `/api/v2/batch` describing the requests you want to submit. The system will process all of your reuqests
at once, and send you back a single result object. The keys of the result will be the same keys you submitted.

## Short GET Syntax

You can also use the short GET syntax to process multiple GET requests all at once:

```shell
curl -H "Content-Type: application/json" \
    http://example.com/api/v2/batch/api/v2/batch?get[stars]=/api/v2/ticket_stars&get[departments]=/api/v2/ticket_departments
```

# Side Loading

A normal request for a resource usually includes IDs of related resources, like so:

# Access Control

Every API request is performed in the context of a specific agent or admin in the system. When you create an API key,
you specify the agent account it is tied to. You can never do anything with the API that the agent/admin account
would not be able to do itself through the web interface.

For example, say you made an API key for John. Using that API key, you could not view, reply, delete etc a ticket
that John himself doesn't already have access to.

This means if you needed some kind of "super" API key that could do _everything_, you would need to make sure the
agent that key was tied to had full permissions in the helpdesk.

## Access control via Tags

Every API endpoint in the system has one or more "tags" associated with it. You can view the tags for a particular
endpoint by finding it in the [Full API Browser](/api/v2/doc). Expand the API endpoint to view the details, and
under the "API tags for this endpoint" heading will be the list of tags applied to the endpoint.

For example, `GET /api/v2/tickets` has a tag of `tickets.tickets.list`. And `DELETE /api/v2/tickets/{id}`
has a tag of `tickets.tickets.delete`.

You can fine-tune access to API endpoitns by specificying a tag pattern when you define the API key in
Admin > Apps > API Keys. In the "Tags" tab, enter one or more tags you want to allow access to.

You can use wildcards to whitelist a whole group of related tags (indeed, the default value for API keys
is to allow _all_ API endpoints so the value is `*`). You can also use a minus symbol to _remove_ specific
actions you don't want to allow.

Examples:

* `*` all API endpoints
* `*, -tickets.*` all API endpoints except tickets
* `tickets.*, -*.delete`' ticket endpoints but nothing to do with deleting
* `tickets.tickets.list` only allow using the ticket listing API

## API Access Modes

Most of the time, most APIs are available to any authenticated user of the API. But some APIs are locked down
so they cannot be used except by users accessing the API by a specific method.

You can view these limitations on each endpoint by viewing the [Full API Browser](/api/v2/doc). In an expanded
node for an endpoint, notice the value for "Applicable API Modes".

* `key` is the msot common and means the API can be used via an API key
* `token` means the API can be used by a token (see below, "API Token Exchange")
* `session` means that the API can be used from the browser if the agent/admin is currently logged in. These are typically
APIs used by the agent or admin interface in DeskPRO itself

You almost never need to worry about this. Typically only internal APIs are marked as session or token only.