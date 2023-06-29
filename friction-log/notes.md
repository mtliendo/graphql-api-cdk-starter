# Amplify CDK Fullstack: Initial Experience

## Description

This is my first time working with the `agqlac` module. My understanding is that this package brings an Amplify-like experience to the CDK by making use of directives like `@model`, `@hasMany` etc. In addition, this will leverage the GraphQL Transform tool to inspect a schema and automatically create CRUDL opeations based on the schema.

## Branch Purpose

This branch will be used to see what the getting started experience is like. Refer to the other branches to best see what it looks like building a full application.

Branch Projects:

1. Todo App ⬅️ You are here. Difficulty: Basic
2. Trip Logger App. Difficulty: Medium
3. MicroSaaS App. Difficulty: Advanced

## Assumptions/Concerns

Before diving in, I'm curious of the following:

- **How well** does the schema inspection and code generation work.
- **Understanding** what the L3 construct does and how customers are expected to use it.
- **What can I ask the team** to enhance/better document. As an L3 construct, what are the baked in opinions and the things that would make a customer use something else (this will be ongoing)
- **Concern**: Will this be updated frequently? What will we do to ensure it keeps up with weekly CDK rollouts?
- **Concern**: What does intellsense look like?

---

## Project Understanding

I started by installing the dependencies:
`npm i`

Before running commands, I reviewed the project structure and came to the following understanding:

- This is a monorepo: CDK backend with CRA frontend
- The `scripts/generate.mjs` file is used to essentially call the Amplify CLI (`npx @aws-amplify/cli codegen add`) and create the `schema.graphqlconfig.yml` file. My assumption is that this will end up being tucked away as a separate package or internal script when this is all finalized.
- The `backend/app.ts` file creates a stack, and an AppSync API. This API has 4 Datasources: 3 as DynamoDB Tables and 1 as a Lambda Function (stubbed out in `backend/repeat.ts`). Looks like the schema is defined inline.
- The `src` directory is the Create React App frontend and is written in TypeScript that makes use of Amplify Generics, UI Library, and yet-to-be-generated operations.
- Instead of an `aws-exports.js` file, it looks like there is a `./appConfig` file. I like this naming.

## Getting Started

### Backend Deployment

Hmmm...I'm actually not sure where to start here. The `README.md` file asks for there to be `amplify` version >=12 installed. It's worth noting that most CDK users don't want to use the Amplify CLI and will instead opt to run Amplify commands from `npx`.

I'm going to follow the `README` and run the `deploy` script.

> I'm using my personal AWS account and also opting to use SSO instead of a straight up AWS account.

`npm run deploy`

Failed to deploy

> Error: Unable to resolve AWS account to use. It must be either configured when you define your CDK Stack, or through the environment

Simple enough. I rely on an AWS profile instead of a `default` account so I have to hardcode those values. But this is the first difference: As a CDK user, I'm used to having a `bin` directory, and a `lib` directory. This begs the question:

> _Who is the target persona here?_ Someone in between Amplify and the CDK in terms of experience? A fullstack dev that wants more control than what Amplify CLI delivers but finds creating an API verbose? Not sure. An error like this would need someone familiar with the CDK.

In future branches I'll likely create a `bin` and a `lib` but for now, I'll keep it the same. I updated the following files and reran the command.

```diff
// app.ts
const app = new cdk.App()
const stack = new cdk.Stack(app, 'TestStack', {
+	env: {
+		account: '842537737558',
+		region: 'us-east-1',
+	},
})
```

```diff
// package.json
- "deploy": "cdk deploy"
+ "deploy": "cdk deploy --profile focus-otter-sandbox"
```

```sh
npm run deploy
```

Once deployed, I was able to see a familiar CDK-style deploy confirmation and relevant CloudFormation output in my terminal.

![test stack deploy](./images/test-stack-deploy.png)

The nice thing here is that I get my expected output: AppSync endpoint, API ID, and API Key.

So far so good, though I don't see my resolvers in my local project.

Verifying the deployment in the AWS console, I notice the following:
![appsync api name](./images/api-name.png)

1. The AppSync API has a `-NONE` suffix. I'm not sure what this relates to and how/if I can configure that.
2. There's an additional datasource I forgot to account for, the `NONE` datasource for subscriptions.
3. **X-Ray is turned off in the settings.** But in the intellisense docs, it's listed as defaulting to enabled. I'll come back to this later on.

![appsync api intellsense](./images/appsync-api-intellisense.png)

4. The biggest thing: This uses VTL. I'm torn here. I don't want to push VTL at all. However, the idea here is that customers won't have to worry about this. This is a tradeoff that customers should be aware of upfront. What I don't like about generated VTL is that our generation of it tends to be very verbose so that I can be generic. This makes it difficult to modify. I'll explore this later on in this branch.

At this point, this feels like Amplify CLI with more steps. What I mean is that I can _imagine_ the benefits, but it's taking some mental gymnastics. Hopefully by the end of this branch I'll have some of these cleared up.

### Generating Code for Frontend

I'm assuming this will be pretty straighforward. From the `README`, looks like I just need to run the following command:

```sh
npm run generate
```

> Error: [1/5] pulling Api Config
> Caught exception while executing generate script Error: Region is missing

It's worth noting that as someone extremely familiar with the CLI and that defaults to building with the CDK, every error that pops up lessens my trust and gets me thinking "I should just use Amplify CLI/CDK. The feedback here is that proper error handling will be crucial for adoption.

Let's see if I can debug this...

Per the error messageIt's failing on step 1. Looking in the `generate.mjs` file it's calling `getApiConfig`. All the AppSync API defaults look fine, but it's in search of a stack called `TestStack`. Poking around in CloudFormation (something a customer wouldn't be expected to do), I see the stack is there. Hmmm...looking in Slack and looks like Ryan has a thread going on this exact issue.

It's unresolved 😩

Looks like it's failing on this line:

```ts
const results = await new cfn.CloudFormationClient().send(
	new cfn.DescribeStacksCommand()
)
```

I added a region:

```diff
const results = await new cfn.CloudFormationClient().send(
-	new cfn.DescribeStacksCommand()
+	new cfn.DescribeStacksCommand({region:"us-east-1"})
)
```

As expected, this results in the following error:

> "Caught exception while executing generate script CredentialsProviderError: Could not load credentials from any providers"

I'd have to instantiate a client to provide temporary SSO credentials based on the SSO profile I'm using.

[Ain't nobody got time for that.](https://chat.openai.com/share/b960825a-d2d7-4e22-9583-d4cce9dfdfc6)

Stopping here due to SSO support lacking.
