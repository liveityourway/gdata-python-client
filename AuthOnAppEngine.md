# Introduction #

On App Engine, it makes sense to store an auth token in the datastore and associate it with a user. Care must be taken to ensure that a token is only used for the correct user. The token allows access to a user's data, but there is no information in the token about whose data it grants access to.

# Use Cases #

1. All users are required to log in to the app, the app accesses a Google Data service for each user individually.

2. The app uses one master account on the backend to talk to Google Data services. These apps often use ClientLogin and embed the email address and password in the code.

3. The app doesn't require users to log in to the app, but each user authorizes the app to access their Google data individually.

# Previous Solutions #

Solution 0:
Originally, developers needed to write their own token management code. They decided how to store and retrieve tokens.

Solution 1:
Already implemented, but doesn't cover some use cases. Each client object has a token\_store to which tokens can be added. When a token is added, the lib looks for the current\_user: if a user is found, the token is stored and associated with the user, if no user is found the token is not stored.

Supports use case 1 but not 2 or 3.

Workaround to support use case 2 is to replace the datastore-based token\_store with the in-memory token store using `client.token_store = atom.token_store.TokenStore()`

# Proposal #

Right now, users modify a client to run on App Engine by using:
```
gdata.alt.appengine.run_on_appengine(gdata_client)
```
We will add new parameters to this function to specify which user mode should be used.
```
def run_on_appengine(gdata_client, store_tokens=True,
                     single_user_mode=False)
```

For use case 1, store the auth token in the datastore associated with the current\_user. `(store_tokens=True, single_user_mode=False)` Users are required to sign in to the app for the token to be saved.

For use case 2, keep the token in memory.
`(store_tokens=False, single_user_mode=True)` or in the datastore `(store_tokens=True, single_user_mode=True)`

Use case 3 is a bit problematic because there needs to be some way to associtate the auth sub token with a user if the token is to be reused. Single use auth sub tokens should pose no problem.