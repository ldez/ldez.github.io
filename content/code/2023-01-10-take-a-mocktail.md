---
title: "Take a Mocktail"
date: 2023-01-10T01:18:13+01:00
categories: ['lib']
tags: ['tool', 'mock', 'go', 'golang', 'testify']
slug: take-a-mocktail
---

![Mocktail](/images/mocktail-social-github.png)

Mocktail: Mock Generator for Strongly-Typed Mocks.

<!--more-->

In software development, mocks are important testing pieces used to produce isolation for a targeted code.
When you need to test a piece of code, in some cases, you want to isolate that tested piece
 — for example, you want to test a service that uses a database, but you don't want to call the real database.

There are several approaches to answering this problem, but in this article, I’ll only focus on the use of mocks.
Mocks are simulated objects that mimic a behavior.
The use of mocks is common and useful, but mocks have certain usage and maintainability constraints.
Mocks can be simple or complex depending on the size of the tested piece and the complexity of the expected behavior.


## Creating mocks manually or with a framework

Mocks can be used to serve several needs that depend on how the piece that you are trying to replace is used inside the tested code.
Maybe the object is just needed to compile, maybe the object needs to return always the same thing,
or maybe the object needs to be "smart" enough to be able to return the right element based on some parameters.

The first approach when you need a mock is to create it manually.

```go
package foo

import "fmt"

type API interface {
   Get(userID string) *User
   Save(user *User) error
}

type User struct {
   ID   	string
   username string
   Domain   string
}

type Service struct {
   api API
}

func (s Service) GetDomain(userID string) (string, error) {
   user := s.api.Get(userID)
   if user == nil {
  	return "", fmt.Errorf("user %q not found", userID)
   }

   if user.Domain != "" {
  	return user.Domain, nil
   }

   user.Domain = user.username + ".example.com"

   err := s.api.Save(user)
   if err != nil {
  	return "", err
   }

   return user.Domain, nil
}
```

```go
package foo

import (
   "testing"

   "github.com/stretchr/testify/assert"
   "github.com/stretchr/testify/require"
)

type apiMock struct {
   getResponse  *User
   getCallCount int

   saveResponse  error
   saveCallCount int
}

func (m *apiMock) Get(_ string) *User {
   m.getCallCount++

   return m.getResponse
}

func (m *apiMock) Save(_ *User) error {
   m.saveCallCount++
   return m.saveResponse
}

func TestSimple(t *testing.T) {
   user := &User{
  	ID:   	"8411fd4127",
  	username: "foo",
  	Domain:   "",
   }

   mck := &apiMock{
  	getResponse:  user,
  	saveResponse: nil,
   }

   service := Service{api: mck}

   data, err := service.GetDomain("8411fd4127")
   require.NoError(t, err)

   assert.Equal(t, "foo.example.com", data)
   assert.Equal(t, 1, mck.getCallCount)
   assert.Equal(t, 1, mck.saveCallCount)
}
```

The handwritten mocks can be enough in a lot of use cases, but sometimes you can have large objects or complex expected behavior.
In those cases, the approach will be to use a framework that helps with that level of complexity.

At Traefik Labs, we use Testify for test assertions.
[Testify](https://github.com/stretchr/testify) is the most popular assertions library in Go,
it helps to improve the test readability and reduce the size of the test plumbing, and it also contains a mock system.

```go
package foo

import (
   "testing"

   "github.com/stretchr/testify/assert"
   "github.com/stretchr/testify/mock"
   "github.com/stretchr/testify/require"
)

type apiMock struct{ mock.Mock }

func newAPIMock(tb testing.TB) *apiMock {
   tb.Helper()

   m := &apiMock{}
   m.Mock.Test(tb)

   tb.Cleanup(func() { m.AssertExpectations(tb) })

   return m
}

func (s *apiMock) Get(userID string) *User {
   ret := s.Called(userID)
   return ret.Get(0).(*User)
}

func (s *apiMock) Save(user *User) error {
   ret := s.Called(user)
   return ret.Error(0)
}

func TestTestify(t *testing.T) {
   mck := newAPIMock(t)

   service := Service{api: mck}

   user := &User{
  	ID:   	"8411fd4127",
  	username: "foo",
  	Domain:   "",
   }
   mck.On("Get", "8411fd4127").Return(user).Once()

   mck.On("Save", user).Return(nil).Once()

   data, err := service.GetDomain("8411fd4127")
   require.NoError(t, err)

   assert.Equal(t, "foo.example.com", data)
}
```

The mock system of Testify is great, but you still have to write a lot of things by hand. It is at this moment that you realize you need a generator.

## The problem with generic generators

When our team started working on our latest product, Traefik Hub, we were faced with the need to use mocks for a number of tests.
At first, we started with simple handwritten mocks. However, we quickly came across a few issues.
The first problem that we faced with handwritten mocks was the fact that writing that kind of code is extremely repetitive and boring to maintain.
After some discussion, the team decided to use a tool to generate them.
There are some existing mock generators in Go, and they are great, but none provided exactly what we wanted.
We expected to have fluent syntax and strongly typed mocks, and the generated mocks from these generators were weak against changes.

The generated mocks used a string to call the method, and the arguments were just variadic of interfaces.
When you are using strongly-typed languages like Go, you always prefer to stay in the world of strongly-typed things!

The main problem with weakly-typed elements is maintainability:
when you need to change the signature of a method (adding a new parameter, changing a type, etc.),
the changes will not be propagated to the mocks, and the compiler will not be able to see if something is broken.

At this point, the team thought that it was easier to come back to our precious handwritten mocks.
However, I had a different idea.
Why not create our own mock generator?

And that’s how Mocktail was born!

## Have a Mocktail, I say!

[Mocktail](https://github.com/traefik/mocktail) generates strongly typed mocks and provides a simple, fluent syntax.
The methods of the mocks have the same signature as the real method signature.
The number of parameters and the types of those parameters are the same as the real methods.

Using Mocktail is extremely simple. Create a file called `mock_test.go` and add directives for the mocks you need (for example, `// mocktail:MyObject`).

```go
package foo

// mocktail:Foo
// mocktail:Bar
// ...
```

Then run Mocktail at the root of your project.

```console
$ mocktail
```

You are now able to use your mocks easily!

```go
package foo

import (
   "testing"

   "github.com/stretchr/testify/assert"
   "github.com/stretchr/testify/mock"
   "github.com/stretchr/testify/require"
)

func TestMocktail(t *testing.T) {
   mck := newAPIMock(t)

   service := Service{api: mck}

   user := &User{
  	ID:   	"8411fd4127",
  	username: "foo",
  	Domain:   "",
   }
   mck.OnGet("8411fd4127").TypedReturns(user).Once()

   mck.OnSave(user).TypedReturns(nil).Once()

   data, err := service.GetDomain("8411fd4127")
   require.NoError(t, err)

   assert.Equal(t, "foo.example.com", data)
}
```

## Summing up

The good thing about mocktails is that you can go wild with them without any severe side effects — except maybe for a bit of a sugar rush!

We created Mocktail to serve the needs of our team and tackle the specific issues that we were facing in our process.
But my sincere hope is that this nifty little tool will be of use to many of you and save you hours of frustration manually creating mocks.

If you’re eager to try out Mocktail, you’ll find it on [GitHub](https://github.com/traefik/mocktail).

---

I published the orignial article inside the Traefik Labs blog: https://traefik.io/blog/mocktail-the-mock-generator-for-strongly-typed-mocks/

The article has been promoted inside the Go Weekly Newsletter: https://golangweekly.com/issues/445

I very happy and proud of that.
