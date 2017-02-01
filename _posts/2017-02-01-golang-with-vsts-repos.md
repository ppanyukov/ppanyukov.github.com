---
layout: post
title: "golang with visualstudio.com git repos"
excerpt: "How to get golang tooling to work with git repos hosted by visualstudio.com"
---

# The opening

By default golang does not work with git repos hosted by visualstudio.com.

This is due to:

1. Authentication is required

2. golang is not aware of visualstudio.com like it is of github.

3. URL paths to repos in visualstudio.com are hostile to golang.
   
   In particular, the fact that the repo URLs contain `_git` makes them
   unusable from go even if you overcome the other issues.


# How things should work

##  Git repos at visualstudio.com

Setup:

- You are a user of visualstudio.com.

- Your git repositories are hosted by ACCOUNT.visualstudio.com

- The path to a repository looks like this:

 `https://ACCOUNT.visualstudio.com/COLLECTION/PROJECT/_git/REPO`

- The repositories require authentication.


## golang tooling

We would like to use standard golang tools with our repos. For example:

```
go get https://ACCOUNT.visualstudio.com/COLLECTION/PROJECT/_git/REPO_A
```

We expect this to fetch and things into:

```
$GOPATH/src/ACCOUNT.visualstudio.com/COLLECTION/PROJECT/_git/REPO_A
```

We would like to list packages there and install things like so:

```
 # this should work
go list ACCOUNT.visualstudio.com/...
```

```
 # this should work too
 go install ACCOUNT.visualstudio.com/...
```


## golang programming

In our code, we expect we can reference the packages from visualstudio.com:

```
// This should in theory work
import (
  "ACCOUNT.visualstudio.com/COLLECTION/PROJECT/_git/REPO_A/SOME_PACKAGE"
)

// And using the package should work too
SOME_PACKAGE.someFunction()
```


# Problem

None of the above expected behaviour works.

## Authentication

This can be solved by adding these lines to global gitconfig:

```
[url "https://USER:ACCESS_TOKEN@ACCOUNT.visualstudio.com/"]
      insteadOf = https://ACCOUNT.visualstudio.com/
```

Now you can use git commands without password and all is good.

```
git clone https://ACCOUNT.visualstudio.com/COLLECTION/PROJECT/_git/REPO
```


## Using `go get`

This still does not work after authentication fix because `go get` does not know 
the URL is a git repo! We get mysterious messages about import paths etc.

How do we tell `go` tools it's a git repo? One way is to add `.git` to the end 
of the URL.

```
go get https://ACCOUNT.visualstudio.com/COLLECTION/PROJECT/_git/REPO.git
```

This does seem to work initially, and we indeed have things in 
`$GOPATH/src` as expected. Except there are other issues.



## Using `go list` and programming

The real problem appears when we try to list packages:

```
 go list ACCOUNT.visualstudio.com/...
```

We get an error that nothing is found.

Nothing works when we also try to reference the package in the source code:

```
// Does not work!
import (
  "ACCOUNT.visualstudio.com/COLLECTION/PROJECT/_git/REPO_A.git/SOME_PACKAGE"
)
```


## That `_git` part is the problem

Why?? Turns out `golang` ignores everything that start with a dot `.` 
and underscore `_`.

How unlucky that our visualstudio.com URLs have `_git` in them!


## Other annoyances

The URL for visualstudio.com is long and ugly. The need to use `.git` at the end 
is counter-intuitive and also can potentially cause issues.


# Solution

## The `<meta>` tag

There is a mysterious mention at https://golang.org/cmd/go/:

> If the import path is not a known code hosting site and also lacks a version 
> control qualifier, the go tool attempts to fetch the import over https/http 
> and looks for a <meta> tag in the document's HTML <head>.
>
> The meta tag has the form:
>
> `<meta name="go-import" content="import-prefix vcs repo-root">`

Interesting!

Let's use some curl:

```
curl -i http://golang.org/x/tools
```

This gives us:

```
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
X-Cloud-Trace-Context: 6304c66d6794084c44551614bfd9ac7a
Date: Wed, 01 Feb 2017 11:04:09 GMT
Server: Google Frontend
Content-Length: 588

<!DOCTYPE html>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
<meta name="go-import" content="golang.org/x/tools git https://go.googlesource.com/tools">
<meta name="go-source" content="golang.org/x/tools https://github.com/golang/tools/ https://github.com/golang/tools/tree/master{/dir} https://github.com/golang/tools/blob/master{/dir}/{file}#L{line}">
<meta http-equiv="refresh" content="0; url=https://godoc.org/golang.org/x/tools">
</head>
<body>
Nothing to see here; <a href="https://godoc.org/golang.org/x/tools">move along</a>.
</body>
</html>
```

And hereby lies the solution.


## How we fix it

The solution is to have a server which will respond with HTML like above and point 
it to the visualstudio.com URL.

This resolves two problems:

- tells golang that our repo is a git repo
- removes the `_git` which prevents us to use packages

This does not solve authentication issues as such but see the `gitconfig` workaround.

Imagine we host this web site at `vsts.golang.io`

Then we could use these URLs to reference our visualstudio.com repos:

```
# this will fetch from ACCOUNT.visualstudio.com/COLLECTION/PROJECT/_git/REPO
go get vsts.golang.io/ACCOUNT/COLLECTION/PROJECT/REPO
go get vsts.golang.io/ACCOUNT/COLLECTION/PROJECT/REPO/PACKAGE
```

In our code, the import path would look like this:

```
// This will work
import (
  "vsts.golang.io/ACCOUNT/COLLECTION/PROJECT/REPO/PACKAGE"
)
```

## Example responses from our web server

```
curl localhost.git/microsoft/DefaultCollection/Personal/philip-test-goapp

<!DOCTYPE html>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
<meta name="go-import" content="localhost.git/microsoft/DefaultCollection/Personal/philip-test-goapp git https://microsoft.visualstudio.com/DefaultCollection/Personal/_git/philip-test-goapp">
</head>
<body>
Nothing to see here!
</body>
</html>
```

```
curl localhost.git/oracle/DefaultCollection/Personal/philip-test-goapp/some_package

<!DOCTYPE html>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
<meta name="go-import" content="localhost.git/oracle/DefaultCollection/Personal/philip-test-goapp/some_package git https://oracle.visualstudio.com/DefaultCollection/Personal/_git/philip-test-goapp">
</head>
<body>
Nothing to see here!
</body>
</html>
```

# One last thing

For completely transparent go tooling the web app must support HTTPS.

# The implementation

See <https://gist.github.com/ppanyukov/d3fbc5d0e85569146a264a994d032b0d> for
sample implementation in `go`.



