# Function Composition

Function composition is a technique that combines simple functions into more complicated ones. At Scaphold, we use function composition to create modular, testable, and maintainable GraphQL resolvers. Logic allows you to do the same.

---

Let's walk through an example of how you might build an app like slack. Let's say a new user was invited to create an account and to join a particular team. This would take a couple of steps.

<ol>
  <li>A user signs up and includes the name of the team.</li>
  <li>Before the user is created we validate the user's email then authorize that they can join the team.</li>
  <li>The user object is persisted to your DB hosted on Scaphold.</li>
  <li>After the user is created, we use the generated ID to add the user to the Many-To-Many connection with the team.</li>
  <li>The new user is returned to the client along with the new connection to the team.</li>
  <li>After the payload was delivered to the client, we asynchronously send push notifications to everyone in the team informing that a new user has joined.</li>
</ol>

---

Complex workflows like this were exactly why we built Logic. Let's break down how we would use logic to build it.