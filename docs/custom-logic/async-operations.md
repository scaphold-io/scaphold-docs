# Async Operations

After the final post-operation function finishes, scaphold will immediately return the result to the client that made the request. This is great and keeps your APIs snappy for a friendly user experience. However, sometimes you have tasks that are either long-running or are completely independent of the client app. In our example, we wanted to send push notifications to everyone on the team as soon as the new member joins. This operation would involve paginating through a potentially long list of users and could take an indefinite amount of time.

For these situations use async functions! Async functions behave exactly the same as post-operation functions except Scaphold will not wait for a result after calling out to them. This makes them ideal for long standing tasks and other actions like logging.
