# Tollbooth

This is a fork of the excellent [tollbooth by didip](https://github.com/didip/tollbooth). The changes are:

- Added the option to add a func to handle requests whenever the limit reached instead of responding with a generic error.
- Removed thirdparty section.

The new function added is `tollbooth.LimitFuncHandler2`:
```go
func main() {
    http.Handle(
        "/",
        tollbooth.LimitFuncHandler2(
            tollbooth.NewLimiter(1, time.Second),
            HelloHandler,
            ErrorHandler,
        ),
    )
    http.ListenAndServe(":12345", nil)
}

func ErrorHandler(w http.ResponseWriter, r *http.Request, err *errors.HTTPError) {
     // Handle error.
}

// The HTTPError looks like this.
type HTTPError struct {
    Message    string
    StatusCode int
}
```

Everything below is from the original README.

---

## Tollbooth

This is a generic middleware to rate-limit HTTP requests.

**NOTE:** This library is considered finished, any new activities are probably centered around `thirdparty` modules.


## Five Minutes Tutorial
```go
package main

import (
    "github.com/didip/tollbooth"
    "net/http"
    "time"
)

func HelloHandler(w http.ResponseWriter, req *http.Request) {
    w.Write([]byte("Hello, World!"))
}

func main() {
    // Create a request limiter per handler.
    http.Handle("/", tollbooth.LimitFuncHandler(tollbooth.NewLimiter(1, time.Second), HelloHandler))
    http.ListenAndServe(":12345", nil)
}
```

## Features

1. Rate-limit by request's remote IP, path, methods, custom headers, & basic auth usernames.
    ```go
    limiter := tollbooth.NewLimiter(1, time.Second)

    // Configure list of places to look for IP address.
    // By default it's: "RemoteAddr", "X-Forwarded-For", "X-Real-IP"
    // If your application is behind a proxy, set "X-Forwarded-For" first.
    limiter.IPLookups = []string{"RemoteAddr", "X-Forwarded-For", "X-Real-IP"}

    // Limit only GET and POST requests.
    limiter.Methods = []string{"GET", "POST"}

    // Limit request headers containing certain values.
    // Typically, you prefetched these values from the database.
    limiter.Headers = make(map[string][]string)
    limiter.Headers["X-Access-Token"] = []string{"abc123", "xyz098"}

    // Limit based on basic auth usernames.
    // Typically, you prefetched these values from the database.
    limiter.BasicAuthUsers = []string{"bob", "joe", "didip"}
    ```

2. Each request handler can be rate-limited individually.

3. Compose your own middleware by using `LimitByKeys()`.

4. Tollbooth does not require external storage since it uses an algorithm called [Token Bucket](http://en.wikipedia.org/wiki/Token_bucket) [(Go library: golang.org/x/time/rate)](//godoc.org/golang.org/x/time/rate).

