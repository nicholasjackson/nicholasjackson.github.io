---
layout: post
title:  "Configuring API Gateway for GraphQL Lambda functions"
tags: [serverless, graphql]
author: nic
image: /images/posts/graphql/graphql.png
---

This is a short post which hopefully will explain a few gotchas when setting up a GraphQL endpoint with AWS Lambda.

The main problem is not Lambda per-se it is actually API Gateway which sits in front of Lambda.  By default API
Gateway expects input and output type of *application/json* and this is fine except when you send a POST request which contains a data payload in the format of *application/graphql*.  The best practice document on the GraphQL website [http://graphql.org/learn/serving-over-http/](http://graphql.org/learn/serving-over-http/) recommends that a POST request should enable both requests in application/json format and in application/graphql.  

If you try to send a query in graphql format to API Gateway like the following:
```
curl -i -XPOST -H 'Content-Type: application/graphql' -d '{ products { code } }' http://myendpoint.com/prod
```

You will find that you get an error message like the following:

```
HTTP/1.1 400 Bad Request
 Content-Type: application/json
 Content-Length: 189
 Connection: keep-alive
 Date: Tue, 13 Dec 2016 11:04:31 GMT
 x-amzn-RequestId: eca43820-c123-11e6-8b65-53d711979cac
 X-Cache: Error from cloudfront
 Via: 1.1 92423f766303b20998ffc97d623638f7.cloudfront.net (CloudFront)
 X-Amz-Cf-Id: MZVqwhSakST3qG4B_lKoGVuDdBvtwJfY4hkamF8pgDN-bb6S9e5ERg==
 
 {"message": "Could not parse request body into json: Unexpected character (\'p\' (code 112)): was expecting double-quote to start field name\n at [Source: [B@77acd1ac; line: 1, column: 4]"}%
```

This is due to API Gateway attempting to convert our post body into JSON, which it clearly is not.

## How to fix this
To fix this problem we need to make some changes to the API Gateway configuration.

### Step 1: Enable Binary Support
The first thing we need to do is to enable binary support, if you click the *Binary Support* menu item you will be presented with a screen which allows you to tell API Gateway to treat payloads associated with certain media types as binary data.  This means that it will not attempt to convert the payload to JSON.   

![](/images/posts/graphql/binary_support.png)  

If we add our *application/graphql* media type and re-deploy the API you will see that we are now getting a slightly different error message.

```
HTTP/1.1 400 Bad Request
 Content-Type: application/json
 Content-Length: 203
 Connection: keep-alive
 Date: Tue, 13 Dec 2016 11:14:14 GMT
 x-amzn-RequestId: 47d5bff1-c125-11e6-aef0-459b4e1c48a4
 X-Cache: Error from cloudfront
 Via: 1.1 92423f766303b20998ffc97d623638f7.cloudfront.net (CloudFront)
 X-Amz-Cf-Id: xoaKYlLZg866CwA14f3p0u5N15sYxQcwiowCy1sdm9HDsmIFp0oxhw==
 
 {"message": "Could not parse request body into json: Unrecognized token \'eyBwcm9kdWN0cyB7IGNvZGUgfSB9\': was expecting (\'true\', \'false\' or \'null\')\n at [Source: [B@68df8622; line: 1, column: 57]"}%
```

### Step 2: Add a mapping template
Just adding binary support is not enough, API Gateway is still attempting to parse the data into JSON however this is now happening in a later stage of the request, if you  click on your resource and look at the Method Execution flow you should have an *Integration Request* item.  Clicking on this will take you to the following screen and expanding the *Body Mapping Templates* section we need to add a new template.


![](/images/posts/graphql/body_mapping_template.png)  

Add a mapping template for the content type *application/graphql* and then configure your template by adding the following code.

```
{ "query": "$util.base64Decode($input.body)" }
```

This transforms the input into the output with the following structure.

```
{
  "query": "{ products { code title } }"
}
```

When we use the binary support our request body represented by *$input.body* is stored as a base64 encoded string.  We can use the utility function *base64Decode(string)* to transform this back into a string.  Lambda is still expecting a Payload in JSON format by using the mapping template we can take our input and wrap this into a format that the lambda function is happy with.

Now again deploy the API and re-test.

You should see your data now returning as requested.

```
HTTP/1.1 200 OK
 Content-Type: application/json
 Content-Length: 1622
 Connection: keep-alive
 Date: Tue, 13 Dec 2016 11:34:07 GMT
 x-amzn-RequestId: 0c943239-c128-11e6-94b6-7967c2010156
 X-Amzn-Trace-Id: Root=1-584fdcab-b22f22b3fe1fc4d5323d71c1
 X-Cache: Miss from cloudfront
 Via: 1.1 be27f8aa8af4ef7f3d1f074ecbf3b266.cloudfront.net (CloudFront)
 X-Amz-Cf-Id: NDfYQ70rUlqMrJt-_gjRMBCYDEj5eED7JM8bPCkFMRb7ZGXoWgmiFQ==
 
 {"data":{"products":[{"code":176176},{"code":465956},{"code":277748}]}}%
```

One thing to note is that your lambda function must be returning the content type as *application/json* if it is not you might find that instead of returning JSON API Gateway wraps your output in a string which will most likely break things.

