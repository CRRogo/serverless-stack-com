---
layout: post
title: Create a Hello World API
date: 2021-08-17 00:00:00
lang: en
description: In this chapter we'll be creating a simple Hello World API using SST. We'll be deploying it using the Live Lambda development environment.
ref: create-a-hello-world-api
comments_id: create-a-hello-world-api/2460
---

With our newly created [SST]({{ site.sst_github_repo }}) app, we are ready to deploy a simple _Hello World_ API.

In `stacks/MyStack.js` you'll notice a API definition similar to this.

```js
import { Api } from "@serverless-stack/resources";

export function MyStack({ stack }) {
  // Create an HTTP API
  const api = new Api(stack, "Api", {
    routes: {
      "GET /": "functions/lambda.handler",
    },
  });

  // Show the endpoint in the output
  stack.addOutputs({
    ApiEndpoint: api.url,
  });
}
```

Here we are creating a simple API with one route, `GET /`. When this API is invoked, the function called `handler` in `functions/lambda.js` will be executed.

Let's go ahead and create this.

## Starting your dev environment

We'll do this by starting up our local development environment.

{%change%} SST features a [Live Lambda Development]({{ site.docs_url }}/live-lambda-development) environment that allows you to work on your serverless apps live.

```bash
$ npx sst start
```

The first time you run this command it'll take a couple of minutes to deploy your app and a debug stack to power the Live Lambda Development environment.

```txt
===============
 Deploying app
===============

Preparing your SST app
Transpiling source
Linting source
Deploying stacks
dev-notes-MyStack: deploying...

 dev-notes-MyStack


Stack dev-notes-MyStack
  Status: deployed
  Outputs:
    ApiEndpoint: https://guksgkkr4l.execute-api.us-east-1.amazonaws.com

==========================
Starting Live Lambda Dev
==========================

SST Console: https://console.sst.dev/notes/dev/local
Debug session started. Listening for requests...
```

The `ApiEndpoint` is the API we just created. Let's test our endpoint. If you open the endpoint URL in your browser, you should see _Hello World!_ being printed out.

![Serverless Hello World API invoked](/assets/part2/sst-hello-world-api-invoked.png)

You can also head over to the **SST Console** link in your browser. The [SST Console]({{ site.docs_url }}/console) is a web based dashboard to manage your SST apps.

![SST Console Local tab](/assets/part2/sst-console-local-tab.png)

The **Local** tab shows you real-time logs from your apps.

Note that when you hit this endpoint the Lambda function is being run locally.

## Deploying to prod

To deploy our API to prod, we'll need to stop our local development environment and run the following.

```bash
$ npx sst deploy --stage prod
```

We don't have to do this right now. We'll be doing it later once we are done working on our app.

The idea here is that we are able to work on separate environments. So when we are working in `dev`, it doesn't break the API for our users in `prod`. The environment (or stage) names in this case are just strings and have no special significance. We could've called them `development` and `production` instead. We are however creating completely new serverless apps when we deploy to a different environment. This is another advantage of the serverless architecture. The infrastructure as code idea means that it's easy to replicate to new environments. And the pay per use model means that we are not charged for these new environments unless we actually use them.

Now we are ready to create the backend for our notes app. But before that, let’s create a GitHub repo to store our code.
