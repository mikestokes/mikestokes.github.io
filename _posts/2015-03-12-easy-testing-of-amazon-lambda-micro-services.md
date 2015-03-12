---
layout: post
title: Easy Testing of Amazon (AWS) Lambda Micro-Services
comments: True
category: aws
tags: [aws, amazon, lambda, micro-services]
---

(See the [GitHub repo](https://github.com/mikestokes/aws-lambda-s3-boilderplate) to clone the code)

[AWS Lambda](http://aws.amazon.com/lambda/) is a service (currently in Preview) that allows us to easily run code in response to certain events. What that means is we can easily create small Micro-Services that do one thing and do it well in response to any given event. Good exmaples include:

 * Automatically optimizing images that are droppped into an S3 bucket
 * Data triggers (DynamoDB or Kineses stream processing)
 * Large volume, parallel task running
 * Scheduled tasks

Currently Lambda only supports [Node.js](https://nodejs.org/)... Luckily I love Node! They also have a very reasonable [free tier](http://aws.amazon.com/lambda/pricing/) so we can try it out.

## How to Test Locally?

Since Lambda is a hosted [PaaS (Platform as a Service)](http://en.wikipedia.org/wiki/Platform_as_a_service) offering from AWS you get a lot for your money (great for lazy coding) but you also lose some control. Testing is one of those things you lose some control of when it's not easy to remote into the machine.

![Lambda Logs in AWS Cloud Watch](http://mikestokes.co/public/img/lambda-cloudwatch-logs.png)
Example of Lambda logs in AWS Cloud Watch.

## Creating a Simple Local Test Harness

Luckily for us, it's easy to create a **test harness** locally to simulate the Lambda environment events and run our code. This allows us to use our existing debugging toolset, e.g. [node inspector](https://github.com/node-inspector/node-inspector), [WebStorm](https://www.jetbrains.com/webstorm/), [Visual Studio](https://www.visualstudio.com/) etc.

For example, in this sample, we simulate an S3 file Put event and call our code in the same way Lambda would.

The test.js file:

```javascript
// Our Lambda function fle is required 
var importify = require('./importify.js');

// The Lambda context "done" function is called when complete with/without error
var context = {
    done: function (err, result) {
        console.log('------------');
        console.log('Context done');
        console.log('   error:', err);
        console.log('   result:', result);
    }
};

// Simulated S3 bucket event
var event = {
    Records: [
        {
            s3: {
                bucket: {
                    name: 'hotlunch-west-2'
                },
                object: {
                    key: 'importing/org_create_table_test/4-25-mytestfile.docx'
                }
            }
        }
    ]
};

// Call the Lambda function
importify.handler(event, context);
```

The importify stub - your Lambda function entry point:

```javascript
var AWS = require('aws-sdk');

// Your exported Lambda entry point
exports.handler = function(event, context) {
	console.log('Received event:');
	console.log(JSON.stringify(event, null, '  '));

	// S3 information from the event
	var bucket = event.Records[0].s3.bucket.name;
	var key = event.Records[0].s3.object.key;

	// Do something with the S3 information
    doSomething(bucket, key, function (err, result) {
		console.log('error:', err);
		console.log('result:', result);

		// Notify Lambda done with error or result
		context.done(err, result);
	});
};
```

## Running the Test Locally

Running the test locally is easy, from the command line (or terminal on Mac):

```bash
> node test.js
```

## Why this isn't Perfect

Of course what we're doing here isn't perfect - we're not simulating the Lambda environment precisely (including memory limits). But we do now have a way to quickly develop and test Lambda micro-services locally and then deploy them to Lambda for integration testing. This process minimizes the "change-zip-upload-run-check logs" process that is required otherwise.

## How to Test Remotely?

I'm working on this one... hopefully I'll have a update soon!

## Lambda Limitation in Preview

Since Lambda is in preview, there are some published limitations and also some that we've found out through using it on a daily basis:

 * Limited events can generate Lambda code to run (S3, DynamoDB and Kinesis events currently)
 * Node.js is the only supported Lambda platform currently supported
 * Limited RAM (1 GB) and time-out (60 seconds max) settings
 * Difficulty debugging using the logs created in CloudWatch (requires verbosity in logging)
 * Potential issues when Lambda re-uses the same environment for multiple runs need to be taken into account - write your code as isolated
 * Requires a better way to control retries on failure (retry count, signal error but no retry etc)
 * Slightly unpredictable things occur sometimes - make sure you over-log your code to detemine what's up
 * Vendor lock-in is a real problem but the benefits for us outweigh the cons so far

Having said this, Lambda is an awesome service that I strongly recommend you try.
