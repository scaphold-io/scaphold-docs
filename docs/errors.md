# Errors

> A typical Scaphold response

```
type ScapholdResult {
    data: any
    errors: [ScapholdError]
}

type ScapholdError {
    message: string!
    code: ErrorCode
    param: string
}
```

<aside class="notice">
Errors from the Scaphold API will be returned as a list under the `errors` attribute at the root of the response.
I.E. Scaphold responses adhere to the ScapholdResult type.
</aside>

Error Code | Meaning
---------- | -------
200 | OK -- Everything is working as expected
400 | Bad Request -- There was an error in your request.
401 | Unauthorized -- Your API token is wrong or missing
402 | Payment Required -- You have exceeded the free pricing teir. Please input a payment option via the Account page.
403 | Forbidden -- You have a valid API token but you do not have permission to this page.
404 | Not Found -- The specified resource could not be found.
429 | Too Many Requests -- You have approached the 60 req/s limit. Please try to spread out your requests more evenly.
440 | Login Failure -- We either couldn't find a user with that email or your password was incorrect.
441 | Registration Failure -- Invalid registration information. E.G. a user with that username already exists.
500 | Internal Server Error -- We had a problem with our server. Please contact us via slack or try again later.
503 | Service Unavailable -- We're temporarially offline for maintanance. Happens very rarely.
