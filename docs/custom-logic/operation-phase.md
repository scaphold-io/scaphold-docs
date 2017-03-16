# The Operation Phase

After all of your pre-operation functions have finished, we take the input and persist it to the storage layer.


The operation phase is where Scaphold does everything it needs to persist the object. The operation
phase often involves saving the object to the database, uploading files to blob storage, and firing
events for subscriptions. The operation phase will also sometime involve integrations. For example,
if you have turned on the Algolia integration, this phase handles indexing the object in your Algolia
account for you.
