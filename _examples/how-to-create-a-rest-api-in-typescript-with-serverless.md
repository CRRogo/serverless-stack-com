---
layout: example
title: How to create a REST API in TypeScript with serverless
short_title: TypeScript REST API
date: 2021-02-04 00:00:00
lang: en
index: 3
# type: api
description: In this example we will look at how to create a serverless REST API on AWS with TypeScript using SST. We'll be using the Api construct to define the routes of our API.
short_desc: Building a REST API with TypeScript.
repo: rest-api-ts
ref: how-to-create-a-rest-api-in-typescript-with-serverless
comments_id: how-to-create-a-rest-api-in-typescript-with-serverless/2306
---

In this example we'll look at how to create a serverless REST API with TypeScript on AWS using [SST]({{ site.sst_github_repo }}). We also have a [a JavaScript version of this example]({% link _examples/how-to-create-a-rest-api-in-typescript-with-serverless.md %}) as well.

## Requirements

- Node.js >= 10.15.1
- We'll be using TypeScript
- An [AWS account]({% link _chapters/create-an-aws-account.md %}) with the [AWS CLI configured locally]({% link _chapters/configure-the-aws-cli.md %})

## Create an SST app

{%change%} Let's start by creating an SST app.

```bash
$ npx create-sst@latest --template=minimal/typescript-starter rest-api-ts
$ cd rest-api-ts
$ npm install
```

By default, our app will be deployed to an environment (or stage) called `dev` and the `us-east-1` AWS region. This can be changed in the `sst.json` in your project root.

```json
{
  "name": "rest-api-ts",
  "region": "us-east-1",
  "main": "stacks/index.ts"
}
```

## Project layout

An SST app is made up of two parts.

1. `stacks/` — App Infrastructure

   The code that describes the infrastructure of your serverless app is placed in the `stacks/` directory of your project. SST uses [AWS CDK]({% link _chapters/what-is-aws-cdk.md %}), to create the infrastructure.

2. `services/` — App Code

   The code that's run when your API is invoked is placed in the `services/` directory of your project.

## Setting up our routes

Let's start by setting up the routes for our API.

{%change%} Replace the `stacks/MyStack.ts` with the following.

```ts
import * as sst from "@serverless-stack/resources";

export default class MyStack extends sst.Stack {
  constructor(scope: sst.App, id: string, props?: sst.StackProps) {
    super(scope, id, props);

    // Create the HTTP API
    const api = new Api(stack, "Api", {
      routes: {
        "GET /notes": "src/list.handler",
        "GET /notes/{id}": "src/get.handler",
        "PUT /notes/{id}": "src/update.handler",
      },
    });

    // Show the API endpoint in the output
    stack.addOutputs({
      ApiEndpoint: api.url,
    });
  }
}
```

We are creating an API here using the [`Api`]({{ site.docs_url }}/constructs/api) construct. And we are adding three routes to it.

```
GET /notes
GET /notes/{id}
PUT /notes/{id}
```

The first is getting a list of notes. The second is getting a specific note given an id. And the third is updating a note.

## Adding function code

For this example, we are not using a database. We'll look at that in detail in another example. So internally we are just going to get the list of notes from a file.

{%change%} Let's add a file that contains our notes in `src/notes.ts`.

```ts
interface Note {
  noteId: string;
  userId: string;
  createdAt: number;
  content: string;
}

const notes: { [key: string]: Note } = {
  id1: {
    noteId: "id1",
    userId: "user1",
    createdAt: Date.now(),
    content: "Hello World!",
  },
  id2: {
    noteId: "id2",
    userId: "user2",
    createdAt: Date.now() - 10000,
    content: "Hello Old World! Old note.",
  },
};

export default notes;
```

Now add the code for our first endpoint.

### Getting a list of notes

{%change%} Add a `src/list.ts`.

```ts
import { APIGatewayProxyResult } from "aws-lambda";
import notes from "./notes";

export async function handler(): Promise<APIGatewayProxyResult> {
  return {
    statusCode: 200,
    body: JSON.stringify(notes),
  };
}
```

Here we are simply converting a list of notes to string, and responding with that in the request body.

Note that this function need to be `async` to be invoked by AWS Lambda. Even though, in this case we are doing everything synchronously.

### Getting a specific note

{%change%} Add the following to `src/get.ts`.

```ts
import { APIGatewayProxyEvent, APIGatewayProxyResult } from "aws-lambda";
import notes from "./notes";

export async function main(
  event: APIGatewayProxyEvent
): Promise<APIGatewayProxyResult> {
  const note =
    event.pathParameters && event.pathParameters.id
      ? notes[event.pathParameters.id]
      : null;
  return note
    ? {
        statusCode: 200,
        body: JSON.stringify(note),
      }
    : {
        statusCode: 404,
        body: JSON.stringify({ error: true }),
      };
}
```

Here we are checking if we have the requested note. If we do, we respond with it. If we don't, then we respond with a 404 error.

### Updating a note

{%change%} Add the following to `src/update.ts`.

```ts
import { APIGatewayProxyEvent, APIGatewayProxyResult } from "aws-lambda";
import notes from "./notes";

export async function main(
  event: APIGatewayProxyEvent
): Promise<APIGatewayProxyResult> {
  const note =
    event.pathParameters && event.pathParameters.id
      ? notes[event.pathParameters.id]
      : null;

  if (!note) {
    return {
      statusCode: 404,
      body: JSON.stringify({ error: true }),
    };
  }

  if (event.body) {
    const data = JSON.parse(event.body);
    note.content = data.content || note.content;
  }

  return {
    statusCode: 200,
    body: JSON.stringify(note),
  };
}
```

We first check if the note with the requested id exists. And then we update the content of the note and return it. Of course, we aren't really saving our changes because we don't have a database!

Now let's test our new API.

## Starting your dev environment

{%change%} SST features a [Live Lambda Development]({{ site.docs_url }}/live-lambda-development) environment that allows you to work on your serverless apps live.

```bash
$ npm start
```

The first time you run this command it'll take a couple of minutes to deploy your app and a debug stack to power the Live Lambda Development environment.

```
===============
 Deploying app
===============

Preparing your SST app
Transpiling source
Linting source
Deploying stacks
manitej-rest-api-ts-my-stack: deploying...

 ✅  manitej-rest-api-ts-my-stack


Stack manitej-rest-api-ts-my-stack
  Status: deployed
  Outputs:
    ApiEndpoint: https://rxk5buowgi.execute-api.us-east-1.amazonaws.com
```

The `ApiEndpoint` is the API we just created.

Let's test our endpoint using the integrated [SST Console](https://console.sst.dev). The SST Console is a web based dashboard to manage your SST apps [Learn more about it in our docs]({{ site.docs_url }}/console).

Go to the **API** explorer and click the **Send** button of the `GET /notes` route to get a list of notes.

Note, The [API explorer]({{ site.docs_url }}/console#api) lets you make HTTP requests to any of the routes in your `Api` construct. Set the headers, query params, request body, and view the function logs with the response.

![API tab get notes response](/assets/examples/rest-api/api-tab-get-notes-response.png)

You should see the list of notes as a JSON string.

To retrieve a specific note, Go to `GET /notes/{id}` route and in the **URL** tab enter the **id** of the note you want to get in the **id** field and click the **Send** button to get that note.

![API tab get specific note response](/assets/examples/rest-api/api-tab-get-specific-note-response.png)

Now to update our note, we need to make a `PUT` request, go to `PUT /notes/{id}` route.

In the **URL** tab, enter the **id** of the note you want to update and in the **body** tab and enter the below json value and hit **Send**.

```json
{ "content": "Updating my note" }
```

![API tab update note response](/assets/examples/rest-api/api-tab-update-note-response.png)

This should respond with the updated note.

## Making changes

Let's make a quick change to our API. It would be good if the JSON strings are pretty printed to make them more readable.

{%change%} Replace `src/list.ts` with the following.

```ts
import { APIGatewayProxyResult } from "aws-lambda";
import notes from "./notes";

export async function handler(): Promise<APIGatewayProxyResult> {
  return {
    statusCode: 200,
    body: JSON.stringify(notes, null, "  "),
  };
}
```

Here we are just [adding some spaces](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify) to pretty print the JSON.

If you head back to the `GET /notes` route and hit **Send** again.

![API tab get notes response with spaces](/assets/examples/rest-api/api-tab-get-notes-response-with-spaces.png)

You should see your list of notes in a more readable format.

## Deploying your API

{%change%} To wrap things up we'll deploy our app to prod.

```bash
$ npx sst deploy --stage prod
```

This allows us to separate our environments, so when we are working in `dev`, it doesn't break the app for our users.

Once deployed, you should see something like this.

```bash
 ✅  prod-rest-api-ts-my-stack


Stack prod-rest-api-ts-my-stack
  Status: deployed
  Outputs:
    ApiEndpoint: https://ck198mfop1.execute-api.us-east-1.amazonaws.com
```

Run the below command to open the SST Console in **prod** stage to test the production endpoint.

```bash
npx sst console --stage prod
```

Go to the **API** explorer and click **Send** button of the `GET /notes` route, to send a `GET` request.

![Prod API explorer get notes response with spaces](/assets/examples/rest-api/prod-api-tab-get-notes-response-with-spaces.png)

## Cleaning up

Finally, you can remove the resources created in this example using the following commands.

```bash
$ npx sst remove
$ npx sst remove --stage prod
```

## Conclusion

And that's it! You've got a brand new serverless API. A local development environment, to test and make changes. And it's deployed to production as well, so you can share it with your users. Check out the repo below for the code we used in this example. And leave a comment if you have any questions!
