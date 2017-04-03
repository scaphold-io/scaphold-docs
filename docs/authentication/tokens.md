Scaphold seamelessly handles user authentication for you. Each Scaphold application comes with a default User model which includes a `username` and `password` that are used to authenticate your users.
We securely encrypt and store each user's password as well as ensure that they are not readable.

## Token

> Example `loginUser` query

```shell
curl -X POST https://us-west-2.api.scaphold.io/graphql/scaphold-graphql \
  -H "Content-Type: application/json" \
  -d '{"query": "mutation LoginUserQuery ($input: LoginUserInput!) { loginUser(input: $input) { token user { id username createdAt } } }",
    "variables": { "input": { "username": "elon@tesla.com", "password": "SuperSecretPassword" } } }'
```

```javascript
import request from 'request';

const data = {
  "query": `mutation LoginUserQuery ($input: LoginUserInput!) {
    loginUser(input: $input) {
      token
      user {
        id
        username
        createdAt
      }
    }
  }`,
  "variables": {
    "input": {
      "username": "elon@tesla.com",
      "password": "SuperSecretPassword"
    }
  }
};

request({
  url: "https://us-west-2.api.scaphold.io/graphql/scaphold-graphql",
  method: "POST",
  json: true,
  headers: {
    "content-type": "application/json",
  },
  body: data
}, (error, response, body) => {
  if (!error && response.statusCode == 200) {
    console.log(JSON.stringify(body, null, 2));
  } else {
    console.log(error);
    console.log(response.statusCode);
  }
});
```

> The above command returns an object structured like this:

```json
{
  "data": {
    "loginUser": {
      "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE0ODI4ODI0ODgsImlhdCI6MTQ4MTU4NjQ4OCwiYXVkIjoiNDRiZTA4NmYtYmYzMy00OTk3LTgxMzYtOWMwMWQ5OWE4OGM0IiwiaXNzIjoiaHR0cHM6Ly9zY2FwaG9sZC5pbyIsInN1YiI6IjcifQ.TDRtD5vD7MIVrViDgVMThhzOzE_teufTo51a4GZ3aGA",
      "user": {
        "id": "VXNlcjo3",
        "username": "elon@tesla.com",
        "createdAt": "2016-12-08T20:43:14.000Z"
      }
    }
  }
}
```

> <b>Important</b>: Use the `token` in the response in the header of future requests as:<br />
`Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE0ODI4ODI0ODgsImlhdCI6MTQ4MTU4NjQ4OCwiYXVkIjoiNDRiZTA4NmYtYmYzMy00OTk3LTgxMzYtOWMwMWQ5OWE4OGM0IiwiaXNzIjoiaHR0cHM6Ly9zY2FwaG9sZC5pbyIsInN1YiI6IjcifQ.TDRtD5vD7MIVrViDgVMThhzOzE_teufTo51a4GZ3aGA`

Logging in a user is simple. Use the `loginUser` mutation we provide you and we will return a JSON Web Token (JWT) if the credentials match. To authenticate a user, you simply set the `Authorization`
HTTP header of your request with the format `Bearer {TOKEN_FROM_LOGIN_USER}`.

This token informs your API what user is logged in at any given time and enables our permissions system to layer access control rules on your data.

<aside class="notice">
  There are two ways to change a user's password:
  <ul>
    <li>
      <b>Reset password</b>: Use the <code>updateUser</code> mutation to update a user's password given a user's <code>id</code>, and new <code>password</code>.
      Note here that no server-side validation for the old password.
    </li>
    <li>
      <b>Forgot password</b>: Use the <code>changeUserPassword</code> mutation to update a user's password given a user's <code>id</code>, <code>oldPassword</code>, and <code>newPassword</code>.
    </li>
  </ul>
</aside>
