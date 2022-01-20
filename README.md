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

> while having some unit tests to verify these pieces work in isolation isn't a
> bad thing, _it doesn't do you any good if you don't **also** verify that they
> work together properly_. ([source])

Say, for example, that we have an application that includes a login flow with
separate components for the login screen, the user's home page, and the logout
screen. While we can test the elements or behavior of each of these components
individually using unit testing, what we really want to verify is that, when the
user logs in, they're taken to their home page and, when they log out, they're
taken to the logout page. Integration tests are what we use to accomplish this.
They will also help us ensure that, if we later refactor the code in one
component, the changes do not break their interaction with the others.

## Creating Integration Tests

Given the difference in the purpose of unit vs. integration tests, you might
expect there to be substantial differences are in how they are written. However,
because we're following Testing Library's [guidingprinciples][guiding-principles]
in testing our React apps, the good news is there really _isn't_ much
difference. In both cases, we write tests that verify the presence of elements
in the DOM, or the effects of user events by checking the state of the DOM
before and after the event occurs.

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
code for a to-do app. In building out our app, we will go through the
usual process of tackling one piece at a time to:

1. determine what we want to accomplish,
2. write the test for that feature or functionality, and
3. write the code to get the test passing

We want the app to include a list of the to-dos, along with a button that allows
users to add a new to-do.

For the initial state, therefore, we want to verify that:

- there are initially no to-dos listed
- the page contains an "Add To-do" text field

For our user events, we want to verify that:

- When the new to-do is submitted, the page updates to include it in the list of
  to-dos

To start, our to-do's will only have one property: the text of the to-do. After
we get the base functionality working, we will add on the ability for users to
delete to-dos. Finally, you will tackle one more functionality: marking to-dos
as done. Given that our to-dos will be fairly complex by the time we're done, we
will plan for that by creating separate components for the list (`TodoList`) and
the individual to-dos (`Todo`).

## Conclusion

A short one or two paragraph summary of the contents of the lessons, recapping
the learning goals.

## Resources

- [Resource Link 1](example.com)
- [Resource Link 2](example.com)

[guiding-principles]: https://testing-library.com/docs/guiding-principles
[source]: https://kentcdodds.com/blog/write-tests
[testing-library]: https://testing-library.com/
