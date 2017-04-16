# Uploading Files

Uploading files is simple. All you need to do is attach the file to a multipart/form-data request
and point to it using the `blobFieldName` attribute in the `Blob` implemented type's input object.

All types that implement `Blob` will receive an additional input field called `blobFieldName`.

!!! tip ""

    Example file upload

!!! note ""

    Mark Zuckerberg's user ID: `VXNlcjoxMA==`

## cURL

```shell
curl -v https://us-west-2.api.scaphold.io/graphql/scaphold-graphql \
  -H "Content-Type:multipart/form-data" \
  -F 'query=mutation CreateFile($input: CreateFileInput!) { createFile(input: $input) { changedFile { id name blobMimeType blobUrl user { id username } } } }' \
  -F 'variables={ "input": { "name": "Profile Picture", "userId": "VXNlcjoxMA==", "blobFieldName": "myBlobField" } };type=application/json' \
  -F myBlobField=@mark-zuckerberg.jpg
```

## JavaScript

```javascript

// You must also install these npm modules:
// npm install --save form-data
// npm install --save node-fetch

var fetch = require('node-fetch');
var FormData = require('form-data');
var fs = require('fs');

var form = new FormData();
form.append("query", `
  mutation CreateFile($input: CreateFileInput!) {
    createFile(input: $input) {
      changedFile {
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
  }
`);
form.append("variables", JSON.stringify({
  "input": {
    "name": "Mark Zuck Profile Picture",
    "userId": "VXNlcjoxMA==",
    "blobFieldName": "myBlobField"
  }
}));
// The file's key matches the value of the field `blobFieldName` in the variables
form.append("myBlobField", fs.createReadStream('./mark-zuckerberg.jpg'));

fetch("https://us-west-2.api.scaphold.io/graphql/scaphold-graphql", {
  method: 'POST',
  body: form
}).then(function(res) {
  return res.text();
}).then(function(body) {
  console.log(body);
});
```

## React Native

```javascript
// You must also install these npm modules:
// npm install --save react-native-fetch-blob
// react-native link react-native-fetch-blob
import RNFetchBlob from 'react-native-fetch-blob';

RNFetchBlob.fetch(
  'POST',
  "https://us-west-2.api.scaphold.io/graphql/scaphold-graphql",
  {
    Authorization : "Bearer " + scapholdAuthToken,
    'Content-Type' : 'multipart/form-data',
  },
  [
    {
      name : 'query',
      data : `
        mutation CreateFile($input: CreateFileInput!) {
          createFile(input: $input) {
            changedFile {
              id
              blobMimeType
              blobUrl
              user {
                id
              }
            }
          }
        }
      `
    },
    {
      name : 'variables',
      data : JSON.stringify({
        "input": {
          "name": "Mark Zuck Profile Picture",
          "userId": "VXNlcjoxMA==",
          "blobFieldName": "myBlobField"
        }
      })
    },
    {
      name : 'myBlobField',
      filename : 'profile',
      data: RNFetchBlob.wrap(filePath),
    },
  ]
);
```

---

The above command returns an object structured like this:

```json
{
  "data": {
    "createFile": {
      "changedFile": {
        "id": "RmlsZTo5",
        "name": "Mark Zuck's Profile Picture",
        "blobMimeType": "image/jpeg",
        "blobUrl": "https://s3-us-west-2.amazonaws.com/production.us-west-2.scaphold.v2.customer/44be086f-bf33-4997-8136-9c01d99a88c4/data/2fb4b11d-cef9-465d-9ad6-e3d8b693f121/2b86488a-7114-4071-9e30-157855475eb7?AWSAccessKeyId=AKIAJIC3JY2ICINJH2OQ&Expires=1481686711&Signature=pa4QbkPCk%2BXlgSrKBWcRKsEckSs%3D",
        "user": {
          "id": "VXNlcjoxMA==",
          "username": "Mark Zuckerberg"
        }
      }
    }
  }
}
```