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

Once recursive includes become available, you can share job templates like this analyzer.

Go GitLab linter plugins are maintained in the gitlab-org/language-tools/go/linters namespace.


