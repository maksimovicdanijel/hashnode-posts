---
title: "TypeScript utility types I use the most"
datePublished: Tue May 23 2023 20:31:49 GMT+0000 (Coordinated Universal Time)
cuid: cli0qf8yg000b09lid8muht7w
slug: typescript-utility-types
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1684873418555/33553783-9d3d-4290-bd61-2d31af17db3c.jpeg
tags: typescript

---

TypeScript comes with a lot of built-in types that can be used for extracting, converting, or manipulating types in many ways. We are going to cover the ones I use the most and we will go through them in a Q&A fashion.

### How do I pick a couple of properties of a type?

Let's say we have a type called `Post` that looks like this:

```typescript
type Post = {
  title: string
  content: string
  tags: string[]
  slug: string
}
```

and we want to create a new post type that only has a title and content. We could create it from scratch and duplicate `Post`'s properties, or we can pick the properties we want from the already created type. We don't want to repeat ourselves so we can do it like so:

```typescript
type DraftPost = Pick<Post, 'title' | 'description'>
```

`Pick` utility types creates a new type from a given type by using only properties provided as a union.

### How can I remove a property from a type?

In our previous example, we have chosen the properties we want to have in our new type. Let's now remove properties we don't want.

```typescript
type DraftPost = Omit<Post, 'slug' | 'author'>
```

`Pick` and `Omit` are two sides of the same coin, they do opposite things that can achieve the same result.

### How can I pick a value from an enum or a union type?

Let's say we have an enum of post categories:

```typescript
enum PostCategories {
  React
  Agile
  Fastify
  GraphQL
  Postgres
}
```

and we would like to create a union of post categories that are only back-end1 related. We could try using `Pick` type that we saw earlier but it won't help much as `Pick` type works with object types. Instead, we could use `Extract` utility type:

```typescript
type BackendCategories = Extract<PostCategories, 'Fastify' | 'GraphQL' | 'Postgres'>
```

Newly-created `BackendCategories` type would be a union `PostCategory.Fastify | PostCategory.GraphQL | PostCategory.Postgres`. `Extract` **doesn't create a new union**, it transforms it into a union.

### How can I remove a property from a union or an enum?

In the same fashion as `Extract`, there is a `Extract` utility type that does a similar thing as the `Omit` type, but for union or enum types.

Let's say we want to remove `Agile` from post categories:

```typescript
type TechCategories = Exclude<PostCategories, 'Agile'>
```

Newly-created `TechCategories` type would be a union of all enum values except `Agile`.

### How do I make a type non-nullable?

Let's say we have a type that is a union of an object type and null, which is a common scenario when typing an API response:

```typescript
type PostResponse = { data: Post | null }
```

and let's say we have a function that is transforming API response in some manner:

```typescript
function transformResponse(apiResponse: PostResponse) {
  // implementation here
}
```

When trying to access the `apiResponse.data` properties TypeScript will error out as we are trying to access a property on a potentially nullable value. If we don't want to have the null check in our transform function we could utilize the `NonNullable` utility type:

```typescript
function transformResponse(apiResponse: NonNullable<PostResponse>) {
  // implementation here
}
```

This will make sure `apiResponse.data` has a `Post` type as `NonNullable` utility will remove `null` and `undefined` from the union.

### How can I make all properties of a type optional?

Sometimes we want to use the existing type but only provide a subset of its properties. For example, we might have a factory function that generates objects of a certain type but would like to provide a way for a user to override generated values:

```typescript
function buildPost(overrides?: Partial<Post>): Post {
  return {
    id: randomUUID(),
    title: 'Just some title',
    content: 'Description of a post',
    tags: ['tag1'],
    slug: 'just-some-title',
    ...overrides
  }
}

buildPost({ title: 'My custom title' })
```

In our builder function, we are setting the `overrides` type to `Partial` of a post type, which makes all properties of a `Post` type optional, which gives us an option to only pass a subset of `Post` properties.

### How can I make all properties of a type required?

Let's say we have a form to submit a new post, and we have a type to define the form's input:

```typescript
type PostInput = {
  title?: string
  description?: string
  tags?: string[]
}
```

but we need a type when all fields are required, like in the submit handler function:

```typescript
function submitPost(postInput: Required<PostInput>) {
  // submit a post
}
```

`Required` utility type is quite useful when we don't want to repeat ourselves just to make all properties of a type required.

### How can I extract a return type of a function?

This one is quite useful when you want to get a type of what a function is returning but you don't have access to the type itself, e.g. when a third-party library exposes a typed function but it doesn't export the returned type. For example:

```typescript
// third-party library
type Slug = // complex slug type that isn't exported

export function slugifyString(str: string): Slug

// your app
type Slug = ReturnType<typeof slugifyString>

type Post = {
  id: string
  title: string
  slug: Slug
}
```

In our example, we are using a function that returns some complicated type (`Slug`) but it isn't exporting it for our use. We want to reuse that value in our application but since it's not exported we should get it in some other way. `ReturnType` utility type extracts the type a function is returning so we can use it in our own application.

### How can get a type wrapped in a Promise?

Let's say our previous example returns a `Promise` instead of an object type:

```typescript
// third-party library
type Slug = // complex slug type that isn't exported

export function slugifyString(str: string): Promise<Slug>

type Slug = ReturnType<typeof slugifyString> // it's a Promise now

type Post = {
  id: string
  title: string
  slug: Slug // won't work anymore
}
```

Our type extraction would now return a `Promise` type as that's what the function is returning. We need to get to the type wrapped in a promise and we can do it like so:

```typescript
type Slug = Awaited<ReturnType<typeof slugifyString>>
```

This might look like a bit too much generics nesting and if it is too much for you, you can split it like so:

```typescript
type SlugifyStringPromise = ReturnType<typeof slugifyString>
type Slug = Awaited<SlugifyStringPromise>
```

### How can I type an object literal?

Object literals are one of the most used data structures in JavaScript and rightfully so. But how do we type them?

```typescript
const postsByCategories = {
  [PostCategory.React]: [post1, post2],
  [PostCategory.GraphQL]: [post3, post4]
  // ...
}
```

Let's say we would like to allow only post categories as keys and only an array of posts as values:

```typescript
const postsByCategories: Record<PostCategory, Post[]> = {
  [PostCategory.React]: [post1, post2],
  [PostCategory.GraphQL]: [post3, post4]
  // ...
}
```

The only keys TypeScript would allow are values from `PostCategory` enum and only an array of `Post`s can be set as a value. The `Record` utility type is really useful when we have a map of set keys and values.

---

These are the utility types I use the most, although this isn't the complete list of available utility types. Had over to the [TypeScript documentation](https://www.typescriptlang.org/docs/handbook/utility-types.html#handbook-content) to get familiar with even more utility types.

Thanks for reading!