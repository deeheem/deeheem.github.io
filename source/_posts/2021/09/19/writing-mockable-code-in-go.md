---
title: Writing Mockable Code in Go
date: 2021-09-19 23:06:32
category:
- Languages
- Go
tags:
- "#go"
- "#testing"
---

## Introduction

Experienced engineers will agree how important writing tests is, and most of the time even prevent a Pull Request from going through when the appropriate tests are not added/modified.

With tests being so important, we need to understand how to write testable/mockable code. If you are new to mocking, you can read the basics first, [this](https://grails.org/blog/2018-06-22.html) is a good reference blog to start with. 

Let's identify the testing best practices in the next section, with the help of a microservice that exposes a single endpoint `/coffee` and returns a coffee type.

## Tasting (read: Testing) Coffee

### Code Walkthrough

We have a `main.go` file from where the code flow begins:

{% codeblock "main.go" lang:golang %}
package main

import (
	"github.com/gin-gonic/gin"
	"github.com/deeheem/goland-unit-test/code/controllers"
)

var (
	router = gin.Default()
)

func main() {
	router.GET("/coffee", controllers.GetCoffee)
	router.Run(":8080")
}
{% endcodeblock %}

The `GetCoffee()` function in `coffee_controller.go` interacts with the appropriate service:

{% codeblock "coffee_controller.go" lang:golang %}
package controllers

import (
    "net/http"
    "github.com/deeheem/goland-unit-test/code/services"
    "github.com/gin-gonic/gin"
 )

func GetCoffee(c *gin.Context) {
	result, err := services.HandleCoffee()
	if err != nil {
		c.String(http.StatusInternalServerError, err.Error())
		return
	}
	c.String(http.StatusOK, result)
}
{% endcodeblock %}

The `HandleCoffee()` function in `coffee_service.go` handles the main logic:

{% codeblock "coffee_service.go" lang:golang %}
package services

import "fmt"

const (
	coffee = "Cappuccino"
)

func HandleCoffee() (string, error) {
	fmt.Println("doing complex things here...")
	return coffee, nil
}
{% endcodeblock %}

Now run the application using `go run main.go` and call the API using `curl localhost:8080/coffee` or `curl localhost:8080/coffee -v`:
```bash
~/Projects/GoProjects/golang-unit-test » curl localhost:8080/coffee       
Cappuccino% 

~/Projects/GoProjects/golang-unit-test » curl localhost:8080/coffee -v
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 8080 (#0)
> GET /coffee HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.64.1
> Accept: */*
> 
< HTTP/1.1 200 OK
< Content-Type: text/plain; charset=utf-8
< Date: Sun, 19 Sep 2021 14:49:59 GMT
< Content-Length: 4
< 
* Connection #0 to host localhost left intact
Cappuccino* Closing connection 0
```

### Basic Test

Let's write a basic test to verify the error code and the result:

{% codeblock "coffee_controller_test.go" lang:golang %}
package controllers

import (
	"fmt"
	"net/http"
	"net/http/httptest"
	"testing"
	"github.com/gin-gonic/gin"
)

func TestCoffee(t *testing.T) {
	response := httptest.NewRecorder()
	context, _ := gin.CreateTestContext(response)

	GetCoffee(context)

	if response.Code != http.StatusOK {
		t.Error("response should be 200")
	}

	if response.Body.String() != "Cappuccino" {
		t.Error("response should say 'Cappuccino'")
	}
}
{% endcodeblock %}

Let's run the test with coverage:

```text
~/Projects/GoProjects/golang-unit-test/code/controllers(master*) » go test . -v -cover
=== RUN   TestCoffee
[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
 - using env:   export GIN_MODE=release
 - using code:  gin.SetMode(gin.ReleaseMode)

doing complex things here...
--- PASS: TestCoffee (0.00s)
PASS
coverage: 75.0% of statements
ok      github.com/deeheem/goland-unit-test/code/controllers    0.645s  coverage: 75.0% of statements
```

We see that as expected, our test does pass. But only 75% of the lines were covered. This is because our test doesn't cover the error scenario, and the following lines from `coffee_service.go` are not evaluated:
 
{% codeblock lang:golang %}
	if err != nil {
		c.String(http.StatusInternalServerError, err.Error())
		return
	}
{% endcodeblock %}

We want to develop a test case that tests this scenario as well. In order to do that we need to have complete and full control over what this `HandleCoffee()` function returns. Because Go is a compiled language, you cannot mock what this function actually does once this code is compiled.

So, the first thing that we need to keep in mind when writing mockable code is that if we put a package function, we are not going to be able to mock this function. So we need to make sure that we always use interfaces where we want to mock.

Before adding an interface though, let's add a struct in the next section.

### Adding struct

We add a struct called `coffeeService` and bind the function `HandleCoffee()` to this service, making it a method.

{% codeblock "coffee_service.go" lang:golang %}
package services

import "fmt"

type coffeeService struct{}

const (
	coffee = "Cappuccino"
)

var (
	CoffeeService = coffeeService{} // public variable to access the service
)

func (service coffeeService) HandleCoffee() (string, error) {
	fmt.Println("doing complex things here...")
	return coffee, nil
}
{% endcodeblock %}

Note that we have added a public variable `CoffeeService` so that other packages (like controllers) can use it to access the service.

### Adding interface

Next we add the interface `coffeeService` and rename our struct to `coffeeServiceImpl`, indicating that this struct now implements the interface.

{% codeblock "coffee_service.go" lang:golang %}
package services

import "fmt"

type coffeeService interface {
	HandleCoffee() (string, error)
}

type coffeeServiceImpl struct{}

const (
	coffee = "Cappuccino"
)

var (
	CoffeeService = coffeeServiceImpl{}
)

func (service coffeeServiceImpl) HandleCoffee() (string, error) {
	fmt.Println("doing complex things here...")
	return coffee, nil
}
{% endcodeblock %}

### Modifying the test

Now we modify our test as shown below:

{% codeblock "coffee_controller_test.go" lang:golang %}
package controllers

import (
	"fmt"
	"net/http"
	"net/http/httptest"
	"testing"
	"github.com/deeheem/goland-unit-test/code/services"
	"github.com/gin-gonic/gin"
)

type coffeeServiceMock struct {
}

func (mock coffeeServiceMock) HandleCoffee() (string, error) {
	fmt.Println("mocking complex things...")
	return "Cappuccino", nil
}

func TestCoffee(t *testing.T) {
    services.CoffeeService = coffeeServiceMock{} // overriding actual service
    
    response := httptest.NewRecorder()
    context, _ := gin.CreateTestContext(response)

    GetCoffee(context)

    if response.Code != http.StatusOK {
        t.Error("response should be 200")
    }
    
    if response.Body.String() != "Cappuccino" {
        t.Error("response should say 'Cappuccino'")
    }
}
{% endcodeblock %}

We are using the same interface we created in the service to create a mock. We provide a custom implementation of this mock in the test itself, and then override this mock implementation in the test case so it gets used during the test run (instead of the actual service implementation). 

This can be verified when we see the line `mocking complex things...` being printed instead of `doing complex things here...` while running the test case. On commenting the line `services.CoffeeService = coffeeServiceMock{}` in the test, we will again see `doing complex things here...` being printed as the mock implementation will not be used anymore.

### Making code coverage 100%

If we run the test, our code coverage is still 75% due to the same reasons discussed earlier. Although, we now have extensible code where we can provide a custom implementation of the `HandleCoffee()` method in order to test the error scenario as well.

{% codeblock "coffee_controller_test.go" lang:golang %}
package controllers

import (
	"fmt"
	"net/http"
	"net/http/httptest"
	"testing"
	"github.com/deeheem/goland-unit-test/code/services"
	"github.com/gin-gonic/gin"
)


type coffeeServiceMock struct {
    handleCoffeeFn func() (string, error)
}

func (mock coffeeServiceMock) HandleCoffee() (string, error) {
    return mock.handleCoffeeFn()
}

func TestCoffeeNoError(t *testing.T) {
    serviceMock := coffeeServiceMock{}
    serviceMock.handleCoffeeFn = func() (string, error) {
        return "Cappuccino", nil
    }
    
    services.CoffeeService = serviceMock
    
    response := httptest.NewRecorder()
    context, _ := gin.CreateTestContext(response)
    
    GetCoffee(context)
    
    if response.Code != http.StatusOK {
        t.Error("response should be 200")
    }
    
    if response.Body.String() != "Cappuccino" {
        t.Error("response should say 'Cappuccino'")
    }
}

func TestCoffeeWithError(t *testing.T) {
    serviceMock := coffeeServiceMock{}
    serviceMock.handleCoffeeFn = func() (string, error) {
        return "", errors.New("error getting coffee")
    }
    
    services.CoffeeService = serviceMock
    
    response := httptest.NewRecorder()
    context, _ := gin.CreateTestContext(response)
    
    GetCoffee(context)
    
    if response.Code != http.StatusInternalServerError {
        t.Error("response should be 500")
    }
    
    if response.Body.String() != "error getting coffee" {
        t.Error("response should say 'error'")
    }
}
{% endcodeblock %}

We added an attribute `handleCoffeeFn` inside the custom implementation of the CoffeeService interface, i.e. `coffeeServiceMock`. This attribute helps us plugin different implementations of HandleCoffee() method as per test case requirements.

Now if we run the test with coverage, we get a 100% result:
```text
~/Projects/GoProjects/golang-unit-test/code/controllers(master*) » go test . -v -cover
=== RUN   TestCoffeeWithError
[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
 - using env:   export GIN_MODE=release
 - using code:  gin.SetMode(gin.ReleaseMode)

--- PASS: TestCoffeeWithError (0.00s)
=== RUN   TestCoffeeNoError
[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
 - using env:   export GIN_MODE=release
 - using code:  gin.SetMode(gin.ReleaseMode)

--- PASS: TestCoffeeNoError (0.00s)
PASS
coverage: 100.0% of statements
ok      github.com/deeheem/goland-unit-test/code/controllers    1.031s  coverage: 100.0% of statements
```

## Learnings

In order to write testable code in Go, we need to keep the following points in mind:
1. Implement functions not as functions, but as methods, i.e. use structs
2. Define an interface that defines every method we need in the struct
3. Inside every test case, specify the behaviour you expect from the mocked object. This makes it easy for you to have control over what the function call actually returns.

## Mocking Frameworks

Like every other programming language, Go also has various libraries and packages to help generate mocks and write tests.

But if you are new to testing, let's first understand why there is a need for mocking frameworks in the first place.

### Need for Mocking Frameworks

Say that you are testing your code that is still in development. In order to achieve the right results, you need to test its interactions with system resources, outside applications, and other dependencies. Unfortunately, you learn early on that that is not possible. Utilizing a mocking framework allows for realistic emulations of the required interactions.
 
Mocked objects take the place of any large/complex/external objects your code needs access to in order to run.
 
They are beneficial for a few reasons:
 
1. Your tests are meant to run fast and easily. If your code depends on, say, a database connection then you would need to have a fully configured and populated database running in order to run your tests. This can get annoying, so you create a replacement - a "mock" - of the database connection object that just simulates the database.
2. You can control exactly what output comes out of the Mock objects and can therefore use them as controllable data sources for your tests.
3. You can create the mock before you create the real object in order to refine its interface. This is useful in Test-driven Development.

### Mocking Frameworks in Go

In a typical business application with a lot of code, writing mocks can easily increase the development time. Fortunately, the following tools come in very handy in Go for writing unit tests:

* [golang/mock](https://github.com/golang/mock)
* [testify/mock](https://github.com/stretchr/testify)

[mockery](https://github.com/vektra/mockery) is an awesome tool to easily generate mocks for golang interfaces. It uses the [testify/mock](https://github.com/stretchr/testify) package internally.
