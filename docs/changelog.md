# Changelog

## May 7, 2017

- Added ability to add connections in the "Add Type" and "Edit Type" side bars

- Bug fixes:
    - Enums now allow single character values
    - Fix authentication error in Scaphold admin portal for expired or malformed tokens

## April 22, 2017

- Added **new** [Hobby pricing tier](https://scaphold.io/pricing)

- Algolia `query` input field is no longer mandatory

- Bug fixes:
    - Fixed font-face for non-Mac environments

## April 12, 2017

- Added [Algolia geo-search query parameters](/integrations/search/#geo-search)

- Updated docs and layout

## April 3, 2017

- Check expired and/or invalid token if `Authorization` header is provided

- Authorization errors for invalid tokens now throw `401` error status codes instead of `403`

- `401` errors are thrown if auth token is malformed now

- Bug fixes:
    - Top-level error handling should not throw stack trace

## March 21, 2017

- Upgraded subscriptions package to Apollo 1.0

## March 10, 2017

- Added [AND & OR operators](/coredata/queries/#querying-with-and-or)

- Allow enum connections

- Bug fixes:
    - Algolia delete indexes after integration is deleted
    - Stripe integration via custom logic
    - Role scoped permissions update for admin user

## February 22, 2017

- List of date errors

- Bug fixes:
    - Updates with logic functions causing problems on return values.

## February 17, 2017

- Added [persisted queries](/persisted-queries/)

- Added `JSON` type for fields

- Bug fixes:
    - Fix for iOS and Android push notifications
    - Return unmasked Stripe IDs
    - Apollo Optics bug to stop background tasks

## February 9, 2017

- Deeply nested mutation permissions: you can now use deep `userFields` to authorize permissions for mutations multple models away.

- `node` on `viewer` now returns app ID

## February 1, 2017

- `self` option for permissions

- Formatting for type descriptions

- Custom payment plans now an option

## January 23, 2017

- UTF8 databases

- Bug fixes:
    - Fix for `_typename` in aggregations.

## January 17, 2017

- Added [aggregations on connections](/aggregations/basics/)

## January 9, 2017

- [Scaphold Community Page](https://scaphold.io/community/)! Find all the resources you need to launch a production app with GraphQL.

- Bug fixes:
    - Subscriptions filter with ID now accepts opaque unique ID properly

## January 6, 2017

- [Custom Logic](#custom-logic): Use Scaphold's powerful webhook system to define custom business logic for your apps.

- Added enums as values to `orderBy` arguments on connections

## December 28, 2016

- Extended ordering functionality on a connection to order by an enum field

## December 26, 2016

- [New landing page](https://scaphold.io)!

- [User authentication](#token):
    - Reset password
    - Forgot password

- Better error handling:
    - JSON syntax errors in Query Variables (no need to input empty object literal anymore)
    - Accessing apps that don't exist or belong to you won't hang on loading page anymore

- Updated [referral email](https://scaphold.io/referral)!

- DateTime input form in Explorer page

## December 7, 2016

- [Multiple Regions](#regions): We're launched to two regions now - US West (Oregon) and EU West (Ireland).
Please message us if you have a need for a new data center in your region!

- [Super Tokens](#super-tokens): Create a new super token with a click of a button. Use this for administrative tasks.

- [Token Expiration](#token-expiration): Configure the expiration time of your JWT tokens for your users.
