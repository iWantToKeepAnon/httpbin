# httpbin(1): HTTP Request & Response Service


A [Kenneth Reitz](http://kennethreitz.org/bitcoin) Project.

![ice cream](http://farm1.staticflickr.com/572/32514669683_4daf2ab7bc_k_d.jpg)

Run locally:
```sh
docker pull kennethreitz/httpbin
docker run -p 80:80 kennethreitz/httpbin
```

See http://httpbin.org for more information.

## Officially Deployed at:

- http://httpbin.org
- https://httpbin.org
- https://hub.docker.com/r/kennethreitz/httpbin/


## SEE ALSO

- http://requestb.in
- http://python-requests.org
- https://grpcb.in/

## Build Status

[![Build Status](https://travis-ci.org/requests/httpbin.svg?branch=master)](https://travis-ci.org/requests/httpbin)

---

## Python 3.14 / Modern Dependency Compatibility Notes

The following changes were required to run this project on Python 3.14 with
current versions of Flask, Werkzeug, and related packages.

### Docker Build

- Command `docker build .`
- To name/tag the image: `docker build -t httpbin:local .`
- Building from `python:3.14-slim` chooses your architecture. I.e. building on an ARM machine will
  build and ARM image; where the docker hub has AMD64 only (as "official" anyway).

### Pipfile / Pipfile.lock

- The `[[source]]` block in `Pipfile` must include a `name` field (required by
  pipenv 2026.x): `name = "pypi"`.
- `brotlipy` was replaced with `brotli` — `brotlipy` pulls in `cffi` which
  requires a C compiler not available in the Alpine base image. `brotli` has
  pre-built wheels and is the modern replacement. `filters.py` already imports
  `brotli` directly.
- `meinheld` was removed from dependencies — it has no Python 3.14 wheels and
  has not been maintained since ~2020. `gunicorn` alone is sufficient.
- `pipenv lock -r` was removed in pipenv 2026.x. Use `pipenv requirements`
  instead to export a `requirements.txt`.
- Deleted and regenerated `Pipfile.lock` to resolve stale pinned versions
  (`gevent==1.3.4`, `cffi==1.11.5`, etc.) incompatible with Python 3.14.

### Werkzeug 3.x Import Fixes (`core.py`, `helpers.py`)

- `from werkzeug.wrappers import BaseResponse` → `from werkzeug.wrappers import Response as BaseResponse`
- `BaseResponse.autocorrect_location_header = False` removed — this attribute
  no longer exists in Werkzeug 2.x+; location header casing is preserved by default.
- `from werkzeug.http import parse_authorization_header` — removed in Werkzeug 3.x.
  Replaced with a compatibility shim using `Authorization.from_header()`.
- `WWWAuthenticate.set_digest()` removed in Werkzeug 3.x. Replaced with
  direct field assignment (`auth["realm"]`, `auth["nonce"]`, etc.).
- `response.headers["Location"] = args["url"].encode("utf-8")` — setting the
  Location header as bytes causes it to be rendered as `b'/path'`. Changed to
  pass the string directly.

### Python 3.x Syntax Fixes (`helpers.py`)

- `ASCII_ART` string changed to a raw string (`r"""..."""`) to avoid
  `SyntaxWarning` for invalid escape sequences (`\_;`, `\"\"\"`).
- Regex pattern `'\s*...'` changed to `r'\s*...'` for the same reason.

### Test Fixes (`test_httpbin.py`)

- Removed `Content-Length: 0` assertions from `test_get` and `test_anything`.
  Newer Werkzeug/Flask no longer injects `Content-Length` on bodyless GET requests.
- `test_base64`: test client `.get()` requires a string path, not bytes
  (`b'/base64/...'`). Pass a decoded string instead.
