---
timeToRead: 3
authors: ["Chip Keyes"]
title: Go Modules In Real Life
excerpt: "UPDATE: I've written a follow up article with some improvements to my approach which you can read about here Ok, so Go modules are the new hot thing in..."
date: 2019-06-16
hero: ''
draft: false

---

> UPDATE: I've written a follow up article with some improvements to my approach which you can read about [here](/go-modules-done-right)

Ok, so Go modules are the new hot thing in Go ever since their beta release in version 1.11. If you've spent time working in Go you know that it's a simple language that packs some powerful features. Also, if you've spent anytime working in Go before 1.11 you know about the infamous `GOPATH`. It was one of the most confusing things for me while learning and using Go. It only takes a simple google search for the term `GOPATH` to show how challenging it has been to the Go community. Suffice it to say, Go developers have been pretty ecstatic about modules since they address a lot of frustrations people had in previous versions.

But Go modules aren't without their own complexities and challenges. Nor do they have the historical blogs and forum posts to help the new comer out. One area that I've struggled with in my team's project is how to organize your code in a way that shares common packages to one or more main programs. It took me a while to find a solution that worked and didn't seem like an anti-pattern but it has made this much simpler in our project. So my hope is that in sharing what we came up with, it might save you some trouble. 

Either that or you can point out how we're doing it wrong so we can improve our code.

## Package vs. Module


> "A module is a collection of related Go packages that are versioned together as a single unit."
--Go Module Docs

First, I want to discuss the difference between a module and package.  The Go module documentation describes a module as, " a module of related Go packages that are versioned together as a single unit". Thus, we see that a package is the base building block and at some point one or more packages can become a module. 

There's been a lot of questions about when to make something a module and when to leave it as a package. Going back to our definition there are two major reasons why you would want to make some code it's own module. 

1. We want to maintain and version the code on its own away from other application pieces
1. We want to share that code across one or more projects or with the public
1. We have a mono-repo codebase containing one or more programs

We'll be discussing the third reason in this post. In this case you have some amount of common code that you'd rather not copy and paste for each sub application. For this we can use what we call a multi-module repo. The Go maintainers created this to allow many versions of the same module to live in the same repo. We're going to use it in a different way which I'll get into shortly.

## The Setup
To give some context on why I started researching this let me describe our project a bit.

Our project has several webhook handlers written in Go as AWS Lambda functions. These lambdas pull messages from Simple Queue Service (SQS) on AWS and process the message. If it processed successfully it's deleted from the queue. We have one repo containing all our lambdas and each lambda is its own executable module. But, each lambda has a subset of common logic shared between them all. Logic like data models, database connection code, and external service calls. They all live in the same Github repository and take on the file structure shown below.

```bash
/webhooks
	/webhookA (executable)
	/webhookB (executable)
	/db
	/models
	/api
```

I didn't want to put a copy of the shared code in every webhook nor did I want to maintain a single repo for an ever growing list of webhook handlers. So I started looking into how I could share this code between all the lambdas. In reading through a lot of documentation I've pieced together something that works for our project and doesn't seem like an off the wall approach.

##### Just to recap, here's the requirements of what we're trying to accomplish
- Maintain several individual pieces of a Go project in a single folder and repo. In our case these pieces are webhooks but they could be standalone microservices, or a set of libraries like an SDK.
- Allow shared common code to live in it's own folder and be imported by one or more of the main programs

## The Example
Since lambdas are a little more work to run and show without deploying to AWS, I'm going to use a contrived example. whereby we build two simple web servers along side a common logger module that both servers use.

As I mentioned above, to do this we're going to create what's referred to as a multi-module repository.

So without further ado... here we go.

## The Base Folder
> Note: make sure you have Go 1.11 or greater installed and configured correctly following the directions [here](https://golang.org/doc/install) as I won't be going into detail about Go setup or configuration.

At the top level we're going to create the start of our project. If you're following along, choose a directory somewhere outside of your `GOPATH` to start writing code.

Inside that folder create the following folder structure:
```bash
/my_project
	/web_server
	/web_server_two
	/logger
```

## The Base Modules
Inside each directory we're going to initialize them as a Go Module by running the following commands:
```bash
go mod init my_project/web_server
go mod init my_project/web_server_two
go mod init my_project/logger
```
This will run Go's built in script to create a `go.mod` file which tracks the module's dependencies. 

After this our folder structure should now look like this:
```bash
/my_project
	/web_server
		go.mod
	/logger
		go.mod
	/web_server_two
		go.mod
```

where each `go.mod` file contains the following with module name swapped out respectively:
```go
module my_project/web_server

go 1.12
```

Notice at this point we have a single repo, single project directory and three individual modules, one of which will be shareable between the other two.

## The Logger
We'll start with the shared logger module which will abstract the log formatting and functionality into a separate module.

Under the *logger* directory, create a file called `logger.go` containing the following code.

```go
package logger

import (
	"fmt"
	"time"
)

// LogLevel is an enum-like type that we can use to designate the log level
type LogLevel int
const (
	DEBUG = iota
	INFO
	WARNING
	ERROR
)

// Logger is a base struct that could eventually maintain connections to something like bugsnag or logging tools
type Logger struct {}

// log is a private function that manages the internal logic about what and how to log data depending on the log level
func (l *Logger) log(level LogLevel, messages ...interface{}) {
	now := time.Now()
	var logType string
	switch level {
	case DEBUG:
		logType = "[DEBUG]"
		break
	case WARNING:
		logType = "[WARNING]"
		break
	case ERROR:
		logType = "[ERROR]"
		break
	default:
		logType = "[INFO]"
		break

	}
	// format the output in a somewhat friendly way
	fmt.Println("-----------------------------------------")
	fmt.Printf("%s - %s\n", logType, now)
	for _, message := range messages {
		fmt.Printf("%+v\n", message)
	}
	fmt.Println("-----------------------------------------")
}

// LogDebug is a publicly exposed info log that passes the message along correctly
func (l *Logger) LogDebug(messages ...interface{}) {
	l.log(DEBUG, messages...)
}

// LogInfo is a publicly exposed info log that passes the message along correctly
func (l *Logger) LogInfo(messages ...interface{}) {
	l.log(INFO, messages...)
}

// LogWarning is a publicly exposed info log that passes the message along correctly
func (l *Logger) LogWarning(messages ...interface{}) {
	l.log(WARNING, messages...)
}

// LogError is a publicly exposed info log that passes the message along correctly
func (l *Logger) LogError(messages ...interface{}) {
	l.log(ERROR, messages...)
}
```
I know, this logger isn't winning any awards, but it does the job for our example.

As the code comments layout, we have a pretty basic enum to show the log level, an empty struct to represent the logger, and a few publicly  available functions to abstract away the formatting logic. 

Now we're ready to start using this module in our other applications.

### The Web Server
The first web server is going to be a basic hello world http server. To make use of our logger I'm go add a middleware function that prints out a lot on every request.

To get started change your directory to  the web server module and create a file called `main.go`. Now remember this is an executable program so it will need a main package and main function to run.

```go
package main

import (
	"net/http"
)

func main() {
	http.Handle("/hello", http.HandlerFunc(handle))
}

func handle(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("world"))
}
```

Ok, now the interesting part. As you can see we need a way to import our shared logger library. We can do this in two ways.

Option 1: We push this code up to github and tag it as a released version number. Depending on if it's a public or private repo, we'd also have to deal with the authentication for the `go get` command. This is a perfectly valid option and to be honest is going to be the right approach 90% of the time. The drawback here is that it's really meant to be one repo = module as I mentioned above. Since our application contains all the webhooks, each as their own module it would mean breaking them out into separate repos. I tried to keep multiple published modules in one repo and was ready to throw out the whole code base due to how un-maintainable it was.

Option 2: The option I'm going to demonstrate here is to create a multi-module repo and use the magic word __replace__. This will tell Go where to look relative to the `go.mod` file to find a module locally.

Until this point we've ignored the `go.mod` files that we generated at the beginning. What these do is tell Go that you are building a module in that directory. Since we haven't added any external dependencies to any of our modules, the `go.mod` file has stayed empty, aside from the boilerplate. If we were to pull in say a Mongo database adapter by running the command

```bash
go get go.mongodb.org/mongo-driver/mongo
```
We would see that Go updates our `go.mod` file to look like what you see below
```go
module my_project/web_server

go 1.12

require (
	github.com/go-stack/stack v1.8.0 // indirect
	github.com/golang/snappy v0.0.1 // indirect
	github.com/xdg/scram v0.0.0-20180814205039-7eeb5667e42c // indirect
	github.com/xdg/stringprep v1.0.0 // indirect
	go.mongodb.org/mongo-driver v1.0.3 // indirect
	golang.org/x/crypto v0.0.0-20190701094942-4def268fd1a4 // indirect
	golang.org/x/sync v0.0.0-20190423024810-112230192c58 // indirect
	golang.org/x/text v0.3.2 // indirect
)
```

As you can see, Go pulled down the latest tagged version from Mongo's repo and added it with its dependencies to our file system. Then it added a record to the `go.mod` file for each of those dependencies. The `go.mod` file acts as a package manifest and keeps track of the dependencies for each module.

The missing piece of our puzzle is a feature that took me a while to find. We can add a statement in our `go.mod` file that will tell Go where to look for an imported module. This way it will look there rather than reaching out to Github or some other package registry.

We do this using the `replace` keyword in the `go.mod` file which tells Go to use the local module instead. Let's see this in action.

Currently our shared logger code is sitting under the directory `my_project/logger`. It's also important to note that this module is called `my_project/logger`. You can see this in the first line of the `go.mod` file which sets the module name.

If you try to import this directly into your web server now like this
```go
package main

import (
	"net/http"
	"my_project/logger" // import logger
)

func main() {
	l := new(logger.Logger) // create and use a new logger
	l.LogError("Not Found")
	http.Handle("/hello", http.HandlerFunc(handle))
	http.ListenAndServe(":5500", nil)
}

func handle(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("world"))
}
```

you'll receive the error
```bash
build my_project/web_server: cannot load my_project/logger: cannot find module providing package my_project/logger
```
This is because Go is reaching out looking for a url to pull down that logger code, which unless you've actually published it somewhere, doesn't exist.

BUT, if you add just one line in your `go.mod` file
```go
replace my_project/logger => ../logger
```
Go magically knows to look relative to the `go.mod` file for a module called
`my_project/logger` in the directory `../logger`.

Now running our code again logs out our silly `Not Found` error and starts up the server.

```bash
-----------------------------------------
[ERROR] - 2019-07-02 08:41:49.511755 -0700 PDT m=+0.000826403
Not Found
-----------------------------------------
```

Any changes you make to the logger module will be reflected when you restart the server. You can add the `replace` directive into any other module in this directory. You could do it in any directory but if you're linking to a local module from all over, then it's a sign that you need to make it a standalone module. Plus this won't work if your modules are in different repositories.

Getting back to our example, we're going to implement a middleware function that uses the logger to print out the url of each request. Then we'll prove that we can share this same code in a different package. Finally, we'll wrap things up with a quick recap of when you should and shouldn't use the `replace` directive.

## The Loggerware
Our final edit to the web server will be to add our loggerware function which should look like the below code,
```go
package main

import (
	"net/http"
	"my_project/logger"
)

func main() {
	l := new(logger.Logger)
	// wrap our hello handler function
	http.Handle("/hello", loggerware(l, http.HandlerFunc(handle)))
	http.ListenAndServe(":5500", nil)
}

func handle(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("world"))
}

// loggerware can wrap any handler function and will print out the datetime of the request
// as well as the path that the request was made to.
func loggerware(l *logger.Logger, next http.HandlerFunc) http.HandlerFunc {
	return http.HandlerFunc(func (w http.ResponseWriter, r *http.Request) {
		requestPath :=  r.URL
		l.LogInfo("Request Made To: ", requestPath)
	})
}
```

As you can see, we didn't do anything super complex here other than to integrate our logger into the request. In a more real world scenario we'd want to break our routes into groups and be able to apply the logger to all the endpoints using something like the gorilla mux package. Or in the case of something like a GraphQL server you could use this to wrap the GraphQL endpoint and create logs for every GraphQL request.

That should do it for the web server. If you run this code and open your browser to `localhost:5500/hello` you should see the response `world` come back as well as the oh so pretty log in the terminal that looks like,

```bash
-----------------------------------------
[INFO] - 2019-07-02 08:55:34.360145 -0700 PDT m=+17.505867800
Request Made To:
/hello
-----------------------------------------
```

## The Other Server
Just to prove that this is shared code and can be applied to any other server, I'm going to quickly build another server that uses the same logger but is a fully isolated service from our other one.

Moving now to your `web_server_two` folder place this code in a file called `main.go`. Remember to have an executable program you need a package `main` with a function `main`.

Here's the code for this web server.

```go
package main

import (
	"net/http"
	"my_project/logger"
)

func main() {
	l := new(logger.Logger)

	http.Handle("/valar-morghulis", loggerware(l, http.HandlerFunc(handle)))
	http.ListenAndServe(":5600", nil)
}

func handle(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("valar dohaeris"))
}

func loggerware(l *logger.Logger, next http.HandlerFunc) http.HandlerFunc {
	return http.HandlerFunc(func (w http.ResponseWriter, r *http.Request) {
		requestPath :=  r.URL
		l.LogInfo("Request Made To Server Two: ", requestPath)
	})
}
```
Again, not the most useful server but it proves that our logger code can be written once and reused through out our various micro-services as a shared but locally available library.

## The Conclusion
Wrapping things up here, let's remember what our goals were. 

1. Maintain a single repository containing all microservices or functions as isolated executables
1. Share common pieces of our Go code across all these applications as a shared module
1. Avoid having to maintain many different published modules with their own separate versioning 


As we saw, we were able to do this using something called a mutli-module repository. Using the replace directive in the go.mod file, we designated a local module relative to our shared module. Go then used our local module rather than reaching out to Github or some other registry.

Following this pattern we created a single logger library as an encapsulated module. We then were able to use it across two  separate, isolated web servers. We can apply this to any scenario where there is a main repository containing one or more modules that reuse the same code.

#### When not to use

> "For all but power users, you probably want to adopt the usual convention that one repo = one module. It's important for long-term evolution of code storage options that a repo can contain multiple modules, but it's almost certainly not something you want to do by default." 
--Russ Cox (Google developer and core Go contributor)

You should not use this method as a substitute for versioning and distributing modules.

As Russ Cox, core Go developer stated in the Go module docs, the most common use case will be one module = one repo. If there is a chance you will use your module in other projects then you should consider distributing it as a standalone module. In fact the local import will only work when the code is in  the same repository. 

This solution only works when you maintain a mono-repo and have no intention of pulling the shared logic into a separate repository.

The nice thing is because you're setting up the shared code as a true module, it's trivial to break it out into an isolated module. All that needs done is to remove the replace directive from any of the modules that use the shared code. Then import the package from Github or some other registry. Because it's so easy, this approach is future proof for any large changes you might make to the code architecture.
[sc name="post-signature"]
