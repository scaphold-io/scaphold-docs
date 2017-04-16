# Querying Files

Querying files acts the same as other types. All types that implement `Blob` have a field `blobUrl` and `blobMimeType`
that are automatically managed by Scaphold. The `blobUrl` is a pre-signed URL that points to your file in a private
blob store hosted on Amazon S3. If youre app is on a paid tier, all pre-signed URLs will be served by
a globally distributed CDN hosted by Amazon CloudFront.

!!! tip ""

    Example of querying for a file

!!! note ""

    File ID: `RmlsZTo5`

## cURL

```shell
curl -X POST https://us-west-2.api.scaphold.io/graphql/scaphold-graphql \
  -H "Content-Type: application/json" \
  -d '{ "query": "query GetFile { getFile(id: \"RmlsZTo5\") { id name blobMimeType blobUrl user { id username } } }",
    "variables": {} }'
```

## JavaScript

```javascript
var request = require('request');

var data = {
  "query": `
    query GetFile {
      getFile(id: "RmlsZTo5") {
        id
        name
        blobMimeType
        blobUrl
        user {
          id
          username
        }
      }
    }
  `,
  "variables": {}
}

request({
  url: "https://us-west-2.api.scaphold.io/graphql/scaphold-graphql",
  method: "POST",
  json: true,
  headers: {
    "content-type": "application/json",
  },
  body: data
}, function(error, response, body) {
  if (!error && response.statusCode == 200) {
    console.log(JSON.stringify(body, null, 2));
  } else {
    console.log(error);
    console.log(response.statusCode);
  }
});
```

---

The above command returns an object structured like this:

```json
{
  "data": {
    "getFile": {
      "id": "RmlsZTo5",
      "name": "Mark Zuck's Profile Picture",
      "blobMimeType": "image/jpeg",
      "blobUrl": "https://s3-us-west-2.amazonaws.com/production.us-west-2.scaphold.v2.customer/44be086f-bf33-4997-8136-9c01d99a88c4/data/2fb4b11d-cef9-465d-9ad6-e3d8b693f121/2b86488a-7114-4071-9e30-157855475eb7?AWSAccessKeyId=AKIAJIC3JY2ICINJH2OQ&Expires=1481690209&Signature=icwTcNl%2B%2BTwQy8Ar6jLuquztwu0%3D",
      "user": {
        "id": "VXNlcjoxMA==",
        "username": "Mark Zuckerberg"
      }
    }
  }
}
```