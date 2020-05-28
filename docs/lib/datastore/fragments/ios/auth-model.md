<amplify-callout>
When using the `@auth` directive with DataStore, currently only following fields are supported:

1. `allow` with "owner" and "groups"
2. `ownerField`
3. `groups`
4. `operations`

</amplify-callout>

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

The API will allow users to query, list, and subscribe to model creation, updates, deletions by other users. A user can only update and delete their own model, that is a model can not be updated or deleted by another user.

## API with Cognito User Pool auth

With the schema, make sure you have selected Cognito User Pool as the auth

```console
? Please select from one of the below mentioned services: 
  `GraphQL`
? Select from the options below 
  `Update auth settings`
? Choose the default authorization type for the API 
  `Amazon Cognito User Pool`
```

Once you have updated the schema and selected Cognito User Pool as the authorization type, run `amplify push` 

## Configuration

`amplifyconfiguration.json` will have section under "awsAPIPlugins" and "awsCoognitoAuthPlugin" with the API and Auth configurations
```json
{
    "api": {
        "plugins": {
            "awsAPIPlugin": { 
                // ...
    "auth": {
        "plugins": {
            "awsCognitoAuthPlugin": {
                // ...
```

Add "AWSCognitoAuthPlugin" and "AWSAPIPlugin" to Amplify before configuring

```swift
Amplify.add(plugin: AWSCognitoAuthPlugin())
Amplify.add(plugin: AWSAPIPlugin())
```

## Saving

In order to save models, make sure the user is signed in using `Amplify.Auth.signIn`, then you can create an instance of the model as normal and call `DataStore.save`.

If the user's session has expired or the user has logged out but the save is still executed, the model instance will be persisted to local storage. The model instance however will fail to be saved to API and the error handler will be called. See [conflict resolution section](~/lib/datastore/conflict.md) to learn more about how to handle errors in the DataStoreConfiguration's `errorhandler`. To prevent unnecessary saves from occurring, you can check if the user is signed in using `Amplify.Auth.fetchAuthSession`:

```swift
Amplify.Auth.fetchAuthSession { (result) in
    switch result {
    case .success(let authSession):
        if authSession.isSignedIn {
            let todo = Todo(content: "my todo content")
            Amplify.DataStore.save(todo) {
                switch $0 {
                case .success:
                    print("Added post")
                case .failure(let error):
                    print("Error adding post - \(error.localizedDescription)")
                }
            }
        }
    case .failure(let authError):
        print("Failed to fetch auth session", authError)
    }
}
```

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
