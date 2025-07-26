# Steam WebAPI Client

This is [yet another](https://www.npmjs.com/search?q=steam+api) node.js module for the
[Steam WebAPI](https://lab.xpaw.me/steam_api_documentation.html). I made it for myself because I'm sure none of the
other modules are any good, but anyone is of course welcome to use it. Don't expect too many updates outside of fixing
stuff.

It detects and handles errors from the `X-eresult` and `X-error_message` headers. It also requests gzip compression for
responses.

# Constructor

The constructor takes three arguments: `key`, `localAddress`, and `accessToken`.

```js
var SteamWebAPI = require('@doctormckay/steam-webapi');

// Using API key
var api = new SteamWebAPI("yourapikey", "192.168.0.5");

// Using access token
var api = new SteamWebAPI(null, "192.168.0.5", "your_access_token");
```

All arguments are optional. You can provide either an API key OR an access token, but not both. If you don't provide 
either, you can only use the methods that don't require authentication.

You can change your API key at any time by assigning it to the `key` property. Same goes for `localAddress` with the
`localAddress` property and `accessToken` with the `accessToken` property.

## Access Token Support

You can also use access tokens instead of API keys. Set them in the constructor or assign directly to the `accessToken` property:

```js
var api = new SteamWebAPI();
api.accessToken = "your_access_token";
```

Note: You cannot set both key and access token simultaneously.

# Methods

### get(iface, method, version[, input], callback)
- `iface` - The WebAPI interface that you want to use. The leading "I" is optional.
    - For example, `IGameServersService` or `SteamUser`.
- `method` - The WebAPI method that you want to use.
    - For example, `GetPlayerSummaries`.
- `version` - The numeric version of the method you want to use.
    - For example, `1`.
- `input` - Optional. An object containing the parameters you want to pass to this request.
    - You shouldn't provide `key` or `format`, as these will be filled automatically.
    - Array inputs (e.g. `input[0]=foo&input[1]=bar`) should be passed as JavaScript arrays. They will be serialized accordingly.
- `callback` - A function to be called when the request completes
    - `err` - An `Error` object on [failure](#failure), or `null` on success
    - `response` - An object containing the [response](#response) data

Performs a GET request to a method.

### post(iface, method, version[, input], callback)

Same as [`get`](#getiface-method-version-input-callback) except with a POST request.

# Failure

On failure, you'll get back an `Error` object with the following properties:

- `statusCode` - The HTTP status code of the response. This may be `200` if an error was detected elsewhere.
- `eresult` - Only if Steam sends back an `X-eresult` header, this will be an [EResult](https://steamerrors.com) value.
- `message` - A string describing the error that occurred. These are the conditions that are checked, in order from top to bottom:
    - If `X-eresult` is returned, is not `1`, and `X-error_message` is returned, then this is the value of `X-error_message`; otherwise,
    - If `X-eresult` is returned and is not `1`, then this is the string name of the EResult value (for example, `NoMatch`); otherwise,
    - If the HTTP status code is not 200, this is the HTTP status message (for example, `Not Found` or `Unauthorized`); otherwise,
    - If the received response wasn't valid JSON, then this is `Malformed response`

# Response

Response data is parsed as JSON. If the top-level object contains exactly one property named `response`, then the value
of the `response` property is returned. Otherwise, the parsed JSON is returned as-is.
