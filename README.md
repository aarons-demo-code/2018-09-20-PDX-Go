# Welcome, Athenians! :smiley:

I'm so glad you're here. Let's talk about [Project Athens](https://github.com/gomods/athens) today!

This repository has demos and code that I used in a talk about Athens at
[the September 20, 2018 PDX Go meetup](https://www.meetup.com/PDX-Go/events/251030520/).

*These instructions assume that you have a Go 1.11 toolchain installed
and that you're running on a Bash shell. You definitely need Go 1.11, but
you can choose to run on other shells (Zsh, Powershell, etc...) - just
make sure to adapt the commands in here to your shell of choice.*

# About The Demo

There's a lot going on in the background to make this demo simple as pie
:sunglasses:!

We're going to build a web application in here, but the actual web app
isn't the exciting part here -- it'll be super simple, actually.

Even the tools that we use to build the application won't be exciting --
we'll be using the standard `go get` toolchain and the
[echo](https://github.com/labstack/echo) web framework.

It's what's going on under the covers that's really, really cool :)

So, let's get started!

# About The App

The application code is in this repository, in [main.go](./main.go).
If you look closely enough, you'll notice that I copy-pasted it right from the 
[Echo README](https://github.com/labstack/echo/tree/a2d4cb9c7a629e2ee21861501690741d2374de10#example) (I made a tiny little change though, just for you #PDXGophers!).

Like I said, nothing remarkable, right?

But, notice what's not in here:

1. There's no `vendor` directory
2. There's no `glide.yaml` or `Gopkg.yaml` files, `Godeps` directory, or 
anything else that our dependency management tools use
3. There's a _new_ file called `go.mod` in it!

# The Dependencies

But our app depends on packages, and we're missing them, our editor even 
told us so with red squiggly lines!

![missing deps](/images/missing-imports.png)

That's right, we still have two dependencies:

- `github.com/labstack/echo`
- `github.com/labstack/echo/middleware`

We'd be crazy to try and build another HTTP router, right?

So, how do we pull them into our app so we can use them?

# Go Modules + Athens to the Rescue!

Let's use the power of [Go Modules](https://cda.ms/FN) and 
[Athens](https://cda.ms/FW) to get these dependencies, but do it _without_

1. Other dependency management tools
2. A `vendor/` directory

It's simple! So let's get to the demo!

## Get the Code, With a Twist :wink:

First, let's get the code. Clone this repository to somewhere on your computer,
but make sure it's outside your `GOPATH`:

```console
cd my/dir
git clone https://github.com/aarons-demo-code/2018-09-20-PDX-Go.git
cd 2018-09-20-PDX-Go
```

### I Got A Fever...

And the only cure is _no `GOPATH`_! 

Ok that one was pretty lame, but we can build our apps outside of the 
`GOPATH` now. We're gonna show that in action right now :tada:!

## Set up Your Environment

Next, let's set up our environment to turn on the Go Modules flag. In Go
1.11, the older GOPATH behavior is turned on by default. Let's turn on Go
Modules by setting an environment variable:

```console
export GO111MODULE=on
```

(by the way, this environment variable allows us to run everything outside 
of the `GOPATH`)

Next, let's point Go modules to use Athens to fetch code from:

```console
export GOPROXY=https://microsoftgoproxy.azurewebsites.net
```

_NOTE: This is an experimental Athens server and could change or disappear any time! It's easy to run your own, though - see [this documentation](https://docs.gomods.io/install/shared-team-instance/) for how to do it._

>This `GOPROXY` environment variable is telling Go modules to download all our code from Athens, and _not_ GitHub!

## Clear Your Caches :boom:

Almost done - let's make sure that our local Go modules cache is empty. This
cache is located in your `GOPATH` under `${GOPATH}/pkg/mod` and is used
for all Go module based builds on our machine.

Let's delete the cache, _and_ since we know we're going to be fetching the
`github.com/labstack/echo` package, let's delete that from our `GOPATH` too:

```console
rm -r ${GOPATH}/pkg/mod # you might need to 'sudo' this!
rm -r ${GOPATH}/src/github.com/labstack/echo
```

## Build The App! :shipit:

And finally, let's go do our build (this one might take a little while while 
`go` downloads code from Athens):

```
go build
```

# What Happened? :scream:

Do you notice some lines of output that look like this?

```console
...
go: finding github.com/valyala/fasttemplate latest
...
```

That's Go modules downloading code from Athens :smile:. For each dependency, Go
modules asks the Athens server:

- What's the latest version?
- Does it have a `go.mod` file (i.e. is it a Go module too?)
- Give me some other metadata about it (i.e. `created_date`)
- Give me a _zip file_ with the source code of the module

>That last part is important! We're not downloading entire Git trees here - just the source code at a particular version. Hooray!!

## There's a New `go.sum` File?

After your build is done, you'll see a new `go.sum` file that looks
kinda like this:

```console
github.com/dgrijalva/jwt-go v3.2.0+incompatible h1:7qlOGliEKZXTDg6OTjfoBKDXWrumCAMpl/TFQ4/5kLM=
github.com/dgrijalva/jwt-go v3.2.0+incompatible/go.mod h1:E3ru+11k8xSBh+hMPgOLZmtrrCbhqsmaPHjLKYnJCaQ=
github.com/labstack/echo v3.2.1+incompatible h1:J2M7YArHx4gi8p/3fDw8tX19SXhBCoRpviyAZSN3I88=
github.com/labstack/echo v3.2.1+incompatible/go.mod h1:0INS7j/VjnFxD4E2wkz67b8cVwCLbBmJyDaka6Cmk1s=
github.com/labstack/gommon v0.2.7 h1:2qOPq/twXDrQ6ooBGrn3mrmVOC+biLlatwgIu8lbzRM=
github.com/labstack/gommon v0.2.7/go.mod h1:/tj9csK2iPSBvn+3NLM9e52usepMtrd5ilFYA+wQNJ4=
github.com/mattn/go-colorable v0.0.9 h1:UVL0vNpWh04HeJXV0KLcaT7r06gOH2l4OW6ddYRUIY4=
github.com/mattn/go-colorable v0.0.9/go.mod h1:9vuHe8Xs5qXnSaW/c/ABM9alt+Vo+STaOChaDxuIBZU=
github.com/mattn/go-isatty v0.0.4 h1:bnP0vzxcAdeI1zdubAl5PjU6zsERjGZb7raWodagDYs=
github.com/mattn/go-isatty v0.0.4/go.mod h1:M+lRXTBqGeGNdLjl/ufCoiOlB5xdOkqRJdNxMWT7Zi4=
github.com/valyala/bytebufferpool v1.0.0 h1:GqA5TC/0021Y/b9FG4Oi9Mr3q7XYx6KllzawFIhcdPw=
github.com/valyala/bytebufferpool v1.0.0/go.mod h1:6bBcMArwyJ5K/AmCkWv1jt77kVWyCJ6HpOuEn7z0Csc=
github.com/valyala/fasttemplate v0.0.0-20170224212429-dcecefd839c4 h1:gKMu1Bf6QINDnvyZuTaACm9ofY+PRh+5vFz4oxBZeF8=
github.com/valyala/fasttemplate v0.0.0-20170224212429-dcecefd839c4/go.mod h1:50wTf68f99/Zt14pr046Tgt3Lp2vLyFZKzbFXTOabXw=
golang.org/x/crypto v0.0.0-20180910181607-0e37d006457b h1:2b9XGzhjiYsYPnKXoEfL7klWZQIt8IfyRCz62gCqqlQ=
golang.org/x/crypto v0.0.0-20180910181607-0e37d006457b/go.mod h1:6SG95UA2DQfeDnfUPMdvaQW0Q7yPrPDi9nlGo2tz2b4=
```

This is the "lockfile" - it describes _all_ the dependencies that our
app needs. Even the dependencies of the dependencies (and dependencies of dependencies of dependencies of ...) -- also called the transitive dependencies.

Commonly, you should check in this `go.sum` file, but we're not going
to check it in here so that the demo looks good :wink:.

## Oh! The `go.mod` Got Updated!

Also notice that there are some more lines in the `go.mod` file now!

```console
module github.com/aarons-demo-code/2018-09-20-PDX-Go

require (
	github.com/dgrijalva/jwt-go v3.2.0+incompatible // indirect
	github.com/labstack/echo v3.2.1+incompatible
	github.com/labstack/gommon v0.2.7 // indirect
	github.com/mattn/go-colorable v0.0.9 // indirect
	github.com/mattn/go-isatty v0.0.4 // indirect
	github.com/valyala/bytebufferpool v1.0.0 // indirect
	github.com/valyala/fasttemplate v0.0.0-20170224212429-dcecefd839c4 // indirect
	golang.org/x/crypto v0.0.0-20180910181607-0e37d006457b // indirect
)
```

When we ran `go build`, the tooling detected our imports, downloaded them from
Athens, and then wrote our imports into the `require` clause in the file.

At this point, our app is built and ready to run 
(which you can do by running `./2018-09-20-PDX-Go` - great app name, right?). 
But it's also ready to check into GitHub, so that the rest of our team, our CI,
etc... can build and run it.

# But Shouldn't We Check In `vendor`? :worried:

We still can, but let's not!

Previously, we checked vendor so that we could keep all our dependencies
under our own control. That's really great for some projects, but for many
it isn't, and here's why.

The underlying reason that a lot of us check in vendor is so that our builds
are _dependable_. That is, if someone changes or deletes their GitHub 
repository, our build won't break because we have a copy of the source code
checked in alongside our app.

Now, Athens stores the copy of our dependency for _everyone_, and Athens 
guarantees that the code never changes - even if it changes on GitHub!

In lots of apps, we can remove the vendor directory, save space and 
build times, and make lives easier for the folks who depend on us (because 
then _they_ don't have to download your vendor directory).

