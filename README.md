# Writing Integration Tests

## Learning Goals

- Understand the purpose of integration tests
- Understand the difference between integration tests and unit tests
- Learn how to write integration tests for React components

## Introduction

In an earlier lesson, you learned about four categories of testing that provide
a rough outline of the types of testing you might use to test your application:

- **Static**: tests for typos in code and errors that can be detected without
  even running the application. You don't actually _write_ static tests: there
  are tools, such as [ESLint][eslint] and [Typescript][typescript], that help
  perform _static analysis_ on your code to catch these types of errors.
- **Unit**: tests for individual "units" (which could be an individual function,
  class, or component) in isolation from the rest of the application.
- **Integration**: testing multiple related units of the application and the
  interactions between them.
- **End-to-end**: sometimes also called "functional testing", this type of test
  works by running your application in an environment as close as possible to
  how a user would interact with it, and verifying that the application
  functions as expected.

So far, the tests we've constructed using Jest have fallen into the **Unit**
category - testing of self-contained units of code that don't have any
dependencies outside themselves. But, of course, so far the applications we've
looked at have been fairly simple. As our applications get more complex, it will
become necessary to separate out the functionality into multiple components to
keep our code organized. When this happens, unit testing alone will no longer
meet our needs.

In this lesson, we'll talk about **Integration** testing, which looks at the
_relationship_ between different units of code and how they interact. We'll
learn how to write integration tests using Jest.

## What is Integration Testing

**Integration** tests are used to test multiple related units of code and how
they interact. They ensure that any modules of code that are related or depend
on each other in any way — i.e., there is at least one user workflow that
incorporates all of them — are interacting as expected. Although unit testing is
important, you can't be confident that your application _as a whole_ is behaving
as expected if that's all you do.

Kent C. Dodds, the creator of [Testing Library][testing-library], explains it
this way:

> While having some unit tests to verify these pieces work in isolation isn't a
> bad thing, _it doesn't do you any good if you don't **also** verify that they
> work together properly_.
>
> — [Kent C. Dodds: Write tests. Not too many. Mostly integration.][source]

Say, for example, that we have an application that includes a login flow with
separate components for the login screen, the user's home page, and the logout
screen. While we can test the elements or behavior of each of these components
individually using unit testing, what we really want to verify is that, when the
user logs in, they're taken to their home page and, when they log out, they're
taken to the logout page. Integration tests are what we use to accomplish this.
They will also help us ensure that, if we later refactor the code in one
component, the changes do not break its interactions with the others.

## Creating Integration Tests

Given the difference in the purpose of unit vs. integration tests, you might
expect there to be substantial differences are in how they are written. However,
because we're following Testing Library's [guiding
principles][guiding-principles] in testing our React apps, the good news is
there really _isn't_ much difference. In both cases, we write tests that verify
the presence of elements in the DOM, or the effects of user events by checking
the state of the DOM before and after the event occurs.

From the user's perspective, it doesn't matter whether all the functionality is
coded within one component or information is passed between components as props.
In our tests, therefore, the guiding principles dictate that rather than
focusing on implementation details — _how_ the code is implemented — we should
instead focus on the end result in the DOM.

One nice side effect of this is that including integration tests will likely
reduce the number of unit tests you need. Returning to our login flow example,
we might write integration tests to verify that the user is taken to their home
page when they log in. To do this, we would verify that one or more elements are
present in the DOM after the user clicks the "Log In" button. If this test
passes, then it is unnecessary to separately check for the presence of those
elements using unit testing.

Integration tests more closely resemble the way our software is used than unit
tests do — which is our goal, according to the guiding principles. As a result,
they give you more confidence that your app is working as you intend.

## My To-Dos App

Let's take a look at an example. We're going to write the tests and create the
code for a to-do app. In building out our app, we will go through the usual
process of tackling one piece at a time to:

1. Determine what we want to accomplish,
2. Write the test for that feature or functionality, and
3. Write the code to get the test passing

We want the app to include a list of the to-dos, along with a button that allows
users to add a new to-do.

For the initial state, therefore, we want to verify that:

- There are initially no to-dos listed
- The page contains an "Add To-do" text field

For our user events, we want to verify that:

- When a new to-do is submitted, the page updates to include it in the list of
  to-dos

To start, our to-do's will only have one property: the text of the to-do. In the
lab that follows this lesson, you will be adding the ability to delete to-dos
and to mark them as done. Given that our to-dos will be fairly complex by the
time we're done, we will plan for that by creating separate components for the
list (`TodoList`) and the individual to-dos (`Todo`). We'll render a `ul`
element in `TodoList` to contain the to-dos, and render each individual to-do
inside an `li` element.

To start, our files look like this:

```jsx
// App.js
import TodoList from "./TodoList";

function App() {
  return <TodoList />;
}

export default App;
```

```jsx
// TodoList.js
function TodoList() {
  return <div></div>;
}

export default TodoList;
```

```jsx
// Todo.js
function Todo() {
  return <li></li>;
}

export default Todo;
```

### Creating the List of To-Dos and the Input Field

Our first task is to create the list of to-dos and the input field that enables
a user to add a to-do. Let's start with the tests of the initial state of those
elements.

Recall that our plan is to render a `ul` element in `TodoList` to contain the
to-dos, and render each individual to-do inside an `li` element. Therefore, we
can use `getByRole("list")` to verify that there is initially a `ul` element on
the page, and `queryByRole("listitem")` to verify that there are initially no
`li` elements on the page:

```jsx
// App.test.js
import { render, screen } from "@testing-library/react";
import App from "../App";

describe("TodoList component initial status", () => {
  test("todo list is initially empty", () => {
    render(<App />);

    expect(screen.getByRole("list")).toBeInTheDocument();
    expect(screen.queryByRole("listitem")).not.toBeInTheDocument();
  });
});
```

As expected, our first test is failing:

![failing test](https://curriculum-content.s3.amazonaws.com/react-hooks-tdd/writing-integration-tests/test-fail.png)

Let's write the code to get it passing. We'll add a page heading in addition to
the `ul` element:

```jsx
// TodoList.js
function TodoList() {
  return (
    <div>
      <h1>My To-dos</h1>
      <ul></ul>
    </div>
  );
}

export default TodoList;
```

Our next test will check for the presence of the `input` element. We want the
element to contain "Add to-do" as the placeholder text, and to have a "Submit"
button. We can use [getByPlaceholderText][] and `getByRole`, respectively, to
check for these:

```jsx
test("the page includes an 'Add To-do' input element that has a submit button", () => {
  render(<App />);

  expect(screen.getByPlaceholderText(/add to-do/i)).toBeInTheDocument();
  expect(screen.getByRole("button", { name: /submit/i })).toBeInTheDocument();
});
```

Then add the code to get this test passing:

```jsx
//TodoList.js
function TodoList() {
  return (
    <div>
      <h1>My To-dos</h1>
      <ul></ul>
      <form>
        <input type="text" placeholder="Add to-do" />
        <input type="submit" />
      </form>
    </div>
  );
}

export default TodoList;
```

We've now written the tests for the initial state of `TodoList` and the code to
get them passing. Next, we'll write our tests to verify that, when a new to-do
is submitted, the page updates to include it in the list of to-dos. Because the
individual to-dos will be rendered by the `Todo` component, this will be our
first integration test.

Since we'll be simulating user events for this test, we'll first need to import
`userEvent` along with our other imports:

```jsx
// App.test.js
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import App from "../App";
```

Then we'll add our next test:

```jsx
describe("TodoList component user events", () => {
  test("when a new to-do is submitted it appears in the list of to-dos", () => {
    render(<App />);

    const inputField = screen.getByPlaceholderText(/add to-do/i);
    const submitButton = screen.getByRole("button", { name: /submit/i });

    userEvent.type(inputField, "take out the trash");
    userEvent.click(submitButton);
    userEvent.type(inputField, "walk the dog");
    userEvent.click(submitButton);

    expect(screen.getAllByRole("listitem").length).toBe(2);
    expect(screen.getByText("take out the trash")).toBeInTheDocument();
    expect(screen.getByText("walk the dog")).toBeInTheDocument();
  });
});
```

In this code, we first render the `App` component. Next, we access the input
element and the button and save them into variables. We then create two new
to-dos by simulating the user events of typing a to-do into the input and
clicking the Submit button. Finally, we verify that those two to-dos are
displayed on the page.

To get this one passing, we need to do a number of things. We'll start with the
changes to `TodoList`.

**1.** Import `useState` and the `Todo` component:

```jsx
import { useState } from "react";
import Todo from "./Todo";
```

**2.** Create state variables to hold the to-do list and individual to-dos:

```jsx
const [todos, setTodos] = useState([]);
const [newTodo, setNewTodo] = useState("");
```

**3.** Create an `enterTodo` function to update an individual to-do value in
state as the user types it in, and an `addTodo` function to add the new to-do to
the list when the user clicks the "Submit" button:

```jsx
const enterTodo = (e) => {
  setNewTodo(e.target.value);
};

const addTodo = (e) => {
  e.preventDefault();
  setTodos([...todos, { text: newTodo }]);
  setNewTodo("");
};
```

**4.** Inside the `ul` element, add the code to render the `Todo` component for
each to-do in the list:

```jsx
<ul>
  {todos.map((todo) => (
    <Todo key={todo.text} todo={todo} />
  ))}
</ul>
```

**5.** Finally, add an `onSubmit` property to the form, and `onChange` and
`value` properties to the `input` element to make it a controlled input:

```jsx
<form onSubmit={addTodo}>
  <input
    type="text"
    placeholder="Add to-do"
    onChange={enterTodo}
    value={newTodo}
  />
  <input type="submit" />
</form>
```

`TodoList.js` now looks like this:

```jsx
// TodoList.js
import { useState } from "react";
import Todo from "./Todo";

function TodoList() {
  const [todos, setTodos] = useState([]);
  const [newTodo, setNewTodo] = useState("");

  const enterTodo = (e) => {
    setNewTodo(e.target.value);
  };

  const addTodo = (e) => {
    e.preventDefault();
    setTodos([...todos, { text: newTodo }]);
    setNewTodo("");
  };

  return (
    <div>
      <h1>My To-dos</h1>
      <ul>
        {todos.map((todo) => (
          <Todo key={todo.text} todo={todo} />
        ))}
      </ul>
      <form onSubmit={addTodo}>
        <input
          type="text"
          placeholder="Add to-do"
          onChange={enterTodo}
          value={newTodo}
        />
        <input type="submit" />
      </form>
    </div>
  );
}

export default TodoList;
```

Then, inside `Todo.js`, we'll add the code to render individual to-dos:

```jsx
// Todo.js
function Todo({ todo }) {
  return <li>{todo.text}</li>;
}

export default Todo;
```

With these changes, all three of our tests are passing!

## Conclusion

In this lesson, you learned about the purpose of integration tests and how they
differ from unit tests. In the next lesson, you'll get some hands-on practice
writing integration tests using Jest and React Testing Library.

## Resources

- [Testing Library: Queries][queries]
- [Jest DOM - Custom Matchers][jest-dom]
- [Testing Library: user-event][user-event]
- [MDN: ARIA Role Reference][mdn-aria-roles]

[queries]: https://testing-library.com/docs/queries/about
[jest-dom]: https://github.com/testing-library/jest-dom
[user-event]: https://testing-library.com/docs/ecosystem-user-event/
[mdn-aria-roles]:
  https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Techniques
[eslint]: https://eslint.org/
[typescript]: https://www.typescriptlang.org/
[guiding-principles]: https://testing-library.com/docs/guiding-principles
[source]: https://kentcdodds.com/blog/write-tests
[testing-library]: https://testing-library.com/
[getbyplaceholdertext]:
  https://testing-library.com/docs/queries/byplaceholdertext/
