# Token Expiration

Scaphold gives you full control over how your tokens are managed for authentication. As such, we provide the ability to control the expiration of the JsonWebToken (JWT) measured in seconds
from when it was issued. This refers to the token issued during mutation calls such as `loginUser`, or any social auth flow.

To configure the token expiration time, please navigate to **Settings Page > Advanced**

!!! note

    The default value is 1,296,000 seconds (i.e. 15 days, or ~2 weeks).