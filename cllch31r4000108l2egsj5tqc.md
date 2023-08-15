---
title: "You should repeat yourself when writing tests"
datePublished: Tue Aug 15 2023 15:42:44 GMT+0000 (Coordinated Universal Time)
cuid: cllch31r4000108l2egsj5tqc
slug: you-should-repeat-yourself-when-writing-tests
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/jVPJW4rvBI4/upload/fc379b6ecc7aaa91c5c12731e025ac84.jpeg
tags: javascript, testing, typescript, jest

---

How would you react if I would say you should duplicate code when writing tests? You would probably think I don't know what I'm talking about and that I will be breaking one of the most followed principles in software development, the [DRY principle](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself). Let me convince you that it's perfectly fine to do exactly that and it's the thing you should be doing more when writing tests.

##### **The case for repeating code**

Let's say we need to test a function called `postStatus` that accepts a post and returns the post status. Here's an implementation we need to test:

```typescript
function postStatus(post: Post): PostStatusEnum {
  if (post.isDraft) {
    return PostStatusEnum.Draft
  }

  if (post.scheduledDate > Date.now()) {
     return PostStatusEnum.Scheduled
  }

  if (!post.authorId) {
    return PostStatusEnum.MissingAuthor
  }

  return PostStatusEnum.Published
}
```

And here's the test I see very often written by developers:

```typescript
describe('postStatus()', () => {
   let postData: Post

   beforeEach(() => {
     postData = {
       isDraft: false,
       scheduledDate: null,
       authorId: null
     }
   })

  it(`returns ${PostStatusEnum.MissingAuthor} if post isn't scheduled, isn't draft but doesn't have an author assigned`, () => {
     expect(postStatus(postData)).toBe(PostStatusEnum.MissingAuthor)
  })

  it(`returns ${PostStatusEnum.Draft} if post is marked as draft`, () => {
    postData.isDraft = true

    expect(postStatus(postData)).toBe(PostStatusEnum.Draft)
  })

   // other test cases
})
```

On the first look, this test looks pretty good, right? What happens when the test breaks and you need to debug it? You skim the failing test case and wonder where does the `postData` comes from, where it's defined, and how it's connected to the failing test case? It's especially true for the first test case as there is only one line for the whole test suite and you have no idea why `postData` makes the function return `PostStatusEnum.Draft` status.

It might be obvious in this example but let's imagine a test case that has 200 lines of code and a dozen test cases and every test depends on multiple variables defined out of the test's scope. It would be very difficult to connect the dots and keep everything in your mental memory while debugging that kind of test.

This is how I would write that same test:

```typescript
it(`returns ${PostStatusEnum.MissingAuthor} if post isn't scheduled, isn't draft but doesn't have an author assigned`, () => {
  const postData: Post = {
    isDraft: false,
    scheduledDate: null,
    authorId: null
  }

  expect(postStatus(postData)).toBe(PostStatusEnum.MissingAuthor)
})

it(`returns ${PostStatusEnum.Draft} if post is marked as draft`, () => {
  const postData: Post = {
    isDraft: true,
    scheduledDate: null,
    authorId: null
  }

  expect(postStatus(postData)).toBe(PostStatusEnum.Draft)
})

// other test cases
```

When a test fails in this scenario, everything a developer needs to understand why the test failed is contained within the test. The test is not relying on variables defined in the enclosing scope and it is easy to follow.

I'm repeating myself in every test case, and that is not what we should be doing, right? The testing code is a bit different than the production code. We want to be spending less time grasping the intent of the testing code, and we want to minimize the cognitive load of testing code so we can focus more on production code. Which test example is more difficult to understand, the one that's **DRY** or the one that **repeats itself**? In my experience, it's the latter that it's way easier to understand and maintain.

#### But I don't want to repeat myself?!

Worry not, as we are going to clear the last test example a bit while and still have a minimal cognitive load. Let's take a look at the alternative approach:

```typescript
it(`returns ${PostStatusEnum.MissingAuthor} if post isn't scheduled, isn't draft but doesn't have an author assigned`, () => {
  const postData = buildPost({
    scheduledDate: null,
    isDraft: false,
    authorId: null
  })

  expect(postStatus(postData)).toBe(PostStatusEnum.MissingAuthor)
})

it(`returns ${PostStatusEnum.Draft} if post is marked as draft`, () => {
  const postData: Post = buildPost({ isDraft: false })

  expect(postStatus(postData)).toBe(PostStatusEnum.Draft)
})

function buildPost(overrides?: Partial<Post>): Post {
  return {
    isDraft: false,
    scheduledDate: null,
    authorId: null
  }
}

// other test cases
```

Now with this approach, we are not repeating ourselves **and** we are colocating our test data with our tests.

The bonus is that the test data is **laser-focused** on the scenario we are testing. In the draft status test case, we are only passing a single `Post` attribute as the test case concern is only that, to test the draft status. In this way, it's much easier and faster to understand what the test is actually testing, and what makes the difference in the function's output.

This approach can be used in every test scenario, be it a simple function test as we had in our example, a React component, a Fastify endpoint, or a full-blown E2E using Playwright or Cypress. Hope this helps a bit in writing more sane and maintainable tests 👋