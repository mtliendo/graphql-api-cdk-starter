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
- The `scripts/generate.mjs` file is used to essentially call the Amplify CLI (`npx @aws-amplify/cli codegen add`) and create the `schema.graphqlconfig.yml` file.
- The `backend/app.ts` file creates a stack, and an AppSync API. This API has 4 Datasources: 3 as DynamoDB Tables and 1 as a Lambda Function (stubbed out in `backend/repeat.ts`).
- The `src` directory is the Create React App frontend and is written in TypeScript that makes use of Amplify Generics, UI Library, and yet-to-be-generated operations.
- Instead of an `aws-exports.js` file, it looks like there is a `./appConfig` file. I like this naming.

## Getting Started

Hmmm...I'm actually not sure where to start here. The `README.md` file asks for there to be `amplify` version >=12 installed. It's worth noting that most CDK users don't want to use the Amplify CLI and will instead opt to run Amplify commands from `npx`.
