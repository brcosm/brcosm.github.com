---
layout: post
title: Working with Glacier
category: Development
tags: [iOS, AWS, Glacier]
excerpt: This post dicusses my first attempt at writing a client for the new AWS service called Glacier.  I also discuss a use case and how it can be served by an iOS device.

---
{% include JB/setup %}

As soon as I read about the new AWS service, [Glacier][g], I started writing a client for iOS/OSX.  Because of some of my [past expiriments][gc] with S3, I was fairly familiar working with AWS REST APIs.

I immediately noticed a few things:

1. The Glacier REST service uses JSON instead of XML
2. There is new authentication protocol
2. The Glacier AWS console doesn't allow any upload/download operations -- you have to do these actions programmatically

A new authentication protocol
-----------------------------

Back in March, Amazon announced a new request signing version for some (eventually all?) of the AWS services.

> The new protocol, Signature Version 4, will enable AWS to support future growth and evolution of the AWS business. It introduces a specialized signing key, derived from the long-term AWS Access Key, which is used for the cryptographic signature. It also features incremental modifications in the canonicalization algorithm that streamline signature verification.

I missed the announcement back in March, but was happend upon the [spec][sv4] while implementing [Cocoa Glacier client][gc].  I don't agree with the premise that the new version 'streamlines' signature verification, but I do think it is interesting and I intend to dedicate an entire post to it once I have really figured it out.  

Needless to say, implementing the spec has been a bit of a challenge.  One thing that has helped is the [test suite][ts] included in the signature version 4 documentation.  In addition to helping me debug my implementation, it also helped me structure my code and be consistent with my nomenclature.

My use case for the v1 client
---------------------------------------

Getting data from Glacier is an asynchronous activity that can take several hours.

> Amazon Glacier is optimized for data that is infrequently accessed and for which retrieval times of several hours are suitable.

This type of behavior creates a unique user experience.  If I want to get an inventory of one of my Glacier Vaults[^vault] or download an Archive[^archive], I have to queue up a request and come back later to actually perform the action.  To me, this sounds like a great use case for a mobile client and is my main motivation.

I queue up an Archive retrieval job from my phone, get a mobile notification when the job is ready, and download the archive to my computer or an S3 Bucket.

AWS architecture
----------------

Combining a few of the AWS services will allow the interaction I described above.  Vaults can be configured to send SNS notifications.  When a job is ready, the notification can be send to an SQS queue which is being monitored by some script.  The script then generates an Apple Push Notification to notify the user that the job is ready.

Right now I am working on a [CloudFormation][cf] template to generate an SNS topic that send notifications to an SQS queue, and an EC2 instance imaged with a script that polls the queue and generates the push notification.

[^vault]: Vault is Glacier's version of a bucket

[^archive]: Archive is file/data that is stored in a Glacier Vault

[sv4]: http://docs.amazonwebservices.com/general/latest/gr/signing_aws_api_requests.html

[g]: http://http://aws.amazon.com/glacier/

[gc]: http://github.com/brcosm/

[ts]: http://docs.amazonwebservices.com/general/latest/gr/signature-v4-test-suite.html

[cf]: http://aws.amazon.com/cloudformation/
