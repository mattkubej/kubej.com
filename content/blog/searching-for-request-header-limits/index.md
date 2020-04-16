---
title: Searching for request header limits 
date: "2020-04-16T01:46:16Z"
description: Letting my computer do the work 
---

I recently encountered an interesting challenge where an application I work on 
rejected HTTP requests due to a large header above the limit of the server.  
Specifically, NodeJS adjusted their max header size by default to 8KB within 
the past couple of years as seen within 
[this commit](https://github.com/nodejs/node/commit/a8532d4d23).  However, even 
though I could adjust the default NodeJS max header size to a larger value,
other systems existed before and after this NodeJS instance that propagate the 
header.  So, these other systems remain susceptible to rejecting my request as 
well.  How can I easily identify the maximum header size accepted by the entire 
system in concert?

Rather than digging into the technical documentation and implementation of 
each system at hand, I figured I could use this as an opportunity to continue 
to explore Golang and get my computer to discover the answer for me.  I could 
feasibly send varying sized request headers and inspect the response of the 
request to determine whether it succeeded or failed.  Through a simple search 
across a range of values I could identify the largest request header size that 
provides a successful response.

This presented an interesting problem.  I want to find an unknown 
value within a range of values, which I will use to construct appropriately 
sized requests to a target application.  A naive approach could consist of 
starting at the lowest part of the range and iterate through the range 
increasing by one until the requests go from success to failure within an 
iteration.  However, if I want the granularity of bytes, then with this 
approach I may send thousands of requests depending on the size of the range. 
Not only does this seem inefficient or slow, but the system might even 
consider my search as a Distributed Denial-of-Service (DDoS) attack.  How 
could I find my answer in as few requests as possible?

This seemed like an appropriate place for 
[binary search](https://en.wikipedia.org/wiki/Binary_search_algorithm).  I had 
a range of sorted values, which consisted of possible request header sizes in 
the unit of bytes.  I could take the median of the range, then use that for my 
first request to see if it succeeds or fails.  If it fails, then I know I do 
not care about range of values above the median and can place my attention on 
the bottom half of values.  Inversely, if it succeeds, then I can forget about 
the lower half of values and focus on the upper half.  With my new range of 
values cut in half, then I can continue this process over and over again until 
I reach my answer.  I know I have found my answer when the low header size 
provides a successful request, the high header size provides a failed request, 
and the low and the high header sizes have a difference of one.

Listed below is some psuedocode for conducting this search similar to my 
implementation.

```go
low := 4000 // bottom end of range
high := 12000 // top end of range

for (high - low) != 1 {
  median := (low + high) / 2

  statusCode := invoke(url, median)
  okStatus := statusCode >= 200 && statusCode < 300

  if okStatus {
    low = median
  } else {
    high = median
  }
}

lowStatusCode := invoke(url, low)
lowStatusOk := lowStatusCode >= 200 && lowStatusCode < 300

highStatusCode := invoke(url, high)
highStatusOk := highStatusCode >= 200 && highStatusCode < 300

if (high - low) == 1 && lowStatusOk && !highStatusOk {
  fmt.Printf("Max Header Size: %s", low)
}
```

The only issue I ran into a decent amount of challenge with consisted of 
identifying the exact header size.  With each strategy I attempted my 
application would find a max header size within ~11-70 bytes of what I would 
expect with my tests against a local NodeJS server.  This led me to believe 
either I did not arrive at the correct header request size value within my 
application or the NodeJS max header size did not abide by its own max request 
header size.  Either way, I think less than 100 bytes of error seemed 
reasonable for the answer I sought.

I ended up calculating the request header size in a simple fashion with 
`httputil.DumpRequest`.

```go
func hdrLength(req *http.Request) int {
	data, err := httputil.DumpRequest(req, false)

	if err != nil {
		fmt.Fprintf(os.Stderr, "error: %v", err)
		os.Exit(1)
	}

	return len(data)
}
```

You can find my full implementation at:
[https://github.com/kubejm/goldilox](https://github.com/kubejm/goldilox)
