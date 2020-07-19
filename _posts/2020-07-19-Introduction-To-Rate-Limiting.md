---
layout: post
title:  Introduction to Rate Limiting
---

  Have you ever wondered how some websites protect themselves from [DDoS attacks](https://en.wikipedia.org/wiki/Denial-of-service_attack) or why we get status code [HTTP 429](https://tools.ietf.org/html/rfc6585#section-4) in response to calling some API (Ex: [GitHubAPI](https://developer.github.com/v3/)) too many times. The solution behind them is Rate limiting.

## What is Rate Limiting?
   Rate limiting is used to control the amount of incoming traffic to a service or an API. For example, an API limits 100 requests per user in an hour using a Rate Limiter. If you send more than 100 requests then you will recieve [TooManyRequests](https://tools.ietf.org/html/rfc6585#section-4) error. This way it can control incoming traffic as well as can mitigate the DDoS attacks.

## Types of Rate Limiting
1. **Server level Rate Limiting**
  - Server level Rate Limiting can be done using Nginx server. 
  - By using Nginx server for running our service, we can control the traffic based on incoming URL or IP address. 
  - You can learn more about it [here](https://www.freecodecamp.org/news/nginx-rate-limiting-in-a-nutshell-128fe9e0126c/).

2. **API level Rate Limiting**
  - API level Rate Limiting can be achieved by writing a middleware on top of API end points.
  - Every request will be inspected by middleware and it will decide whether the request should be processed or rejected.
  - If the request is rejected by middleware, API sends response with status code [HTTP 429](https://tools.ietf.org/html/rfc6585#section-4).
   
In this post, we will be discussing more about API level Rate limiting.

## API level Rate Limiting
  Lets understand API level Rate Limiting by implementing simple Rate Limiter which accepts one request per sec as middleware on an API end point. 
  {% highlight golang %}
  // main.go
  package main
  import (
    "fmt"
    "net/http"
  )

  func main() {
    mux := http.NewServeMux()
    mux.HandleFunc("/hello", ratelimit(helloHandler))
    fmt.Println("Server listening at 8888:...")
    http.ListenAndServe(":8888", mux)
  }

  func helloHandler(w http.ResponseWriter, r *http.Request) {
    w.Write([]byte("Hello World!\n"))
  }
  {% endhighlight %}

  Above code snippet listens at end point `localhost:8888/hello` . We are limiting the number of incoming calls to the endpoint by calling `ratelimit` method before calling `helloHandler`. This `ratelimit` method decides whether the incoming request should be processed or rejected. 
  Lets implement the `ratelimit` using package [x/time/rate](golang.org/x/time/rate). This package internally implements [Token bucket](https://en.wikipedia.org/wiki/Token_bucket) algorithm which we will discuss in the next section.
  {% highlight golang %}
  // RateLimiter.go
  package main
  import (
    "net/http"
    "golang.org/x/time/rate"
  )

  var limiter *rate.Limiter = rate.NewLimiter(1, 1)

  func ratelimit(next http.HandlerFunc) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
      if !limiter.Allow() {
        http.Error(w, "Limit on Number requests has Exceeded", http.StatusTooManyRequests)
        return
      }
      next.ServeHTTP(w, r)
    }
  }
  {% endhighlight %}

  In the above code, we first created a global instance of Limiter by calling NewLimiter with rate and burst equal to one. The `ratelimit` method accepts `helloHandler` as parameter and returns a new `http.HandlerFunc`. New handler first checks whether the request should be processed or not. If not, it will respond with status code `http.StatusTooManyRequests` else it proceeds.

## How Token Bucket Algorithm works?
  - Lets define a struct `TokenBucket` with `Rate`, `Burst` and `Available` as parameters.
    {% highlight golang %}
      /* Burst is the maximum number of tokens can be present in bucket
         Rate is the number of tokens to be added per sec to the bucket
         Available is the number of tokens currently available in the bucket*/
      type TokenBucket struct {
        Rate      int
        Burst     int
        Available int
        mu        sync.RWMutex
      }
    {% endhighlight %}
  - `Fill` method fills the bucket with `Rate` tokens every second
      {% highlight golang %}
        func (tokenBucket *TokenBucket) Fill() {
          ticker := time.NewTicker(time.Second)
          for range ticker.C {
            tokenBucket.mu.Lock()
            defer tokenBucket.mu.Unlock()
            tokenBucket.Available = max(tokenBucket.Burst, tokenBucket.Available+tokenBucket.Rate)
          }
        }
      {% endhighlight %}
  - `Take` method reduces the available tokens by one and returns true if there are any available tokens
    {% highlight golang %}
      func (tokenBucket *TokenBucket) Take() bool {
        tokenBucket.mu.Lock()
        defer tokenBucket.mu.Unlock()
        if tokenBucket.Available > 0 {
          tokenBucket.Available--
          return true
        }
        return false
      }
    {% endhighlight %}
  Complete implementation for Token Bucket Algorithm is available [here](https://github.com/leninkumar31/GoTutorials/blob/master/TokenBucket/TokenBucket.go)
## More Usecases
  - We have discussed the usecase which limits number of requests globally. What if we want to limit number of requests per user?
    - Hint: In Memory Cache and Invalidating the old entries
  - How would you scale the above usecase When there are millions of users?
    - Hint: External cache like Redis or Memcached

## Conclusion
   In this we dicussed about what is rate limiting, how to apply rate limiting to your APIs and how token bucket algorithm works.

## Further Reading
  - [Very Good explanation on Token Bucket Algorithm](https://github.com/vladimir-bukhtoyarov/bucket4j/blob/master/doc-pages/token-bucket-brief-overview.md)
  - [Other RateLimiting Algorithms in GoLang](https://medium.com/@justin.graber/rate-limiting-in-golang-f3ed2c62df36)
  - [Scaling your API with rate limiters](https://stripe.com/blog/rate-limiters)