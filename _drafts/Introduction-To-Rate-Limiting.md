---
layout: post
title:  Introduction to Rate Limiting
---

  Have you ever wondered how some websites protect themselves from [DDoS attacks](https://en.wikipedia.org/wiki/Denial-of-service_attack) or why we get status code [HTTP 429](https://tools.ietf.org/html/rfc6585#section-4) in response to calling some API (Ex: [GitHubAPI](https://developer.github.com/v3/)) too many times. The solution to this is Rate limiting.

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
