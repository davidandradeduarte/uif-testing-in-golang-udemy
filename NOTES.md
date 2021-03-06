# Course notes

- Types of tests: unit, integration, functional
- Unit tests should test one single unit of code (usually single functions), mocking all the external dependencies
- Integration tests should test the integration between multiple layers of the application
- Functional tests should test input/output of an entire system or high level functional requiremenet
- Test files should be in the same directory as the files they'r testing
- Unit tests should be part of the same package as the package they'r testing. This also allows access to private, consts etc from the implementation
- Any test case should start with `Test`FunctionName
- Any test must have a `*testing.T` parameter
- Test steps:
    ```golang
    // init
    // execution
    // validation
    ```
    much like `arrange, act, assert`
- `go test -cover` will run the tests and print code coverage
- Code coverage can be missleading. you can have 100% code coverage without testing possible scenarios. code coverage only tells you which lines are executed when you run your tests 
- Prefer white box testing over black box testing (e.g write your tests inside the same package you are testing)
- Both integration and unit tests use the same interface `t *testing.T`. You can only tell the difference my looking at the test implementation
- go has native benchmark support. we can write a benchmark by creating a function starting with `Benchmark` receiving the `*testing.B` has parameter. benchmarks are written inside `_test` files
- `go test -bench=.` to run benchmarks cases
- We can use go routines and channels to test function timeouts/infinite loops.  
    e.g
    ```golang
    timeoutChan := make(chan bool, 1)
	defer close(timeoutChan)

	go func() {
		BubbleSort(elements)
		timeoutChan <- false
	}()

	go func() {
		time.Sleep(500 * time.Millisecond)
		timeoutChan <- true
	}()

	if <-timeoutChan {
		assert.Fail(t, "BubbleSort took more than 500ms")
		return
	}
    ```
- asserts are missing by design (because usually they stop the test execution on failure)
- we can use `github.com/stretchr/testify/assert` to write asserts in Go that don't stop the test execution on failure
- `TestMain` is the entry point for tests. It receives the interface `testing.M`
- the `init` functions run as soon as the package is imported
- we can't mock functions inside packages. Instead we should create methods on structs instead, that way we can mock them
- we can mock single methods by implementing their own interface and providing a mock implementation:  
	`service.go`
	```golang
	package service

	type service struct{}

	type serviceInterface interface {
		GetSomething() string
	}

	var (
		Service serviceInterface
	)

	func init() {
		Service = &service{}
	}

	func (s *service) GetSomething()string {
		// implementation
	}
	```
	`service_test.go`
	```golang
	package service
	
	type serviceMock struct{}

	var (
		getSomethingFunc func() string
	)

	func (s *serviceMock) GetSomething() string {
		return getSomethingFunc()
	}

	func TestGetSomething(t *testing.T) {
		// mock
		getSomethingFunc = func() string {
			return "mocked implementation"
		}
		sut.Service = &serviceMock{}
	}
	```
- functional tests are black box tests and should be created on their own separate package
- try to keep the main package (entry point) as clean as possible by calling another package's startup func. This way is easier to hit the entry point of your app without having to call the main function. E.g:  
	`main.go`
	```golang
	package main

	func main() {
		app.StartApp()
	}
	```
	`app.go`
	```golang
	package app

	func StartApp() {
		// run
	}
	```
	This way we can use `app.StartApp()` in our functional tests

## Additional notes

I didn't really liked `Section 5 - Testing SQL integration`. Seems a bit rubbish trying to reinvent the wheel by implementing your own mock client. Also the fact that we relied on this condition `if isMocked && !isProduction() {...}` makes me sick.  
Another thing I didn't like was the fact that he was always repelling external libraries (for dumb reasons imo), and then he came up with a rest client library, developed by the company he works for, and used it as an example (ツ)

Funny quote:
*deploying to production without tests is like drinking and driving*

Have a look at `github.com/stretchr/testify/mock`