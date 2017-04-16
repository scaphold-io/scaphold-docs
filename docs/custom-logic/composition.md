# Function Composition

Function composition is a technique that combines simple functions into more complicated ones. At Scaphold, we use function composition to create modular, testable, and maintainable GraphQL resolvers. Logic allows you to do the same.

---

!!! tip "Example"

Let's walk through an example of how you might build an app like slack. Let's say a new user was invited to create an account and to join a particular team. This would take a couple of steps.

1. A user signs up and includes the name of the team.
2. Before the user is created we validate the user's email then authorize that they can join the team.
3. The user object is persisted to your DB hosted on Scaphold.
4. After the user is created, we use the generated ID to add the user to the Many-To-Many connection with the team.
5. The new user is returned to the client along with the new connection to the team.
6. After the payload was delivered to the client, we asynchronously send push notifications to everyone in the team informing that a new user has joined.

---

Complex workflows like this were exactly why we built Logic. Let's break down how we would use logic to build it.