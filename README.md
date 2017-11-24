# retry

[![Release](https://img.shields.io/github/release/avast/retry-go.svg?style=flat-square)](https://github.com/avast/retry-go/releases/latest)
[![Software License](https://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat-square)](LICENSE.md)
[![Travis](https://img.shields.io/travis/avast/retry-go.svg?style=flat-square)](https://travis-ci.org/avast/retry-go)
[![AppVeyor](https://ci.appveyor.com/api/projects/status/fieg9gon3qlq0a9a?svg=true)](https://ci.appveyor.com/project/JaSei/retry-go)
[![Go Report Card](https://goreportcard.com/badge/github.com/avast/retry-go?style=flat-square)](https://goreportcard.com/report/github.com/avast/retry-go)
[![GoDoc](https://godoc.org/github.com/avast/retry-go?status.svg&style=flat-square)](http://godoc.org/github.com/avast/retry-go)
[![codecov.io](https://codecov.io/github/avast/retry-go/coverage.svg?branch=master)](https://codecov.io/github/avast/retry-go?branch=master)
[![Sourcegraph](https://sourcegraph.com/github.com/avast/retry-go/-/badge.svg)](https://sourcegraph.com/github.com/avast/retry-go?badge)

Simple library for retry mechanism

slightly inspired by
[Try::Tiny::Retry](https://metacpan.org/pod/Try::Tiny::Retry)


### SYNOPSIS

http get with retry:

    url := "http://example.com"
    var body []byte

    err := retry.Do(
    	func() error {
    		resp, err := http.Get(url)
    		if err != nil {
    			return err
    		}
    		defer resp.Body.Close()
    		body, err = ioutil.ReadAll(resp.Body)
    		if err != nil {
    			return err
    		}

    		return nil
    	},
    )

    fmt.Println(body)

[next examples](https://github.com/avast/retry-go/examples)


### SEE ALSO

* [giantswarm/retry-go](https://github.com/giantswarm/retry-go) - slightly
complicated interface.

* [sethgrid/pester](https://github.com/sethgrid/pester) - only http retry for
http calls with retries and backoff

* [cenkalti/backoff](https://github.com/cenkalti/backoff) - Go port of the
exponential backoff algorithm from Google's HTTP Client Library for Java. Really
complicated interface.

* [rafaeljesus/retry-go](https://github.com/rafaeljesus/retry-go) - looks good,
slightly similar as this package, don't have 'simple' `Retry` method

* [matryer/try](https://github.com/matryer/try) - very popular package,
nonintuitive interface (for me)


### BREAKING CHANGES

0.3.0 -> 1.0.0

* `retry.Retry` function are changed to `retry.Do` function

* `retry.RetryCustom` (OnRetry) and `retry.RetryCustomWithOpts` functions are
now implement via functions produces Options (aka `retry.OnRetryFunction`)

## Usage

#### func  Do

```go
func Do(retryableFunction Retryable, opts ...Option) error
```

#### type Error

```go
type Error []error
```

Error type represents list of errors in retry

#### func (Error) Error

```go
func (e Error) Error() string
```
Error method return string representation of Error It is an implementation of
error interface

#### func (Error) WrappedErrors

```go
func (e Error) WrappedErrors() []error
```
WrappedErrors returns the list of errors that this Error is wrapping. It is an
implementation of the `errwrap.Wrapper` interface in package
[errwrap](https://github.com/hashicorp/errwrap) so that `retry.Error` can be
used with that library.

#### type OnRetry

```go
type OnRetry func(n uint, err error)
```

Function signature of OnRetry function n = count of tries

#### type Option

```go
type Option func(*config)
```

Option represents an option for retry.

#### func  Delay

```go
func Delay(delay time.Duration) Option
```
Delay set delay between retry default are 1e5 units

#### func  OnRetryFunction

```go
func OnRetryFunction(onRetryFunction OnRetry) Option
```
OnRetryFunction function callback are called each retry

log each retry example:

    retry.Do(
    	func() error {
    		return errors.New("some error")
    	},
    	retry.OnRetryFunction(func(n unit, err error) {
    		log.Printf("#%d: %s\n", n, err)
    	}),
    )

#### func  RetryIfFunction

```go
func RetryIfFunction(retryIfFunction RetryIfFunc) Option
```
RetryIfFunction controls whether a retry should be attempted after an error
(assuming there are any retry attempts remaining)

skip retry if special error example:

    retry.Do(
    	func() error {
    		return errors.New("special error")
    	},
    	retry.RetryIfFunction(func(err error) bool {
    		if strings.Contains(err.Error, "special error") {
    			return false
    		}
    		return true
    	})
    )

#### func  Tries

```go
func Tries(tries uint) Option
```
Tries set count of retry default is 10

#### func  Units

```go
func Units(units time.Duration) Option
```
Units set unit of delay (probably only for tests purpose) default are
microsecond

#### type RetryIfFunc

```go
type RetryIfFunc func(error) bool
```

Function signature of retry if function

#### type Retryable

```go
type Retryable func() error
```

Function signature of retryable function

## Contributing

Contributions are very much welcome.

### Makefile

Makefile provides several handy rules, like README.md `generator` , `setup` for prepare build/dev environment, `test`, `cover`, etc...

Try `make help` for more information.

### Before pull request

please try:
* run tests (`make test`)
* run linter (`make lint`)
* if your IDE don't automaticaly do `go fmt`, run `go fmt` (`make fmt`)

### README

README.md are generate from template [.godocdown.tmpl](.godocdown.tmpl) and code documentation via [godocdown](https://github.com/robertkrimen/godocdown).

Never edit README.md direct, because your change will be lost.
