# ring-cors [![Build Status][1]][2]
simple ring middleware for easy cross-origin resource sharing

[](dependency)
```clojure
[jumblerg/ring-cors "2.0.0"] ;; latest release
```
[](/dependency)

## rationale
the promise of black-box abstraction, realized here in the form of a ring handler, makes it unnecessesary for the consumer to be concerned with the intricacies of http.  specific to the case of cross-origin requests, there's little reason to expose the many options and behaviors enumerated by the w3c to an application developed with an awareness of cors from the outset; the vast majority of these settings exist to provide an upgrade path to servers that predate the specification.  this library consequently abstracts away these details, assuming that it will be used with a modern, cors-aware service in order to expose the most straightforward api possible.

## application
simply pass the wrapper some regular expressions or a predicate that accepts the request's origin.

```clojure
(require '[jumblerg.middleware.cors :refer [wrap-cors]])

;; accept everything
(wrap-cors routes #".*")
(wrap-cors routes identity)

;; accept some things
(wrap-cors routes #".*localhost.*" #".*mydomain.org$")
(wrap-cors routes #(= (:allowed-origin db) %))

;; accept one thing
(wrap-cors routes #"^http://myapp.mydomain.org$")

;; accept nothing
(wrap-cors routes)
```
notice that the regular expressions are anchored in order to prevent partial matches from being accepted.

## considerations
complexities such as the cors preflight [exist to ensure that legacy servers predating][5] the [cors specification][6] (and, for example, never anticipated a cross-domain DELETE) can upgrade to a new world where browsers permit cross-domain requests in a backwards-compatible way.  the preflight allows these legacy servers to selectively expose individual methods and headers to cross-origin requests without having to support all of them.  since this scenario doesn't apply to a newly-developed service, this wrapper permissively authorizes the user agent of an allowed origin to use all requested methods, headers, and cookies.

this middleware instructs the browser to cache the preflight for 24 hours based on a w3c discussion of the subject.  mozilla has adapted 24 hours as the max cache time for the preflight, while webkit will max out in just five minutes (some chrome documentation states the default is 1/2 hour).  the cache can be cleared manually from the browser if necessary during development.  in chrome, this is achieved by setting the "disable cache while devtools is open" flag in the development tools settings panel.  [sometimes maybe.][7]

this middleware works with all desktop and mobile browsers except the usual suspects: internet explorers 8 and 9 ([full cors support was added by "the fourth platform" of ie 10][8]). more information can be found regarding ie's partial cors support at [caniuse][11] and [microsoft's website][12].

## license
copyright (c) jumblerg. all rights reserved.

distributed with clojure under the eclipse public license

[1]: https://travis-ci.org/jumblerg/ring-cors.png?branch=master
[2]: https://travis-ci.org/jumblerg/ring-cors
[5]: http://stackoverflow.com/questions/15381105/cors-what-is-the-motivation-behind-introducing-preflight-requests
[6]: http://www.w3.org/TR/cors/
[7]: https://developers.google.com/storage/docs/cross-origin
[8]: http://blogs.msdn.com/b/ie/archive/2012/02/09/cors-for-xhr-in-ie10.aspx
[11]: http://caniuse.com/cors
[12]: http://blogs.msdn.com/b/ieinternals/archive/2010/05/13/xdomainrequest-restrictions-limitations-and-workarounds.aspx
