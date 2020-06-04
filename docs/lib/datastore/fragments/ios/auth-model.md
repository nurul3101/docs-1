## Updated schema

The following schema is an example of a model named `Todo` that is readable and writable by the creator, and read only to everyone else.

```graphql
type Todo
    @model
    @auth(rules: [
        { allow: owner, ownerField: "owner", operations: [create, update, delete] },
    ]) {
    id: ID!
    content: String!
    owner: String
}
```

The schema will allow users to query, list, and subscribe to model creation, updates, deletions by other users. A user can only update and delete their own model, that is a model can not be updated or deleted by another user.

The `@auth` directive when used with Amplify DataStore supports the following fields:

  - `allow` with "owner" and "groups"
  - `ownerField`
  - `groups`
  - `operations`

## Configure API with Amazon Cognito user pool

While using the `@auth` directive in your schema to limit read/write access, we require authentication to be integrated into your app in order to differentiate who the owner is. To set this up issue the command:

```console
amplify update api
```

then follow the prompts:

```console
? Please select from one of the below mentioned services: 
  `GraphQL`
? Select from the options below 
  `Update auth settings`
? Choose the default authorization type for the API 
  `Amazon Cognito User Pool`
```

Once finished, run `amplify push` to provision your services in the backend.

## Configuration

Upon successfully running `amplify push` per the last step, `amplifyconfiguration.json` will have a section under "awsAPIPlugins" and "awsCoognitoAuthPlugin" with the corresponding API and Auth configurations. Your `amplifyconfiguration.json` may look like this:

```json
{
    "api": {
        "plugins": {
            "awsAPIPlugin": {
                // ...
            }
        }
    },
    "auth": {
        "plugins": {
            "awsCognitoAuthPlugin": {
                // ...
            }
        }
    }
```

Add "AWSCognitoAuthPlugin" and "AWSAPIPlugin" to Amplify before configuring

```swift
Amplify.add(plugin: AWSCognitoAuthPlugin())
Amplify.add(plugin: AWSAPIPlugin())
Amplify.configure()
```

## Authenticate before Save

In order to save models, make sure the user is signed in using `Amplify.Auth.signIn`, then you can create an instance of the model as normal and call `DataStore.save`. See [Auth.SignIn](~/lib/auth/signin.md) for more details.

If the user has terminated the app or toggled between background and foreground of the app, you can check if the user is already signed in using `Amplify.Auth.fetchAuthSession`. See [Auth.fetchAuthSession](~/lib/auth/getting-started.md#check-the-current-auth-session) for more details.

If the user becomes unauthenticated, make sure the app is able to handle this scenario by hiding actions which trigger DataStore operations provided to authenticated users. To handle this scenario, listen to Hub events for Auth. See [Auth Events](~/lib/auth/auth-events.md) for more details. If the user is unauthenticated and the app performs a DataStore operation, it will fail due to an authentication error. See [conflict resolution section](~/lib/datastore/conflict.md) to learn more about how to handle these errors in the DataStoreConfiguration's `errorhandler`. 

## Model owner

To check who the owner of the model is, call `DataStore.query` and check the user's username against the owner field. 
```swift
if let user = Amplify.Auth.getCurrentUser() {
    if user.username == todo.owner {
        print("This todo was created by the current user")
    }
}
```

If you attempt to call `DataStore.save` or `DataStore.delete` on a model that is not owned by the user, then the conflict handler will be called. See [conflict resolution section](~/lib/datastore/conflict.md) to learn more about how to handle conflicts in the DataStoreConfiguration's `conflict handler`

