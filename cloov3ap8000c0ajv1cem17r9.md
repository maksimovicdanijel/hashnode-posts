---
title: "How to test React Router components?"
datePublished: Tue Nov 07 2023 21:47:12 GMT+0000 (Coordinated Universal Time)
cuid: cloov3ap8000c0ajv1cem17r9
slug: how-to-test-react-router-components
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/C7B-ExXpOIE/upload/4c09ec3a51bbe0ec9c03ebfdd96d97e8.jpeg
tags: react-router, reactjs, testing, best-practices

---

Every React app I had worked with used [react-router](https://reactrouter.com/en/main). There isn't much sense in having an application without it. In this article, we will be looking at how to test the component that is doing the navigation and why mocking React Router's APIs isn't the optimal solution.

### Component to test

Let's set an example of a component that navigates to another page on user action:

```typescript
import { useNavigate } from 'react-router-dom'

export const BackButton: React.FC<{ to?: string }> = ({ to }) => {
  const navigate = useNavigate()

  return (
    <button onClick={navigate(to ?? -1)}>
      Go back
    </button>
  )
}
```

The component in our example is navigating the user back to the previous page or to the path provided as `to` prop. It's the usual stuff you see in almost every app.

### The mocking approach

How do we test this component to ensure it navigates to the correct page upon clicking the button? Let's take a look at the approach with mocking:

```typescript
import { useNavigate } from 'react-router-dom'
import { render, screen } from '@testing-library/react'
import { userEvent } from '@testing-library/user-event'

import { BackButton } from './BackButton'

jest.mock('react-router-dom', () => ({
  useNavigate: jest.fn(),
}));

const useNavigateMock = jest.mocked(useNavigate)

it('navigates back to the previous page if `to` prop is not passed', async () => {
  const navigateMock = jest.fn()

  useNavigateMock.mockReturnValue(navigateMock)

  render(<BackButton />)

  await userEvent.click(screen.getByRole('button', { text: /go back/i }))

  expect(navigateMock).toHaveBeenCalledTimes(1)
  expect(navigateMock).toHaveBeenCalledWith(-1)
})

it('navigates to the path provided as a prop', async () => {
  const navigateMock = jest.fn()

  useNavigateMock.mockReturnValue(navigateMock)

  render(<BackButton to='/about' />)

  await userEvent.click(screen.getByRole('button', { text: /go back/i }))

  expect(navigateMock).toHaveBeenCalledTimes(1)
  expect(navigateMock).toHaveBeenCalledWith('/about')
});
```

We are testing both use cases of the component, with and without `to` prop, by clicking on the button and asserting that the `navigate` function returned by `useNavigate` hook is called with the adequate arguments.

Let's imagine that we don't like how we are navigating in our `BackButton` component and would like to refactor it to navigate in a declarative way by using the [`Navigate`](https://reactrouter.com/en/main/components/navigate) component from the `react-router-dom`.

```typescript
import { Navigate } from 'react-router-dom'
import { useState } from 'react'

export const BackButton: React.FC<{ to?: string }> = ({ to }) => {
  const [shouldNavigate, setShouldNavigate] = useState(false)  

  if (shouldNavigate) {
    return <Navigate to={to ?? -1} />
  } 
 
  return (
    <button onClick={() => setShouldNavigate(true)}>
      Go back
    </button>
  )
}
```

Notice our functionality didn't change. We are still navigating back to the previous location if `to` prop isn't passed and are navigating to `to` path if provided. Will our test still work?

Unfortunately, no 🫠 Since we are mocking `react-router-dom` the module in our tests, it will scream at us that there is no `Navigation` named export from the module and it will fail miserably. Of course, we could refactor the test as well and provide a mock `Navigate` component and thus fix the test, but the test should survive code refactoring if the behavior from a user perspective didn't change.

> Tests should give developers confidence to freely refactor code knowing that they will still pass. 🎉

The reason our test failed is that we have **intertwined an implementation detai**l of how we navigate from page to page with our test. In order to write a better, more stable test we should think of what actually happens when a user clicks on the button.

When a user clicks on the back button a user is taken to the previous page, meaning the URL changes. How can we test this side-effect of the user's action? 🤔

### Writing a test without mocking

A while back while I was browsing `react-router-dom` docs I stumbled upon the `MemoryRouter` component. In the docs it says it stores its state internally in memory, making it perfect for environments where there is no `history` object, like tests. That sounds interesting, right?

Let's see how we can use the `MemoryRouter` to refactor our `BackButton` component test and make it more resilient:

```typescript
import { MemoryRouter, Routes, Route } from 'react-router-dom'
import { render, screen, waitFor } from '@testing-library/react'
import { userEvent } from '@testing-library/user-event'

import { BackButton } from './BackButton'

it('navigates back to the previous page if `to` prop is not passed', async () => {
  render(
    <MemoryRouter initialEntries={['/first-page', '/current-page']}>
      <BackButton />
      <Routes>
        <Route path='/first-page' element={<p>first page</p>} />
        <Route path='/current-page' element={<p>current page</p>} />
      </Routes>
    </MemoryRouter>,
  )

  expect(screen.getByText(/current page/i)).toBeInDocument()

  await userEvent.click(screen.getByRole('button', { name: /go back/i }))

  await waitFor(() => {
    expect(screen.getByText(/first page/i)).toBeInDocument()
    expect(screen.queryByText(/current page/i).not.toBeInDocument()
  })
})

it('navigates to the path provided as a prop', async () => {
  render(
    <MemoryRouter initialEntries={['/']}>
      <BackButton to='/about' />
      <Routes>
        <Route path='/' element={<p>home</p>} />
        <Route path='/about' element={<p>about page</p>} />
      </Routes>
    </MemoryRouter>,
  )

  expect(screen.queryByText(/about/i)).not.toBeInDocument()

  await userEvent.click(screen.getByRole('button', { name: /go back/i }))

  await waitFor(() => {
    expect(screen.getByText(/about/i)).toBeInDocument()
  })
})
```

We need to wrap our `BackButton` component with a `MemoryRouter` so it can work with the `react-router` APIs and we need to set a realistic test environment, providing an array of paths in the history stack.

When a user clicks on the button we check if the correct component is being rendered based on the routing setup and the expected behavior. **There is no mocking of the** **React Router module.**

The test written in this way resembles the user behavior, making our test more resilient to code refactors and giving us more confidence in our test suite.