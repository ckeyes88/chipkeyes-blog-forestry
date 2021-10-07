---
timeToRead: 6
authors: ["Chip Keyes"]
title: Go Modules Done Better
excerpt: 'A follow up to my previous article Go Modules in Real Life A little over a year ago, I published an article detailing how my team at Helpful Human...'
date: 2020-10-03
hero: ''
draft: false

---
> A follow up to my previous article [Go Modules in Real Life](/go-modules-in-real-life "Go Modules in Real Life")

A little over a year ago, I published an article detailing how my team at [Helpful Human](https://helpfulhuman.com) and I set up our project using Go modules. We used a multi-module repo and used the `replace` directive to point the Go tool chain to relative path modules. If you're interested in how we did that you can go back and read that post linked above.

### Our Original Approach
If you read my previous article you'll recall that our project consisted of a growing collection of webhook handlers. These webhooks were each individual AWS Lambda functions as isolated executables. They also shared a lot of functionality since they were all from the same application. Because of this, we included some shared data structures, database access libraries, etc. in the project resulting in us setting up what's referred to as a multi-module repository. We used Go's `replace` directive in the `go.mod` file to tell any module where local dependencies are found.

This allowed us to share the modules across each of the various webhooks. It also allowed us to keep all our code in the same repository. We didn't have to duplicate code across 7+ webhooks and we didn't have to set up individual CI/CD pipelines for each one. We can argue the merits of that last goal depending on how micro you feel microservices should get but that's a topic for a different post.

In all honesty, I was also on a bit of a time crunch for getting these webhooks working. So when it worked, I was pretty pumped and not thinking about alternative solutions.

I've laid out those details in my previous post. All that is to say, I've recently had a change of heart and mind since writing that article.

### That Nagging Feeling
>  If you've never written some code that made you want to go back and rewrite it a month or even a week later, you haven't been coding long enough.

Ever since we got the webhooks set up and working as desired, I'd had a nagging feeling that my solution was a bit hacky.  _I'm sure none of you know what this is like since you all write perfect code from the start, but humor me for a moment_

Russ Cox's (core Go contributor) confirmed my suspicions when he stated that you should use multi-module repos sparingly. I knew I needed to find a better way to approach this but I didn't have the time to go down that road until more recently.

Let's get into some code to see how things have changed in our repo over the last year for the better.

## One Module To Rule Them All
If you followed my previous walk through then you'll recognize some of this code.

To catch you up on where we left off, we have three modules. Two of which are isolated executables that work as simple web servers. These could be anything with a main function. The third is a logging library that represents shared logic. It could be a database connection, middleware, or anything not run as a main executable. The original file structure should match what you see below:
```
/my_project
    /web_server
        go.mod
		go.sum
		main.go
    /logger
        go.mod
		go.sum
		logger.go
    /web_server_two
        go.mod
		go.sum
		main.go
```

#### Step 1: Combine into a single module
The first thing we're going to do is remove the three submodules.

To do this, delete the `go.mod` and `go.sum` files in all the sub directories. If you're just joining us, create a top level working directory and three sub directories labeled like you see above.

Next, we're going to create a new single module at the top level directory. To do this run the command below:

```bash
go mod init github.com/<GH USERNAME>/myproject
```
> Side note about module names. You can actually name it whatever you want but there are some reasons why it's nice to stick with the github url. This naming convention only matters if you're planning to publish it as a public go module. If it's a private module then name it what you want. It also helps with backwards compatibility for Go <1.11. To my knowledge this only matters if you're building a module for use with <1.11 so if you're not then don't worry about it.

At the end of this, your file structure should match what's below.
 ```
 /my_project
 	go.mod
    /web_server
		main.go
    /logger
		logger.go
    /web_server_two
		main.go
```

We now have a single go module (or the start of one) that contains two executables and a shared logging library. The `go.mod` file will be empty except for the two lines below:

```go
module github.com/<GH USERNAME>/my_project

go 1.12
```

#### Step 2: Update the Logger
I've switched up the logger a little from the last post to show a 3rd party dependency. I'll be using the logging library `logrus` which I wrapped with my own logger package. In a real project, you might do this so that you can report a log to multiple outputs, create a middleware function and to make it easy to mock for testing.

In our case, we're going to wrap the logrus library and expose a log function for each of our severity levels, `error`, `warn`, `info`, and `debug`. Your logger should match my logger file below:

```go
package logger

import (
	log "github.com/sirupsen/logrus"
)

// Logger is a base struct that could eventually maintain connections to something like bugsnag or logging tools
type Logger struct {
	serviceName string
}

// NewLogger creates a new instance of the custom logger struct and returns it
func NewLogger(serviceName string) *Logger {
	var l = new(Logger)
	l.serviceName = serviceName

	return l
}

// LogDebug is a publicly exposed info log that passes the message along correctly
func (l *Logger) LogDebug(messages ...interface{}) {
	log.WithField("service-name", l.serviceName).Debug(messages)
}

// LogInfo is a publicly exposed info log that passes the message along correctly
func (l *Logger) LogInfo(messages ...interface{}) {
	log.WithField("service-name", l.serviceName).Info(messages)
}

// LogWarning is a publicly exposed info log that passes the message along correctly
func (l *Logger) LogWarning(messages ...interface{}) {
	log.WithField("service-name", l.serviceName).Warn(messages)
}

// LogError is a publicly exposed info log that passes the message along correctly
func (l *Logger) LogError(messages ...interface{}) {
	log.WithField("service-name", l.serviceName).Error(messages)
}
```

Once you get the logger in place you can run `go get -u` to fetch the `logrus` dependency. Note that this is the package `logger` which is inside the module `github.com/<GH USERNAME>/my_project`.

Now, let's test this out by importing it into one of our main applications.

#### Step 3: Import your sub packages
Now that we have this sub-package set up we can be import it anywhere else in this project using `import "github.com/<GH USERNAME>/my_project/<SUB PKG>"`.

Let's see this in action by importing it into our web server.

```go
package main

import (
	"net/http"

	"github.com/ckeyes88/my_project/logger"
)

func main() {
	// Create a new logger initialized with the service name
	l := logger.NewLogger("Web Server 1")

	// wrap our hello handler function
	http.Handle("/hello", loggerware(l, http.HandlerFunc(handle)))
	http.ListenAndServe(":5600", nil)
}

func handle(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("world"))
}

// loggerware can wrap any handler function and will print out the datetime of the request
// as well as the path that the request was made to.
func loggerware(l *logger.Logger, next http.HandlerFunc) http.HandlerFunc {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		requestPath := r.URL
		l.LogInfo("Request Made To: ", requestPath)
	})
}
```
As you can see, we had to change almost nothing about our web server. The only change we made was to the logger constructor because we added that initialization function.

#### Step 4: Update Web Server Two
Lastly, just to prove that it's shareable and has feature parody with the multi-module version, lets add our second web server, cleverly named `web_server_two`.

 Add this code to your `web_server_two/main.go` file:

```go
package main

import (
	"net/http"

	"github.com/ckeyes88/my_project/logger"
)

func main() {
	l := logger.NewLogger("Web Server 2")

	http.Handle("/valar-morghulis", loggerware(l, http.HandlerFunc(handle)))
	http.ListenAndServe(":5500", nil)
}

func handle(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("valar dohaeris"))
}

func loggerware(l *logger.Logger, next http.HandlerFunc) http.HandlerFunc {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		requestPath := r.URL
		l.LogInfo("Request Made To Server Two: ", requestPath)
	})
}
```

As you can see you have no trouble importing the shared logger library anywhere else in the module. And when you run it you see that you get specific messages corresponding to the respective service names. This demonstrates a much simpler way to share code across many main executables.

## Why should we avoid replace and when should we use it?
In closing, I want to bring up a couple improvements that this gives us over the original solution. I also want to offer one or two reasons why you might still use the `replace` directive.

As you can see this solution is much less maintenance for the same functionality. There isn't any real benefit to the multi-module approach over this when you're trying to achieve our initial goals:

- Single shared sub-packages
- Multiple main executables with a single repo
- A single CI/CD pipeline

The multi-module approach adds unnecessary complexity by forcing you to manage individual dependencies across all the sub modules.

There isn't any reason to use multi-module repositories in the way I did in my previous article. A more legitimate use case for the multi-module approach to quarantine a version of your module which will always be available in your repository. This allows you to make potentially major changes to the main version at the master level. The Go team provided this exception knowing that people might want to lock down a current stable version before making major changes.

Another good use case for `replace` is when you're making local changes to Go modules that you're using in another project. This is a good way to test out your development changes locally so you don't have to publish new versions of your Go module every time you want to test them.

## Conclusion
Hopefully, this gives you some more tools to use while building out your Go modules. Thanks for reading this far. If you have questions or comments feel free to use the form below to reach out!

### More Resources
Here's some great resources to continue learning about Go modules.
- [The Official Docs](https://blog.golang.org/using-go-modules)
- [More from the author of Go modules](https://research.swtch.com/vgo)

[sc name="post-signature"]