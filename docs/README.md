# Recurly Python

This library is the python client for Recurly's V3 API. It's currently in the "Early Access" phase of development.
To learn more about getting early access to the next iteration of our API ecosystem, see [this link](https://dev.recurly.com/page/recurly-v3-api-early-access).

If you were looking for the V2 client, see the [master branch](https://github.com/recurly/recurly-client-python/tree/master) of the recurly-client-python repo.

## Getting Started

### Installing

This library is published on pypi as a pre-release. We recommend targeting a specific version
in your requirements.txt or setup.py:

```
recurly==3.0b5
```

Until we reach GA with version `3.0.0`, we will bump the beta level on each release `3.0bX` by 1.
Some of these releases may contain breaking changes. We will try to document these
in the release notes.

### Importing the library

We recommend importing `recurly` and preserving the namespace:

```python
import recurly
```

### Reference Docs Overview

1. The [Client class](recurly.html?highlight=client py#recurly.client.Client) documents all the endpoints in the API as methods.
2. The [resources module](recurly.html?highlight=clientpy#module-recurly.resources) documents all responses (resources) that are returned from client calls.
3. The [errors module](recurly.html?highlight=clientpy#module-recurly.errors) documents all API errors that a client method might throw.

You can [the search page](search.html) to search across all calls, errors, resources, attributes, etc.

### Creating a client

A client represents a connection to the Recurly servers. Every call
to the server exists as a method on this class. To initialize, a private api key.

```python
api_key = '83749879bbde395b5fe0cc1a5abf8e5'
client = recurly.Client(api_key)
```

### Operations

The Client contains every `operation` you can perform on the site as a list of methods.
See all operations listed on the documentation for the [Client class](recurly.html?highlight=client py#recurly.client.Client).

```python
account = client.get_account("code-benjamin.dumonde@example.com")
#=> returns a recurly.Account object
```

### Pagination

There are 2 methods for pagination:

1. Per-page
2. Per-item

*Warning: These are being worked on and will likely change*

All `client.*_list` methods return a `Pager`. On the pager, `pages()` returns a `PageIterator` and `items()`
returns an `ItemIterator`.

#### per-page
```python
pages = client.list_accounts(limit=200).pages()
for page in pages:
    for account in page:
        print(account.code)
```

#### per-item
```python
accounts = client.list_accounts(limit=200).items()
for account in accounts:
    print(account.code)
```

### Creating Resources

Resources are created by passing in a `body` argument in the form of a `dict`.
This dict must follow the schema of the documented request type. For example, see the
[create_account operation doc](https://developers.recurly.com/api/v2019-10-01/index.html#operation/create_account)
to understand what parameters may be sent to create an account.

```python
account_create = {
    "code": "benjamin.dumonde@example.com",
    "first_name": "Benjamin",
    "last_name": "Du Monde",
    "shipping_addresses": [
        {
            "nickname": "Home",
            "street1": "1 Tchoupitoulas St",
            "city": "New Orleans",
            "region": "LA",
            "country": "US",
            "postal_code": "70115",
            "first_name": "Aaron",
            "last_name": "Du Monde"
        }
    ]
}
account = client.create_account(account_create)
#=> returns a recurly.resources.Account object
```

### Error Handling

This library currently throws 2 primary types of exceptions:

1. `recurly.ApiError` when the Recurly API server returns an error.
2. `recurly.NetworkError` when the connection to the Recurly API server fails.

The `ApiError` comes in a few flavors which help you determine what to do next.
They are thrown as exceptions. To see a full list, view the [errors module docs](recurly.html?highlight=errors#module-recurly.errors).

```python
try:
    expired_sub = client.terminate_subscription(subscription.id, refund='full')
except recurly.errors.ValidationError as e:
    # If the request was invalid, you may want to tell your user
    # why. You can find the invalid params and reasons in e.error.params
    print("ValidationError: %s" % e.error.message)
    print(e.error.params)
except recurly.errors.NotFoundError as e:
    print("Some id was not found, probably the subscription.id. The error message will explain:")
    print(e)
except recurly.errors.ApiError as e:
    print("Generic catch all for all recurly specific errors")
    print(e)
except recurly.NetworkError as e:
    print("Something happened with the network connection.")
    print(e)
```

### HTTP Metadata

Sometimes you might want to get some additional information about the underlying HTTP request and response. Instead of
returning this information directly and forcing the programmer to unwrap it, we inject this metadata into the top level
resource that was returned. You can access the [Response](recurly.html?highlight=response#recurly.response.Response) by
calling `get_response()` on any Resource.

**Warning**: Do not log or render whole requests or responses as they may contain PII or sensitive data.


```python
account = client.get_account("code-benjamin")
response = account.get_response()
response.rate_limit_remaining #=> 1985
response.request_id #=> "0av50sm5l2n2gkf88ehg"
response.request.path #=> "/sites/subdomain-mysite/accounts/code-benjamin"
response.request.body #=> None
```

This also works on [Empty](recurly.html?highlight=empty#recurly.resource.Empty) responses:

```python
response = client.remove_line_item("a959576b2b10b012").get_response()
```
And it can be captured on exceptions through the [Error](recurly.html?highlight=error#recurly.resources.Error) object:

```python
try:
    account = client.get_account(account_id)
except recurly.errors.NotFoundError as e:
    response = e.error.get_response()
    print("Give this request id to Recurly Support: " + response.request_id)
```