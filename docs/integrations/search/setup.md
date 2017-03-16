## Search

[See the full tutorial](tutorials/sending-push-notifications/)

### Setup the Algolia integration

The Algolia integration lets you instantly add search capabilities to your API. As soon as you enable the integration and select the types
that you would like indexed, we immediately push your data to Algolia. Scaphold will then make sure that your data held on our servers
is kept in sync with your data on algolia.

1. Create a free [Algolia](https://www.algolia.com/) account. This will help you manage your search indices and monitor your usage. Once you've created your account, you'll receive an Application ID and an API Key.

<aside class="notice">
        You can share a single algolia account for multiple apps. Each index we create for you is unique to the app and type being indexed.
</aside>

2. Configure **Algolia** in Scaphold from the [Integrations Portal](https://scaphold.io/apps) and select the checkboxes in the popup to index data of that particular type.

<img class="news-how-to-img" src="/images/integrations/Algolia_Index.png" alt="Configure your Algolia integration">

3. As soon as you enable search on a particular type, Scaphold will automatically index each object in your API
as well as ensure that each create, update, and delete mutation is mirrored to Algolia. This makes id dead simple to combine
the best of Algolia with the best of GraphQL and Scaphold.

4. You can use the Algolia portal to fine tune your indexes to improve the search experience for your apps.