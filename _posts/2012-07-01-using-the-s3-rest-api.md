---
layout: post
title: Using the S3 REST API
category: Development
tags: [iOS, AWS, S3]
excerpt: This post discusses some of my learnings from using the S3 REST API in an iOS App.  This particular post focuses on generating signed requests.

---
{% include JB/setup %}

### Using the S3 REST API in iOS Apps ###

One of the apps I am working on makes heavy use of Amazon's S3 web service.  Other than only offering XML(no JSON), I like the API.  The following post contains some notes and comments from my implementation.  I am working on a objective-C library that provides a higher level of abstraction than even the official [AWS iOS SDK][aws_ios_sdk].   

### Generating the HTTP request ###

The API, and S3 in general, is defined by operations on three main abstractions:

1. Service - The S3 service itself
2. Bucket - A collection of objects(files) used for organization and access control
3. Object - The actual files that are stored in S3  

The only service operation lists the available buckets.  This operation is done via

    HTTP GET /
    HOST s3.amazonaws.com

There are a variety of bucket operations, but the canonical request is 

    HTTP VERB /?<param1=val1&param2=val2>
    HOST <BucketName>.s3.amazonaws.com

Similar to buckets, the object operations follow

    HTTP VERB /<ObjectName>?<param1=val1&param2=val2>
    HOST <BucketName>.s3.amazonaws.com

### Authorization Header ###

In addition to the host and resource path [some headers][aws_headers_docs] must be included in each request.  The most interesting(and troublesome) of these is the Authorization header that is used for authentication.  The signing process, as described in the [AWS S3 Rest Documentation][aws_s3_docs], is straight forward.  In practice, however, my implementation for iOS was more complicated than I anticipated.

The value of the Authorization header is a token that takes the form:

    AWS <AWSAccessKey>:<RequestSignature>  
    
The 'RequestSignature' is the important piece and is a Base64 encoded HMAC-SHA1 hash of the request information with your AWS Secret key.


### Generating the Request Signature ###

To generate the signature used in the Authorization header, I had to take the following steps:

1. Generate a string to sign from the NSURLRequest for the S3 resource
2. Encrypt the string as an HMCA-SHA1 hash with the AWS Secret key
3. Mutate the NSURLRequest to include this string in the token for the 'Authorization' header  

I will cover steps 1 and 2 in another article, step 3 is trivial.


[aws_ios_sdk]:http://aws.amazon.com/sdkforios/
[aws_s3_docs]:http://docs.amazonwebservices.com/AmazonS3/latest/API/APIRest.html 
[aws_headers_docs]:http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTCommonRequestHeaders.html
