# My thoughts on Hasura

I've been using Hasura for the more significant part of the year, working every day with it. It's marketed as a BaaS (backend as a service), which means that it exposes a GraphQL API for you from some data source (DB, Rest API, other GraphQL). It's open source but it also has its own cloud offering. I'll discuss the open-source one.

Hasura makes it really easy to start developing and prototyping quickly. It supports migrations, auth and authz, permissions, relationships between entities, custom actions, remote schemas, and events. It also supports transactions when multiple mutations are sent in the same operation. This all sounds great, right?

It's quite good for small projects, that don't have that much business logic. If CRUD is all your app is doing and you only have a single client, you will probably be fine with using Hasura. But, if you are working in a complex domain and/or you have multiple clients I don't think Hasura is a good choice.

#### The client knows too much

Since Hasura is exposing all of the CRUD operations (or the subset, depending on the permissions), the client would try to use it as much as possible, right? But, when there is a more complicated feature like some data aggregation or complex filtering, that can make the client code really complicated. The client could reach for a custom action to extract that logic and simplify the client and that's when the first deal-breaker for me enters the scene.

> Generated DB types can't be used in custom actions, except in a relationship.

☝️ means that you have to declare your own types used in your action, which inevitably leads to the duplication of types. And if you are using TypeScript (and you probably should if you are using GraphQL), generating Typescript types from such a bloated schema is a real problem. Another downside of actions, is that you can't use `union` and `interface` when designing your schema 👎

#### Permissions

Permissions in Hasura are role-based. They are set up in Hasura's UI, and each role can restrict the access to table's rows and columns. They will get you pretty far until the same role needs to access different rows in different scenarios. 

For example, permissions for the role `author` are set up so the author can see only their own articles. But then the feature comes up that the author should see their own articles on one page and some unrelated articles on some other page. We could reach for loosening up the permissions for that role and add some client-side filtering, but that is a security issue right there. The other potential solution is to extract the logic for the new feature into custom action, but then we hit the limitation discussed in the previous point. We could try to mitigate it using relationships in actions but permissions are run on them as well.

In the end, permissions can get quite complex, resulting in using multiple relationships in order to resolve whether a row should be visible to the role or not.

Regarding the points above, imagine if you had a couple of clients instead of just one. It would be a pain.

The last concern I have with Hasura (and potentially other BaaS tools) is that it's a black box. As a developer, I don't know what the query that Hasura is generating behind scenes looks like, and I don't know how to optimize it (probably there is no way for it).

#### The good parts

There are good parts to using Hasura as well. 

Remote schemas are awesome. You can connect to some other GraphQL API, and have a unified Graph for your app. You can even add permissions on the parts of that remote schema.

Events and scheduled events can simplify the infrastructure of your app. You don't need to manage cron jobs or queues. But again, that will work on some smaller app.

-----

That is my experience using Hasura, yours might be different and that's OK. 