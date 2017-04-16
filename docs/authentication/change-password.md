# Change Password

There are two ways to change a user's password:

1. **Reset password**

    Use the `updateUser` mutation to update a user's password given a user's `id`, and new `password`. Note here that no server-side validation for the old password.

2. **Forgot password**

    Use the `changeUserPassword` mutation to update a user's password given a user's `id`, `oldPassword`, and `newPassword`.