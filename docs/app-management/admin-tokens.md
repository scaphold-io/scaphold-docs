# Admin Tokens

With admin tokens, you have the ability to bypass any permissions that have been set in your API. By setting this as `Authorization: Bearer {ADMIN_TOKEN}` in your header
like any other auth token in Scaphold, you become an admin user. This is useful for importing or exporting data, any one-off data management tasks that need to be done, or scheduled jobs.

To create a admin token, navigate to **Settings Page > Admin Tokens > Create**

!!! note

    Admin tokens never expire, unless you delete them.