---
title: Models with authorization
description: Learn more about how DataStore handles Models with finer grain authorization access checks.
---

DataStore with [sync to cloud enabled](~/lib/datastore/sync.md) has the capability to operate on models with finer grain authorization checks at the API. In GraphQL, this is done with the `@auth` directive defined in the [GraphQL Transformer documentation](~/cli/graphql-transformer/directives.md#auth).

<amplify-callout>

**Note:** this API is under development and it is not generally available yet.

</amplify-callout>

<inline-fragment platform="ios" src="~/lib/datastore/fragments/ios/auth-model.md"></inline-fragment>
