# load.link

Easily upload files (and paste things, and shorten URLs) to your HTTP server and share them with your friends using a nice link.

## Work in progress

Everything. Updates coming soon. Maybe.

## You may also be interested in...

- If you want to make a client and you (rightfully) like Python you may be interested in [115100/ll_functions](https://github.com/115100/ll_functions), a (kind-of-)library that implements all of the API calls.
- If you like Go or (rightfully) hate PHP you should take a look at [moshee/airlift](https://github.com/moshee/airlift). Also his CLI client has a cooler progress bar.


# API

Every function is accessible from the HTTP/JSON API.
- All requests must be POSTed to the API route: `/api` if you're using the PATH routing mode or `?api` if you're using GET.
- All requests must be of `Content-Type: multipart/form-data`
- All requests must have a header part with `Content-Disposition: form-data; name="headers"; filename="headers"` for the actual JSON request.


## get_token

Get an authentication token to be used with other requests.

#### REQUEST

```
{ "action": "get_token",
  "username": "<YOUR_USERNAME>",
  "password": "<YOUR_PASSWORD>" }
```

#### RESPONSE

HTTP Status Code: **200**

```
{ "message": "OK.",
  "token": "<TOKEN>" }
```

##### OR

HTTP Status Code: **403**

```
{ "message": "Access Denied." }
```


## get_links

Get \<LIMIT\> links starting from \<OFFSET\>.

#### REQUEST

```
{ "action": "get_links",
  "limit": "<LIMIT>",
  "offset": "<OFFSET>",
  "token": "<YOUR_AUTHENTICATION_TOKEN>" }
```

#### RESPONSE

HTTP Status Code: **200**

```
{ "message": "OK.",
  "links": [ { "uid": "<ITEM_UID>",
               "path": "<ITEM_PATH>",
               "name": "<ITEM_NAME>",
               "ext": "<ITEM_EXTENSION>",
               "mime": "<ITEM_MIMETYPE>" },
             ... ] }
```


## count

Get the \<TOTAL\> number of items.

#### REQUEST

```
{ "action": "count",
  "token": "<YOUR_AUTHENTICATION_TOKEN>" }
```

#### RESPONSE

HTTP Status Code: **200**

```
{ "message": "OK.",
  "count": <TOTAL> }
```

## get_thumbnail

Get the thumbnail (base64 encoded) of an image item.

#### REQUEST

```
{ "action": "get_thumbnail",
  "token": "<YOUR_AUTHENTICATION_TOKEN>" }
```

#### RESPONSE

HTTP Status Code: **200**

```
{ "message": "OK.",
  "thumbnail": { "data": "<BASE64_ENCODED_THUMBNAIL>",
                 "width": "<THUMBNAIL_WIDTH>",
                 "height": "<THUMBNAIL_HEIGHT>",
                 "mime": "<THUMBNAIL_MIMETYPE>" } }
```

##### OR

HTTP Status Code: **202**

```
{ "message": "Could not get thumbnail." }
```


## upload

Upload an item. In this case you need another part with `Content-Disposition: form-data; name="data"; filename="data"` for the actual file data.

#### REQUEST

```
{ "action": "upload",
  "filename": "<FILENAME>",
  "token": "<YOUR_AUTHENTICATION_TOKEN>" }
```

#### RESPONSE

HTTP Status Code: **201**

```
{ "message": "OK.",
  "uid": "<ITEM_UID>",
  "name": "<ITEM_FILENAME>",
  "mime": "<ITEM_MIMETYPE>",
  "ext": "<ITEM_EXTENSION>",
  "link": "<ITEM_LINK>" }
```

##### OR

HTTP Status Code: **202**

```
{ "message": "Upload Failed." }
```


## shorten_url

Shorten an URL.

#### REQUEST

```
{ "action": "upload",
  "url": "<URL>",
  "token": "<YOUR_AUTHENTICATION_TOKEN>" }
```

#### RESPONSE

HTTP Status Code: **201**

```
{ "message": "OK.",
  "uid": "<ITEM_UID>",
  "link": "<ITEM_LINK>" }
```

##### OR

HTTP Status Code: **202**

```
{ "message": "Shortening Failed." }
```


## delete

Delete an item by its \<UID\>.

#### REQUEST

```
{ "action": "delete",
  "uid": "<UID>",
  "token": "<YOUR_AUTHENTICATION_TOKEN>" }
```

#### RESPONSE

HTTP Status Code: **200**

```
{ "message": "OK." }
```


## edit_settings

Edit any of the settings you can find in default_settings.ini. Any number of them can be put inside the "settings" dictionary.

Be aware that this requires the password to be sent to the API. Just the authentication token isn't enough.

#### REQUEST

```
{ "action": "edit_settings",
  "settings": <SETTINGS_DICTIONARY>,
  "password": "<YOUR_PASSWORD>"
  "token": "<YOUR_AUTHENTICATION_TOKEN>" }
```

#### RESPONSE

HTTP Status Code: **200**

```
{ "message": "OK." }
```

##### OR

HTTP Status Code: **202**

```
{ "message": "Could not update settings. Reason: <ERROR_MESSAGE>" }
```

##### OR

HTTP Status Code: **403**

```
{ "message": "Could not update settings: wrong password." }
```


## release_token

Release the current authentication token.

#### REQUEST

```
{ "action": "release_token",
  "token": "<YOUR_AUTHENTICATION_TOKEN>" }
```

#### RESPONSE

HTTP Status Code: **200**

```
{ "message": "OK." }
```


## release_all_tokens

Release all authentication tokens.

#### REQUEST

```
{ "action": "release_all_tokens",
  "token": "<YOUR_AUTHENTICATION_TOKEN>" }
```

#### RESPONSE

HTTP Status Code: **200**

```
{ "message": "OK." }
```


## prune_unused

Prune unused links (i.e. delete database entries for the items whose associated files you have manually deleted from the server).

#### REQUEST

```
{ "action": "prune_unused",
  "token": "<YOUR_AUTHENTICATION_TOKEN>" }
```

#### RESPONSE

HTTP Status Code: **200**

```
{ "message": "OK."
  "pruned": <NUMBER_OF_PRUNED_ITEMS> }
```

## *Badly formatted requests*

If your request is badly formatted you"ll get the following response:

HTTP Status Code: **400**

```
{ "message": "Badly Formatted Request." }
```

# Alternative Upload Method

Alternatively files may be uploaded by sending a `Content-Type: multipart/form-data` request to the main API endpoint with a single part in this format:

```
Content-Disposition: form-data; name="file"; filename="<FILENAME>"

<PAYLOAD>
```

The request will need to provide an `Authorization: Bearer <YOUR_AUTHENTICATION_TOKEN>` header.

#### RESPONSE

HTTP Status Code: **201**

Payload: `<ITEM_LINK>`

##### OR

HTTP Status Code: **202**

Payload: `Upload Failed.`

##### OR

HTTP Status Code: **403**

Payload: `Access Denied.`
