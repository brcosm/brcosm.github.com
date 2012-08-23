---
layout: post
title: Fun with NSURLCache
category: Development
tags: [iOS, UIWebView]
excerpt: This post covers an interesting use case for for an iOS App's shared NSURLCache.  Specifically I discuss, using the cache to manipulate sub-requests of a UIWebView and how it help me work around a pretty big bug in one of my Apps.
---
{% include JB/setup %}

### Some Background ... ###

I recently built an internal iOS app at Amazon that we are using to accept textbook trade-ins on college campuses.  An Amazon associate scans a student's textbook with the phone's camera and, if we accept the book and the student is happy with the price, it deposits a giftcard direclty into that student's account.  While the app has a completely native UI, I use a UIWebView to get session cookies (one session for the student and one session for the associate).

### The Problem ###

The app was working great until the owners of the Amazon.com login page deployed a change that introduced a strange bug [^1].  When going to the login page in Safari (or a UIWebView), there was one subrequest that could take up to 10 minutes to respond.  In Chrome, the same request returned immediately with an HTTP status code 204 (No Content).  

Unfortunately, the UIWebView delegate method webViewDidFinishLoad: doesn't get called until all of the sub-requests have finished loading.  Additionally, sub-requests don't cause the webView:shouldStartLoadWithRequest:navigationType: or webViewDidStartLoad: to be called either.  This resulted in really slow logins.

### The Solution ###

I tried several things including everyone's favorite trick of using categories to gain access to private APIs.  However, what ended up working the best was replacing the application's shared cache.  Unlike the UIWebView delegate methods, every request -- including the UIWebView's subrequests -- checks the shared NSURLCache [^2].  Knowing this, I was able to create an NSURLCache subclass that tricked the UIWebView into thinking there was a valid response in the cache for the bad request.

<script src="https://gist.github.com/3036248.js?file=gistfile1.m">

</script>

Sure enough, this change fixed the issue and still exists in the production version of the app.

[^1]: It would appear that this issue has been addressed.  I just checked the sign-in page with Safari and don't see the runaway request anymore.

[^2]: It is possible to configure a request to ignore the cache.  Doing so obviously breaks this technique.
