# Coding Guide

This document describes various guidelines and best practices for Golang projects under f5ingress 

These guidelines have been developed to achieve following outcomes:

* Enable high velocity of feature development by adopting to consitent code layout and format across different services and
  libraries
* Investing in tools and processes to catch problems early in dev-test-deploy cycle
* Making code reviews efficient and faster 



## Golang guidelines

We will follow recommendations from
[How to Write Go Code](https://golang.org/doc/code.html) and
[Effective Go]( https://golang.org/doc/effective_go.html). In addition,

* Document the public usage of a library or executable using `doc.go` for e.g.
  see [this](https://github.com/coreos/etcd/blob/master/embed/doc.go)
* Specify [interfaces](http://jordanorelli.com/post/32665860244/how-to-use-interfaces-in-go)
  exported by a library using `interface.go`
  ([example](https://github.com/appcelerator/amp/tree/2dd6ddfef77ba0efe0563f34e0805d1111ed104a/data/storage))  
  * The implementation for the interfaces SHOULD be in file(s) in a
    sub-directory (and therefore separate package,
    ([example](https://github.com/appcelerator/amp/blob/2dd6ddfef77ba0efe0563f34e0805d1111ed104a/data/storage/etcd/store.go))
  * Prefer to expose library abstractions via interface type instead of
    struct type. This allows for enhanced versions of structs over time
    without needing to modify consumers that don’t need it.
  * When possible implement and/or extend golang stdlib [interfaces](http://sweetohm.net/article/go-interfaces.en.html)
    (e.g [net.Conn](https://github.com/golang/go/blob/go1.9.2/src/net/net.go#L112))
    instead of new interface type. 
* Use NewXXX for XXX struct type, hiding complex constructions of struct w.r.t.
  set of fields to initialize exposed by simpler parameters in NewXXX, aka
  factory pattern, for e.g. see
  [this](http://matthewbrown.io/2016/01/23/factory-pattern-in-golang/)


## Language agnostic guidelines

* Use a pattern of developing most functionality in small libraries with
  succinct interfaces (e.g. VFS for filesystem) and executables as a thin layer
  on top of libraries with command line flags. I/O endpoints can be decided by
  executables and libraries can work with interfaces - e.g. libraries use
  [Reader](https://tour.golang.org/methods/21)/Writer interface and executable
  chooses whether to a file or socket.
* Use [context](https://blog.golang.org/context) extensively  
  A context could be started because of an API invocation or timer invocation.
  Context information should propagate through multiple process or thread
  boundaries. Log and trace invocations should specify context to use. A
  detailed description of using context is explained [here](https://gitlab.com/volterra/ves.io/specifications/issues/11).
  This pattern helps in monitoring response times at different
  layers and also for faster MTTI/MTTR. Deadlines and Cancellations on context
  SHOULD be used.
* Use a worker pool library similar to [this](https://brandur.org/go-worker-pool).
* Configuration preference for an executable should be following (increasing
  order of priority):
  * Code default
  * Environment variable
  * Launch flags / command line.
  * For daemons, runtime configuration through API (which SHALL be persisted).
    Daemons should also expose via introspect API currently evaluated values
    for all configuration items.
* Services SHALL be able to source system configuration from a config
  interface which could have a database or file-system backing store. This
  solves chicken/egg problem of bootstrapping.
* Any security related configuration SHALL NOT have any defaults in code,
  packaging or deployment scripts - force user to specify it.
* Register signal for SIGKILL and place graceful termination code
* Register readiness, liveness, and drain-mode endpoint handlers for service
  daemons
* Use application level keepalives on long-lived connections. Connection
  lifecycle should not have any implied meaning for state change - i.e. audit
  changes happen only on app-level keepalive loss.
* All service daemons SHALL expose a REST API and gRPC API. Transport layer
  (REST/gRPC) should normalize I/O to protobuf messages and invoke business
  logic in service.
* All library init/register routine should return a 'handle' so that multiple
  instances of the library abstraction could be used in same executable. This
  handle could be allocated in init/register to serve as context inside which
  library can maintain house-keeping data.
* All errors (input-errors, handled-server-errors or unhandled-server-errors)
  returned by a daemon SHALL have both human-readable and machine-parsable
  details. Errors should have additive contextualized message.  
  For e.g. as mentioned [here:](https://dave.cheney.net/2016/04/27/dont-just-check-errors-handle-them-gracefully)


Automatic linting
All Go projects should include these GitLab CI/CD jobs:

```
lint:
  image: registry.gitlab.com/gitlab-org/gitlab-build-images:golangci-lint-alpine
  stage: test
  script:
    # Use default .golangci.yml file from the image if one is not present in the project root.
    - '[ -e .golangci.yml ] || cp /golangci/.golangci.yml .'
    # Write the code coverage report to gl-code-quality-report.json
    # and print linting issues to stdout in the format: path/to/file:line description
    # remove `--issues-exit-code 0` or set to non-zero to fail the job if linting issues are detected
    - golangci-lint run --issues-exit-code 0 --out-format code-climate | tee gl-code-quality-report.json | jq -r '.[] | "\(.location.path):\(.location.lines.begin) \(.description)"'
  artifacts:
    reports:
      codequality: gl-code-quality-report.json
    paths:
      - gl-code-quality-report.json

Including a .golangci.yml in the root directory of the project allows for configuration of golangci-lint. All options for golangci-lint are listed in this example.
```

Once recursive includes become available, you can share job templates like this analyzer.

Go GitLab linter plugins are maintained in the gitlab-org/language-tools/go/linters namespace.


## Dependencies
Dependencies should be kept to the minimum. The introduction of a new dependency should be argued in the merge request, as per our Approval Guidelines. Both License Scanning and Dependency Scanning should be activated on all projects to ensure new dependencies security status and license compatibility.

## Modules
In Go 1.11 and later, a standard dependency system is available behind the name Go Modules. It provides a way to define and lock dependencies for reproducible builds. It should be used whenever possible.

When Go Modules are in use, there should not be a vendor/ directory. Instead, Go automatically downloads dependencies when they are needed to build the project. This is in line with how dependencies are handled with Bundler in Ruby projects, and makes merge requests easier to review.

In some cases, such as building a Go project for it to act as a dependency of a CI run for another project, removing the vendor/ directory means the code must be downloaded repeatedly, which can lead to intermittent problems due to rate limiting or network failures. In these circumstances, you should cache the downloaded code between.

There was a bug on modules checksums in Go versions earlier than v1.11.4, so make sure to use at least this version to avoid checksum mismatch errors.


## Testing
Testing frameworks
We should not use any specific library or framework for testing, as the standard library provides already everything to get started. If there is a need for more sophisticated testing tools, the following external dependencies might be worth considering in case we decide to use a specific library or framework:

Testify
httpexpect


### Table-Driven Tests
Using Table-Driven Tests is generally good practice when you have multiple entries of inputs/outputs for the same function. Below are some guidelines one can follow when writing table-driven test. These guidelines are mostly extracted from Go standard library source code. Keep in mind it’s OK not to follow these guidelines when it makes sense.


## Benchmarks
Programs handling a lot of IO or complex operations should always include benchmarks, to ensure performance consistency over time.


## Error handling
Adding context
Adding context before you return the error can be helpful, instead of just returning the error. This allows developers to understand what the program was trying to do when it entered the error state making it much easier to debug.

For example:

// Wrap the error
return nil, fmt.Errorf("get cache %s: %w", f.Name, err)

// Just add context
return nil, fmt.Errorf("saving cache %s: %v", f.Name, err)

A few things to keep in mind when adding context:

Decide if you want to expose the underlying error to the caller. If so, use %w, if not, you can use %v.
Don’t use words like failed, error, didn't. As it’s an error, the user already knows that something failed and this might lead to having strings like failed xx failed xx failed xx. Explain what failed instead.
Error strings should not be capitalized or end with punctuation or a newline. You can use golint to check for this.

### Naming
When using sentinel errors they should always be named like ErrXxx.
When creating a new error type they should always be named like XxxError.

### Checking Error types
To check error equality don’t use ==. Use errors.Is instead (for Go versions >= 1.13).
To check if the error is of a certain type don’t use type assertion, use errors.As instead (for Go versions >= 1.13).


## Logging
The usage of a logging library is strongly recommended for daemons. Even though there is a log package in the standard library, we generally use Logrus. Its plugin (“hooks”) system makes it a powerful logging library, with the ability to add notifiers and formatters at the logger level directly.

## Structured (JSON) logging
Every binary ideally must have structured (JSON) logging in place as it helps with searching and filtering the logs. At GitLab we use structured logging in JSON format, as all our infrastructure assumes that. When using Logrus you can turn on structured logging by using the build in JSON formatter. This follows the same logging type we use in our Ruby applications.

## How to use Logrus
There are a few guidelines one should follow when using the Logrus package:

When printing an error use WithError. For example, logrus.WithError(err).Error("Failed to do something").
Since we use structured logging we can log fields in the context of that code path, such as the URI of the request using WithField or WithFields. For example, logrus.WithField("file", "/app/go").Info("Opening dir"). If you have to log multiple keys, always use WithFields instead of calling WithField more than once.

## Tracing and Correlation
LabKit is a place to keep common libraries for Go services. Currently it’s vendored into two projects: Workhorse and Gitaly, and it exports two main (but related) pieces of functionality:

gitlab.com/gitlab-org/labkit/correlation: for propagating and extracting correlation ids between services.
gitlab.com/gitlab-org/labkit/tracing: for instrumenting Go libraries for distributed tracing.
This gives us a thin abstraction over underlying implementations that is consistent across Workhorse, Gitaly, and, in future, other Go servers. For example, in the case of gitlab.com/gitlab-org/labkit/tracing we can switch from using Opentracing directly to using Zipkin or Gokit’s own tracing wrapper without changes to the application code, while still keeping the same consistent configuration mechanism (that is, the GITLAB_TRACING environment variable).


## style guidelines
The following are some style guidelines that are specific to the Secure Team.

## Code style and format

Use goimports -local gitlab.com/gitlab-org before committing. goimports is a tool that automatically formats Go source code using Gofmt, in addition to formatting import lines, adding missing ones and removing unreferenced ones. By using the -local gitlab.com/gitlab-org option, goimports groups locally referenced packages separately from external ones. See the imports section of the Code Review Comments page on the Go wiki for more details. Most editors/IDEs allow you to run commands before/after saving a file, you can set it up to run goimports -local gitlab.com/gitlab-org so that it’s applied to every file when saving.

## Naming branches
Only use the characters a-z, 0-9 or - in branch names. This restriction is due to the fact that go get doesn’t work as expected when a branch name contains certain characters, such as a slash /:

$ go get -u gitlab.com/gitlab-org/security-products/analyzers/report/v3@some-user/some-feature

go get: gitlab.com/gitlab-org/security-products/analyzers/report/v3@some-user/some-feature: invalid version: version "some-user/some-feature" invalid: disallowed version string

If a branch name contains a slash, it forces us to refer to the commit SHA instead, which is less flexible. For example:

$ go get -u gitlab.com/gitlab-org/security-products/analyzers/report/v3@5c9a4279fa1263755718cf069d54ba8051287954

go: downloading gitlab.com/gitlab-org/security-products/analyzers/report/v3 v3.15.3-0.20221012172609-5c9a4279fa12
...


