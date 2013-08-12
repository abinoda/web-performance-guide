# Web Performance

The goal of this article is to provide an introductory overview of web performance.

## What is performance? Why do we optimize?

It's important to understand that "performance" can refer to both **end-user performance** (the amount of time it takes to load a website), as well as the **speed, complexity, and resource usage of the server**. 

Performance has a direct effect on a business' bottom line. The correlation between web performance and business metrics -- whether it be user experience, conversions and engagement, bounce rates, search engine ranking -- has been well-proven with data and research studies. Some examples include [Google and Bing's analysis of performance and user impact](https://dl.dropboxusercontent.com/u/598519/web-performance-guide-assets/The%20User%20and%20Business%20Impact%20of%20Server%20Delays%2C%20Additional%20Bytes%2C%20and%20HTTP%20Chunking%20in%20Web%20Search%20Presentation.pptx), [a presentation by Shopzilla.com's VP of Enginnering](https://dl.dropboxusercontent.com/u/598519/web-performance-guide-assets/Shopzilla%27s%20Site%20Redo%20-%20You%20Get%20What%20You%20Measure%20Presentation.ppt), and [a study by the performance analytics company Torbit](https://dl.dropboxusercontent.com/u/598519/web-performance-guide-assets/sept-2012-rum-talk-120910130322-phpapp01.key).

Performance optimization is therefore driven by business value. And, since business value is primarily derived from good user experience, **performance optimization is about improving user experience**. Optimizations non-related to user experience such as refactoring code or lowering server costs are generally a lesser priority.

## What should I optimize?
Delivering a fast and optimized user experience in the browser requires careful thinking across many layers of the stack - TCP and up. An HTTP request is initiated by the client, data is transmitted over the internet, and eventually arrives at the application server…. the application server may then communicate with various services (eg. database, APIs, memcached) before finally producing a response that is sent back to the client.

Every layer and segment of the request/resonse cycle presents opportunities for performance optimization. Thus the biggest challenge in optimization isn't knowing *how* to optimize, but rather identifying and deciding *when and what* to optimize. In the real world, the answers to those questions depend entirely on the circumstances (eg. developers, codebase, deadlines, budget, architecture).

We generally refer to two separate areas of performance optimziation -- client and server. **Front-end performance** refers to how efficiently the browser fetches data from the server and renders it. **Back-end performance** is about how well the server can generate and serve responses.

It is worth noting that although we refer to areas of optimization distinctly, in terms of development, the two are highly intertwined. For example, you can improve client-side performance by enabling HTTP caching, which is implemented by returning specific HTTP headers from the server.


## Front-end Performance
Front-end optimization is low hanging fruit because:

1. on average, the front-end often accounts for ~80% of the time spent waiting for a page to load
2. front-end perfomance improvements are relatively straightforward to implement

There are great tools out there for profiling your front-end code such as [yslow](http://developer.yahoo.com/yslow/) and [page speed](https://developers.google.com/speed/pagespeed/) -- both tools offer browser extensions which analyze front-end peformance issues such as minification, http caching, parse order, etc.

Fundamentally, the client (eg. a web browser) does two things: 

1. sends and receives data over the network
2. renders javascript and html in the browser

Since web pages typically spend 80-90% of their load time sending and receiving data over the network, the rest of this section focuses on meothds for mitigating this.

It's worth mentioning that there are many useful rednering optimization techniques such as specifying image sizes in html, minimizing browser reflow, and using efficient css selectors. For further reading on rendering I recommend Google's [rendering best practices](https://developers.google.com/speed/docs/best-practices/rendering) and [article on browser reflow](https://developers.google.com/speed/articles/reflow).

Now, without further ado, let's dive into strategies for reducing network time...

##### Reduce Response Sizes
The more data our browser downloads, the longer it takes to load a page. Therefore, reducing response sizes is a tried and true strategy for improving client-side performance. Some common techniques include:

- **minifying html, css, js** - Minification is an easy way to reduce response sizes. Modern asset management libraries such as Sprockets and the Rails Asset Pipeline support minification out of the box.
- **compressing files** - If you've ever dealt with zip files, you are already familiar with this concept. Most modern browsers support a protocol for downloading resources in compressed form in order to expedite the network transfer. Read Google's ["How Gzip works"](https://developers.google.com/speed/articles/gzip).
- **serving scaled images** - It's convenient to serve images however you've stored them, but often times this means we are serving larger images than we need to. We can achieve performance gains by generating  thumbnails.

##### Reduce Request Sizes
More often overlooked, the size of your browser's *requests* also impact performance. The larger the size of the browser's requests, the longer it takes to upload them to the server and load pages. 

Since most users have asymmetric Internet connections (upload-to-download bandwidth ratios are commonly in the range of 1:4 to 1:20), a 500-byte HTTP header request can take the equivalent time to upload as 10 KB of HTTP response data takes to download.

Every time a client sends an HTTP request, it has to send all associated cookies that have been set for that domain and path along with it. Thus a common culprit is sending useless cookies in requests for static assets such as images. We can fix this by serving static concent from a separate, cookieless domain.

Finally, it's worth noting that all requests are transmitted via network packets which have a standard size limit of 1500 bytes  -- requests that are larger than 1500 bytes are sent over multiple network packets which pay a performance penalty. Keeping requests under 1500 bytes is optimal.

##### Reduce roundtrip time
The way browsers go about fetching resources is quirky and can vary from one browser to another.

For example, most browsers halt other requests while loading external JavaScript files. Browsers also limit the number of external requests they make to a single hostname in parallel -- so if a browser has 6 stylesheets to load on a page, it will fetch only 2 at a time while the rest are queued.

Therefore it's faster to load resources in a single request as one file, or parallelize requests by serving resources from different hostnames.

Serial requests cost us round-trip time which is overhead we want to avoid by making as few requests as possible and parallelizing where possible.

There are many techniques for reducing round-trip time. The most basic methods involve combining javascript, css, and images into single files to reduce the number of requests a browser has to make to load a page. The Rails Asset Pipeline provides this feature.

[Google's article on Round-trip Time](https://developers.google.com/speed/docs/best-practices/rtt) covers more strategies in detail. 


#### HTTP Caching
As we've seen so far, optimizing HTTP requests can be complex! Wouldn't it be ideal if we could skip sending requests altogether? YES WE CAN! ![](https://dl.dropboxusercontent.com/u/598519/ofa.png) Well, sort of.

HTTP protocol lets us specify how long a resoource should be cached so that browsers can load cached resources locally rather than requesting them from the server. HTTP protocol defines headers such as Last-Modified, ETag, Expires, Max-Age which can be used to determine how long a resource should be cached for.

It's important to remember that when the cache is expired or non-existent (eg. on the very first load of a page), we don't get any performance boost. So, for example, for a mission-critical first page load time of a website, HTTP caching won't help us (although on subsequent page loads we'd get performance benefits).

An added benefit (also a potential danger) of HTTP headers is that they can automatically trigger caching by various ISP's and proxy caches.

For a great explanation of HTTP caching I recommend [this article](http://betterexplained.com/articles/how-to-optimize-your-site-with-http-caching/). Heroku has a good [guide for HTTP caching in Rails](https://devcenter.heroku.com/articles/http-caching-ruby-rails).


## Back-end Performance
Back-end performance is defined by the time and resources required to serve a response that is sent to the client, including request handling, API calls, database queries, response rendering, etc.

Although front-end optimization can win us siginificant performance improvements with relative ease, especially for applications with serious data models and business logic, back-end performance is an inescapable issue.

Back-end performance becomes a larger issue because it scales exponentially when there are bottlenecks. This rarely happens on the front-end, where performance scales linearly. If your images are too big, or the page has too many files, you’ll see a slow-down but it will be pretty much constant regardless of load. It is unlikely you’ll hit a true bottleneck on the front-end.

The back-end, on the other hand, has a lot of potential bottlenecks. Not enough RAM or heap size? Thread contention? Too few database connections? A misconfigured database table, the wrong kind of locking, or missing indexes? A web server with max connections set too low? These are just a few possibilities. If any back-end factor becomes a bottleneck, you’ll see an exponential slowdown in performance.

Server-side performance tuning requires a hands-on understanding of the architecture, a good way to measure performance repeatedly, and lots of trial and error.

It can be helpful to think of an application as one big function that returns an HTTP response. The process of optimizng an application is similar to the process of optimizing a function: by isolating, testing, measuring, and refactoring.

Some common ways of measuring and examining web application performance include:

- **Profiling** -- Examining the performance of an application by disecting it with a code profiler (see: [rack-mini-profiler](https://github.com/SamSaffron/MiniProfiler/tree/master/Ruby))
- **Benchmarking** -- Assessing the relative performance of code by running it through standardized tests (see: [rails benchmarkable](http://api.rubyonrails.org/classes/ActiveSupport/Benchmarkable.html))
- **Load Testing** -- Testing an application to see how it reponds under a certain load (see: [httperf](http://mervine.net/performance-testing-with-httperf)). When abnormally high load is applied we refer to it as a *stress test*.

With that, let's talk about the most common server-side performance bottleneck…


### I/O
Whereas the bane of client performance is network time, one of the most prominent issues in back-end performance is I/O (input/output). Within the context of web performance, I/O generally refers to databases (disk I/O) and network services (network I/O).

In many ways, speeding up I/O is similar to reducing network time on the front-end. For example, API requests are HTTP requests with network latency no different than when using a browser. Requests to services such as database can also to some extent involve round-trip times similar to HTTP requests. All in all, I/O is slow, and I/O bottlenecks can be fatal.

We optimize I/O by reducing the quantity and expense of I/O operations, caching in memory, and making I/O non-blocking and asynchronous.
 
 
### Optimizing databases
Since the majority of I/O is typically database-related, understanding how to design, query, and manage databases propertly is very important.

Database optimization is a broad subject far beyond the scope of this article. But at a high-level, database optimization can be described as the proper tuning and configuration of databases along with optimizing the quantity, frequency, and expense of queries.

It's important to know how to identify slow queries and squash them, including common problems like n+1 queries, missing indexes, and knowning when to denormalize.

Fine-tuning doesn't just apply to SQL databases. Every service that the server interacts with -- whether it be an API or a Memcached process -- involves making sure that it is configured and utilized optimally. While this may sound straightforward, fine tuning each of these services (as well as others) takes significant time and expertise, extending out as far as the overall application architecture itself.

Next, we'll talk about some general pupose strategies for speeding up I/O.


### Caching

Due to the underlying technologies involved, we pay a price for database queries and API requests.

We've already explored aspects of network latency in front-end performance, but the worst part about network I/O via API's is that our application performance can become tied to a 3rd party. For example, if our app queries Twitter's API before displaying a page, our load time is dependent on Twitter's response time.

The reliability of databases are generally more controllable than 3rd party API's, but introduce another danger -- disk I/O. Disk I/O encompasses the input/output operations on a physical disk. The killer when working with a disk? Access time. This is the time required for a computer to process a data request from the processor and then retrieve the required data from the storage device. Since hard disks are mechanical, you need to wait for the disk to rotate to the required disk sector. Although database systems try to minimize disk I/O, oftem times it is unavoidable.

And this is where caching comes in -- when we talk about server-side caching, in most cases we mean caching *in memory* (although there are certainly other ways to cache data). Disk latency is around 13ms. API round trips are around 20ms (though faster if within the same datacenter) RAM latency is around 83 nanoseconds!

This is why memory caching systems like [Memcached](http://memcached.org/) have become such a staple of web application development – the savings in latency are enormous. 

For more reading I recommend the [Rails guide on caching](http://guides.rubyonrails.org/caching_with_rails.html) and Brett Slatkin's ["Number's Everyone Should Know"](http://highscalability.com/numbers-everyone-should-know).

###### Cache warming
Here are some general caching terms to familiarize yourself with:

- **Cache miss** - not in cache, must be loaded from the original source
- **Cache hit** - was loaded from cache
- **Cold cache** - data is not cached
- **Warm cache** - data is cached

As we saw with HTTP caching, caches inherently need to be written before they can be read from. We call this warming the cache.

Sometimes it's good enough to cache data upon a read request and then serve subsequent reads from cache. Other times, a cold cache is too costly so we proactively warm our cache with a computer program which is referred to as a *cache warmer*.


### Make I/O non-blocking
The worst thing about I/O isn't necessarily that its slow -- it's that in a typical request/response cycle, the end-user incurs the delay cost. 

For example, let's say that we need to make a series of API requests to Twitter in order to fetch lists of tweets to display on a page. If you are developing in Ruby, by default each API request (which is a Ruby expression) will have to complete a round-trip before the next the request even begins. Our code is "blocked" while each request processes. This is very slow! It's like a hamburger restaurant grilling one patty at a time when there are customers waiting in line. We call this synchronous I/O or blocking I/O because it "blocks" the process of a program while communication is in progress.

We can speed this up by parallelizing the requests to make them non-blocking and asynchronous. We can use some parallelization method such as [Ruby fibers](http://www.igvita.com/2010/03/22/untangling-evented-code-with-ruby-fibers/
) to send and process multiple requests simultaneously so that they aren't waiting on eachother to fire.

Another way to optimize serial I/O is to move them out of the response process altogether. By forking processes or using background jobs, we can offload work that can be performed out of band and handle it asynchronously so it doesn't delay response time eg. (synching a database with an external API, sending out notification emails).

Serial requests don't just affect API calls -- check out a [Rails plugin](http://ninjasandrobots.com/rails-faster-partial-rendering-and-caching)
 that one of my former co-workers wrote that solved the problem of serial Memcached requests when fetching cached fragments. It uses a Memcached multi-get, which batches memcached requests.

#### Additional resources:
- [High Scalability](http://highscalability.com)
- [Google Page Speed](https://developers.google.com/speed/docs/best-practices/rules_intro)
- [Rails Caching](http://guides.rubyonrails.org/caching_with_rails.html)