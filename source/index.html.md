---
title: API Reference

language_tabs:
  - shell

search: true
---

# Getting Started

> Here's an example call made to the `helpdesk/discover` endpoint:

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

The DeskPRO API is a REST-based API that runs over HTTP(S). All API requests are made to a URL that begins with
`http://example.com/api/v2/`.

<aside class="notice">
All API requests are under the path <strong>/api/v2/</strong>. Note the "v2" is important!
</aside>

Since all API requests are simply HTTP requests, it's easy to use in any programming language. For the purposes of demonstration, we'll use cURL in our examples here.

## API Browser

On any DeskPRO instance, browse to `/api/v2/doc` to view a full auto-generated API browser.

For example, `http://example.com/api/v2/doc`.

The auto-generated API browser can help if you need to quickly locate a particular API or if you want to see metadata about a particular API endpoint (such as tags, versions or stability flags).

## Authentication

```shell
curl -H "Authorization: key 2:YWK2PGCS8CNW62SR8TBTBKWMY" \
	http://example.com/api/v2/me
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

The above example is one of the few API endpoints that is public. Almost every API endpoint in DeskPRO that
is actually useful requires authentication. The most common way to use the API is via an API key. An admin can
create keys from Admin > Apps > API Keys.

To authenticate an API request, you send an Authorization header like `Authorization: key YOURKEY`. Your API key
will be an integer, followed by a colon, followed by a long random string of characters.

> Example request to the /me endpoint to see info about the agent the key is assigned to:

## API Versions and Backwards Compatibility

> API request without a version prefix
> (always using the latest API version)

```shell
curl -H "Authorization: key 2:YWK2PGCS8CNW62SR8TBTBKWMY" \
	http://example.com/api/v2/me
```

> API request with a version prefix

```shell
curl -H "Authorization: key 2:YWK2PGCS8CNW62SR8TBTBKWMY" \
	http://example.com/api/v2/20161122/me
```

The major API version is version 2, as denoted by the path prefix `/api/v2/`. But you may also specify a hard-coded date in the format of `YYYYMMDD` to guarantee API compatibility for that date.

Examples:

* `/api/v2/20161122/tickets`
* `/api/v2/20150101/tickets`
* `/api/v2/tickets`

If you do NOT specify a date segment (such as the last example above), then the API endpoint you use will always be the latest.

<aside class="notice">
The version date can be any arbitrary date. For example, if you started writing an app, you would be wise to simply use the date for today as that is when you started using the API.
<br/><br/>
When the system routes your API request, it will look at the date and will choose the closest version to your requested date without going over.
</aside>

We guarantee that the API for a particular version will remain compatible, even if you upgrade DeskPRO.

This means that the "shape" of JSON payloads (both input requests and output responses) will remain the same. Though we may make _additions_ which are optional and do not change the overall behaviour of the API.

For example, we might add a new input parameter to `PUT /tickets` for a new feature. But if your application code doesn't submit this parameter, then it doesn't matter; the API would just function the way it always did. This is an example of an API change that is backwards compatible and as such it would not constitute a breaking change and we would not create a new API version for it.

In rare cases, we may need to make a _breaking change_ (for example, to fix a bug that affects actual behaviour). In those cases a new API version will be created. If you are using the API with a version prefix as described above, then you will continue to use the old version and no changes will be required in your application. But if you are not using a version prefix (which means you are always using the "latest API"), then it's possible a breaking change will require you to update your code.

<aside class="warning">
It is best practice to always include a date version prefix in your API request to guarantee API compatibility. If you do not, then it will ALWAYS be the latest API which may (rarely) introduce breaking changes as you upgrade DeskPRO.
</aside>

**API Endpoint Stability**

Note that some API endpoints are marked as _unstable_ in the API browser. Unstable APIs means they may change without warning, and without a new API version being issued.

This means that if you are using an unstable API, upgrading DeskPRO may result in that API being changed in a backwards-incompatible way without notice.

These APIs are generally APIs intended for internal use, or are otherwise experimental. If you intend to use an unstable API, you should submit a ticket to support@deskpro.com requesting the API become stable.

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
    "meta": {},
    "linked": {}
}
```

* `data` is the JSON representation of the resource requested, or an array of objects if you requested a collection
* `linked` optional; when specified, it is an object that contains additional entities requested in `?include` param.
* `meta` optional; misc information that varies based on endpoint. Resource collections should include “count”, “total_count*, “current_page”, “total_pages” under "pagination" key.

### Collection Responses

> Example request to /tickets which is a collection resource:

```shell
curl -H "Authorization: key 2:YWK2PGCS8CNW62SR8TBTBKWMY" http://example.com/api/v2/tickets?include=person
```

> Example response of a collection resource

```json
{
   "data": [
      {
        "id": 1,
        "person": 4,
        "subject": "Foo Bar",
        "...": "..."
      },
      {
        "id": 2,
        "person_id": 12,
        "subject": "Fizz Buzz",
        "...": "..."
      }
   ],
  "linked": {
        "person": {
          "4": {
            "id": 4,
            "first_name": "John",
            "last_name": "Doe"
          }
        }
   },
   "meta": {
      "pagination": {
        "total": 5445,
        "current_page": 1,
        "per_page": 10,
        "total_pages": 545
      }
   }
}
```

> Example fetching ticket objects for 3 specific ticket IDs

```shell
curl -H "Authorization: key 2:YWK2PGCS8CNW62SR8TBTBKWMY" \
	http://example.com/api/v2/tickets?ids=123,456,789
```

During a resource collection request, you can specify the following optional query parameters:

* `count` the number of results to return
* `page `the page you are requesting

Collection responses are always an _array_  of objects inside of `data`.

`meta` will contain some useful data under `pagination` key:

* `total` is the total count (all matching resources)
* `current_page` is current page number
* `per_page` is a count of the number of resources returned to you on this page
* `total_pages` is the total number of pages for this request

**Tip: Most collections accept a `ids` parameters**

Most collection resources accept an `ids` parameter where you can provide specific IDs for resources you want to retrieve. This makes it easy to load multiple objects at once.

### Creating and Updating Content

When you update a resource with `POST`, the API will simply return a `204 No Content` empty response to acknowledge that
the request succeeded. When you create a resource with a `PUT` request, the API will return a `201 Created` response.

In both cases, a `Location` header will be supplied with the full URL to the API endpoint where
you can `GET` the resource.

TIP: If you would rather your `POST/PUT` request return the full resource automatically (i.e. you want to use it right away),
you can submit your request with the `?follow_location=1` query string parameter. This will cause the API
to return the resource as if you did the GET yourself.

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

### API Key Limits

You can specify API limits on every API key you create. For example, it is not uncommon to set an upper limit on
an API key to prevent some kind of mistake in it's use (such as a flood or infinite loop situation). You can define
these limits in terms of an hourly limit or a daily limit.

For cloud customers, there is also default global limit to prevent abuse. You can raise this on request by contacting
 us at support@deskpro.com.
 
When you hit a rate limit, requests will begin to fail with a HTTP status code `429 Too Many Requests` error.

## Error Responses

```
{
    "status": 400,
    "code": "invalid_json_body",
    "message": "Invalid JSON body"
}
```

Error responses are also JSON and follow the shape described here.

* The `status` will be the HTTP status code that most closely matches the error.
* The `code` is a DeskPRO specific string to represent the specific error that happened. You may
wish to use this value in your code in an if/switch etc to base your error handling on.
* The `message` is a human-readable English string that describes what went wrong. This value is typcially
only useful for developers.

### Detailed Errors

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

In some cases, detailed error messages are available, such as when there are form validation errors.

The root `errors` property is an object that has two properties:

* `errors.errors` is a JSON array of error objects that happened on a global level (not specific to any one input).
* `errors.fields` is a JSON object that has a property for every expected input name that had an error.
  Inside of each field is a property named “errors” which is an array of error objects (code/message).

# API Token Exchange

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
curl -H "Authorization: token 18:BZWWGQ58QDX8H5B4W4RAJ978Q" \
	http://example.com/api/v2/me
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

You can also use the API to exchange an agent login for an API token which can be used to authenticate requests.
An API token is similar to an API key except that it doesn't need to be created by an admin. It's created
by the system when given a successful set of credentials.

A token is used in the same way except the Authorzation header looks like `Authorization: token YOURTOKEN`. This is
the same as the API key method except it starts with the word "token" instead of "key".

**`POST /api_tokens`**

| Param | Description | Example |
| --- | --- | --- |
| `email` | Email address or username | user@example.com, my_username |
| `password` | Plaintext password | mypassword |

## External Authentication Sources

> Request user sources used for agents:

```curl
curl http://example.com/api/v2/api_tokens/user_sources/agent
```

> Example response including a Google+ user source:
> You could then render a button "Google+ Sign-In" which loads the login_url when clicked

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
                "login_url": "http:\/\/example.com\/api\/v2\/api_tokens\/user_sources\/5\/login?format=default"
            },
            "is_enabled": true
        }
    ],
    "meta": {},
    "linked": {}
}
```

The above example works for DeskPRO-based logins or form-based logins where a username/email address
and password can be gathered from the user and then passed directly to the API for validation. But it does
NOT work for other types of authentication where we need to send the user off to some external site (e.g. JWT, SAML, oAuth, Google+, Facebook, etc).

For these types of apps, you need to render a website with a special URL for the user to log in to. When the user
finishes logging in, they are redirect back to DeskPRO which can then communicate the changes up to your app
through Javascript.

Typically, the first step in your app will be to get a list of usersources that exist so you can render
some kind of button for external usersources.

## Handling External Login

In your app, you would render a button that loaded the URL in the `data.display_options.login_url` field. In a web app,
this might be rendering an `<a>` tag for example. In a desktop or mobile application, you would initialise a new
web view to accept te login.

After the user logs in, they are returned to DeskPRO which renders a special page like this:

> After a user successfully logs in on the remote site, they are redirected
> back to DeskPRO which renders this special script in your webview:

```html
<script>sendPayload({
    "token": "18:BZWWGQ58QDX8H5B4W4RAJ978Q",
    "person_id": 1
})</script>
```

Your app will need to make sure the web view implements this special `sendPayload` function so it can accept the
Javascript object.

# Batch Requests

> Example request using a key-value map of requests to execute in a batch:

```shell
curl -H "Authorization: key 1:CVRRGQ58QDX8H5B4W4RAJ978Q" \
     -H "Content-Type: application/json" \
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

Sometimes you want to perform multiple queries all at once. For example, maybe you need to load a few collections
to populate a form. Normally you would need to submit multiple HTTP requests which could be slow.

You POST a map to the `/batch` endpoint describing the requests you want to submit. The system will process all of your requests
at once, and send you back a single result object. The keys of the result will be the same keys you submitted.

**`POST /batch`**

Post an object/map (top-level) with the following schema:

| Property | Description | Example |
| --- | --- | --- |
| Note: `KEY` | Any arbitrary string you want to use to identify the request. | req_1, department_info, foobar, 12 |
| `KEY.url` | The API endpoint you want to call. You can also specify query-string params here. | `/api/v2/me`, `/api/v2/tickets?page=2` |
| `KEY.method` | (optional) GET, POST, DELETE, PUT. Defaults to GET. | GET |
| `KEY.payload` | (optional) When using POST/PUT, this is the JSON object you want to submit as the payload itself. | `{"foo": "bar"}` |


## Short GET Syntax

You can also use the short GET syntax to process multiple GET requests all at once by specifying multiple `get[KEY]=endpoint_path` parameters.

**`GET /batch`**

| Param | Description | Example |
| --- | --- | --- |
| Note: `KEY` | The 'key' in `get[KEY]` can be any arbitrary string. It serves the same purpose as KEY above in the POST version of batch. | |
| `get[KEY]` | The API endpoint to fetch | `/api/v2/me`, `/api/v2/tickets` |

If you need to specify parameters for any of the `get[KEY]` calls (for example, to specify a page number), you can use the extended syntax.

> Example GET call
> Note: For purposes of readability, the URL has been split on multiple lines.

```shell
curl -H "Authorization: key 1:CVRRGQ58QDX8H5B4W4RAJ978Q" \
    http://example.com/api/v2/batch/api/v2/batch?
	    get[stars]=/api/v2/ticket_stars
	    &get[departments]=/api/v2/ticket_departments
```

> Here's an example where the endpoint contains parameters.
> You need to encode '?' and '&' so they don't get read by the main batch request itself.

```shell
curl -H "Authorization: key 1:CVRRGQ58QDX8H5B4W4RAJ978Q" \
    http://example.com/api/v2/batch/api/v2/batch?
	    get[stars]=/api/v2/ticket_stars
	    &get[tickets]=/api/v2/tickets%3Fpage%3D2%26ids%3D1%2C2%2C3%2C4
	    
# the last line decodes to a URL like:
# /api/v2/tickets?page=2&ids=1,2,3,4
```

# Side Loading

> Example ticket response without side loading

```shell
curl -H "Authorization: key 1:CVRRGQ58QDX8H5B4W4RAJ978Q" \
	http://example.com/api/v2/tickets/123
```

> Example response. Notice how we have IDs for various relationships such as the person, agent or department.

```json
{
    "data": {
        "id": 123,
        "ref": "XXXX-0028-IOCC",
        "department": 20,
        "person": 59080,
        "agent": 2,
        "status": "awaiting_user",
        "subject": "Example Ticket"
    },
    "meta": [],
    "linked": []
}
```

> Example WITH sideloading on department and person

```shell
curl -H "Authorization: key 1:CVRRGQ58QDX8H5B4W4RAJ978Q" \
	http://example.com/api/v2/tickets/123?include=department,person
```

> Example response with sideloaded objects

```json
{
    "data": {
        "id": 123,
        "ref": "XXXX-0028-IOCC",
        "department": 20,
        "person": 59080,
        "agent": 2,
        "status": "awaiting_user",
        "subject": "Example Ticket"
    },
    "meta": [],
    "linked": {
        "department": {
            "20": {
                "id": 20,
                "title": "Support"
            }
        },
        "person": {
            "2": {
                "id": 2,
                "name": "John Doe",
                "primary_email": "agent@example.com"
            },
            "59080": {
                "id": 59080,
                "name": "Fionna Apple",
                "primary_email": "user@example.com"
            }
        }
    }
}
```

Most objects in DeskPRO have other objects that are related to them. For example, a ticket has various users associated with it such as the creator, the assigned agent and any followers or CC'd users.

By default, the API will only return the IDs of these related objects and if you needed more information, you would need to use the API again to fetch the objects you wanted (for example, to get the actual title of the department a ticket was in).

However, this is tedious if you are doing it a lot. For this reason, the DeskPRO API has the idea of _side-loading_ which instructs the API to return related objects of certain types. You do this by specifying the type of object you want to side-load in the request as a query parameter: `include=type1,type2,type3`.

Examples:

* `/tickets/123?include=department`
* `/tickets/123?include=department,person,category,product,workflow`
* `/people/456?include=usergroup`

When you specify an `include` parameter, the related objects are returned in the response under the `linked` object. You'll get properties like `linked.TYPENAME.ID`.

(Note that the actual data response stays the same. Your own app logic will need to connect the ID from the `data` to the object returned in the `linked` section.)

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
endpoint by finding it in the [Full API Browser](#api-browser). Expand the API endpoint to view the details, and
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

You can view these limitations on each endpoint by viewing the [Full API Browser](#api-browser). In an expanded
node for an endpoint, notice the value for "Applicable API Modes".

* `key` is the msot common and means the API can be used via an API key
* `token` means the API can be used by a token (see below, "API Token Exchange")
* `session` means that the API can be used from the browser if the agent/admin is currently logged in. These are typically
APIs used by the agent or admin interface in DeskPRO itself

You almost never need to worry about this. Typically only internal APIs are marked as session or token only.
