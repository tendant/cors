# CORS net/http middleware

This is a fork of [go-chi/cors](https://github.com/go-chi/cors) (which itself is a fork of [github.com/rs/cors](https://github.com/rs/cors)) that
provides a `net/http` compatible middleware for performing preflight CORS checks on the server side. These headers
are required for using the browser native [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API).

This middleware is designed to be used as a top-level middleware on the [chi](https://github.com/go-chi/chi) router.
Applying with within a `r.Group()` or using `With()` will not work without routes matching `OPTIONS` added.

## Fork Changes

This fork (`github.com/tendant/cors`) includes the following changes from the upstream go-chi/cors:

- **Module name**: Changed from `github.com/go-chi/cors` to `github.com/tendant/cors` to support local development with Go module replace directives
- **Credentials support**: Properly sets `Access-Control-Allow-Credentials: true` header when `AllowCredentials: true` is configured
- **Debug logging improvements**: Enhanced debug output for troubleshooting CORS issues

## Install

For use with Go modules replace directive (local development):

```go
// go.mod
require github.com/tendant/cors v0.0.0-20250701140642-3a5381283113

replace github.com/tendant/cors => /path/to/local/cors
```

Or install directly:

`go get github.com/tendant/cors`

## Usage

### Basic CORS

```go
package main

import (
  "net/http"
  "github.com/go-chi/chi/v5"
  "github.com/tendant/cors"
)

func main() {
  r := chi.NewRouter()

  // Basic CORS
  // for more ideas, see: https://developer.github.com/v3/#cross-origin-resource-sharing
  r.Use(cors.Handler(cors.Options{
    // AllowedOrigins:   []string{"https://foo.com"}, // Use this to allow specific origin hosts
    AllowedOrigins:   []string{"https://*", "http://*"},
    // AllowOriginFunc:  func(r *http.Request, origin string) bool { return true },
    AllowedMethods:   []string{"GET", "POST", "PUT", "DELETE", "OPTIONS"},
    AllowedHeaders:   []string{"Accept", "Authorization", "Content-Type", "X-CSRF-Token"},
    ExposedHeaders:   []string{"Link"},
    AllowCredentials: false,
    MaxAge:           300, // Maximum value not ignored by any of major browsers
  }))

  r.Get("/", func(w http.ResponseWriter, r *http.Request) {
    w.Write([]byte("welcome"))
  })

  http.ListenAndServe(":3000", r)
}
```

### CORS with Credentials (for cookie-based authentication)

```go
package main

import (
  "net/http"
  "github.com/go-chi/chi/v5"
  "github.com/tendant/cors"
)

func main() {
  r := chi.NewRouter()

  // CORS with credentials support for HTTP-only cookies
  r.Use(cors.Handler(cors.Options{
    AllowedOrigins:   []string{"http://localhost:5173", "http://localhost:3000"},
    AllowedMethods:   []string{"GET", "POST", "PUT", "DELETE", "OPTIONS"},
    AllowedHeaders:   []string{"Accept", "Content-Type", "Authorization"},
    ExposedHeaders:   []string{"Link"},
    AllowCredentials: true,  // CRITICAL: Required for HTTP-only cookies
    MaxAge:           300,
    Debug:            true,  // Enable debug logging
  }))

  r.Get("/", func(w http.ResponseWriter, r *http.Request) {
    w.Write([]byte("welcome"))
  })

  http.ListenAndServe(":4000", r)
}
```

**Important**: When using `AllowCredentials: true`, you must specify exact origins in `AllowedOrigins`. Wildcards like `"https://*"` or `"*"` are not allowed by browsers when credentials are included.

## Credits

All credit for the original work of this middleware goes out to [github.com/rs](https://github.com/rs).
