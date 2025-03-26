---
 title: OpenNext for Cloudflare
 date: 2025-03-26
 author: opennextjs
 description: This exists only to facilitate pasting the complete source content into an LLM
---

Go to the 👉 [original source](https://opennextjs.org/docs/cloudflare)

Next.js, unlike Remix, Astro, or the other modern frontends, doesn't have a way to self-host across different platforms. You can run it as a Node.js application, but this doesn't work the same way as it does on Vercel.

There have been several attempts to fix this over the years, broadly falling into two categories: open source framework specific implementations, or closed source product specific implementations.

These can be really hard to maintain separately because Next.js is constantly being updated.

OpenNext is an initiative to bring all these efforts together.

### Get Started

#### New apps

To create a new Next.js app, pre-configured to run on Cloudflare using `@opennextjs/cloudflare`, run:

```sh
npm create cloudflare@latest -- my-next-app --framework=next --experimental
```

#### Existing Next.js apps

##### 1. Install @opennextjs/cloudflare

First, install [@opennextjs/cloudflare(opens in a new tab)](https://www.npmjs.com/package/@opennextjs/cloudflare):

```sh
npm install --save-dev @opennextjs/cloudflare@latest
```

##### 2. Install Wrangler

Install the [Wrangler CLI(opens in a new tab)](https://developers.cloudflare.com/workers/wrangler/) as a devDependency:

```sh
npm install --save-dev wrangler@latest
```

💡 You must use Wrangler version `3.99.0` or later to deploy Next.js apps using `@opennextjs/cloudflare`.

##### 3. Create a wrangler configuration file

ℹ️ This step is optional since `@opennextjs/cloudflare` creates this file for you during the build process (if not already present).

A [wrangler configuration file(opens in a new tab)](https://developers.cloudflare.com/workers/wrangler/configuration/) is needed for your application to be previewed and deployed, it is also where you configure your Worker and define what resources it can access via [bindings(opens in a new tab)](https://developers.cloudflare.com/workers/runtime-apis/bindings/service-bindings).

You can create one yourself in the root directory of your Next.js app with the name `wrangler.jsonc` and the following content:

```jsonc
{
  "$schema": "node_modules/wrangler/config-schema.json",
  "main": ".open-next/worker.js",
  "name": "my-app",
  "compatibility_date": "2024-12-30",
  "compatibility_flags": ["nodejs_compat"],
  "assets": {
    "directory": ".open-next/assets",
    "binding": "ASSETS",
  },
  "kv_namespaces": [
    // Create a KV binding with the binding name "NEXT_CACHE_WORKERS_KV"
    // to enable the KV based caching:
    // {
    //   "binding": "NEXT_CACHE_WORKERS_KV",
    //   "id": "<BINDING_ID>"
    // }
  ]
}
```

💡 As shown above:
- You must enable the [nodejs_compat compatibility flag(opens in a new tab)](https://developers.cloudflare.com/workers/runtime-apis/nodejs/) *and* set your [compatibility date(opens in a new tab)](https://developers.cloudflare.com/workers/configuration/compatibility-dates/) to `2024-09-23` or later, in order for your Next.js app to work with @opennextjs/cloudflare
- The `main` and `assets` values should also not be changed unless you modify the build output result in some way
- You can add a binding named `NEXT_CACHE_WORKERS_KV` to make use of Next.js' caching as described in the [Caching docs](/cloudflare/caching)

#### 4. Add an `open-next.config.ts` file

ℹ️ This step is optional since `@opennextjs/cloudflare` creates this file for you during the build process (if not already present).

Add a [`open-next.config.ts`(opens in a new tab)](https://opennext.js.org/aws/config) file to the root directory of your Next.js app:

```ts
import { defineCloudflareConfig } from "@opennextjs/cloudflare";
import kvIncrementalCache from "@opennextjs/cloudflare/kv-cache";

export default defineCloudflareConfig({
  incrementalCache: kvIncrementalCache,
});
```

💡 To use the `OpenNextConfig` type as illustrated above (which is not necessary), you need to install the `@opennextjs/aws` NPM package as a dev dependency.

#### 5. Add a `.dev.vars` file

Then, add a [`.dev.vars` (opens in a new tab)](https://developers.cloudflare.com/workers/testing/local-development/#local-only-environment-variables) file to the root directory of your Next.js app:

```text
NEXTJS_ENV=development
```

The `NEXTJS_ENV` variable defines the environment to use when loading Next.js `.env` files. It defaults to "production" when not defined.

#### 6. Update the `package.json` file

Add the following to the scripts field of your `package.json` file:

```json
"preview": "opennextjs-cloudflare && wrangler dev",
"deploy": "opennextjs-cloudflare && wrangler deploy",
"cf-typegen": "wrangler types --env-interface CloudflareEnv cloudflare-env.d.ts",
```

- `npm run preview`: Builds your app and serves it locally, allowing you to quickly preview your app running locally in the Workers runtime, via a single command.
- `npm run deploy`: Builds your app, and then deploys it to Cloudflare
- `cf-typegen`: Generates a `cloudflare-env.d.ts` file at the root of your project containing [the types for the `env`(opens in a new tab)](https://developers.cloudflare.com/workers/wrangler/commands/#types).

#### 7. Add caching with Workers KV

See the [Caching docs](/cloudflare/caching) for information on enabling Next.js caching in your OpenNext project.

#### 8. Remove any `export const runtime = "edge";` if present

Before deploying your app, remove the `export const runtime = "edge";` line from any of your source files.

The edge runtime is not supported yet with `@opennextjs/cloudflare`.

#### 9. Add `.open-next` to `.gitignore`

You should add `.open-next` to your `.gitignore` file to prevent the build output from being committed to your repository.

#### 10. Remove `@cloudflare/next-on-pages` (if necessary)

If your Next.js app currently uses `@cloudflare/next-on-pages`, you'll want to remove it, and make a few changes.

Uninstalling the [`@cloudflare/next-on-pages`(opens in a new tab)](https://www.npmjs.com/package/@cloudflare/next-on-pages) package as well as the [`eslint-plugin-next-on-pages`(opens in a new tab)](https://www.npmjs.com/package/eslint-plugin-next-on-pages) package if present.

Remove any reference of these packages from your source and configuration files. This includes:

- `setupDevPlatform()` calls in your Next.js config file
- `getRequestContext` imports from `@cloudflare/next-on-pages` from your source files (those can be replaced with `getCloudflareContext` calls from `@opennextjs/cloudflare`)
- next-on-pages eslint rules set in your Eslint config file

#### 11. Develop locally

You can continue to run `next dev` when developing locally.

Modify your Next.js configuration file to import and call the `initOpenNextCloudflareForDev` utility from the `@opennextjs/cloudflare` package. This makes sure that the Next.js dev server can optimally integrate with the open-next cloudflare adapter and it is necessary for using bindings during local development.

This is an example of a Next.js configuration file calling the utility:

```ts
// next.config.ts
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  /* config options here */
};

export default nextConfig;

import { initOpenNextCloudflareForDev } from "@opennextjs/cloudflare";
initOpenNextCloudflareForDev();
```

After having added the `initOpenNextCloudflareForDev()` call in your Next.js configuration file, you will be able, during local development, to access in any of your server code, local versions of Cloudflare bindings as indicated in the [bindings documentation](/cloudflare/bindings).

In step 3, we also added the `npm run preview`, which allows you to quickly preview your app running locally in the Workers runtime, rather than in Node.js. This allows you to test changes in the same runtime as your app will run in when deployed to Cloudflare.

#### 12. Deploy to Cloudflare Workers

Either deploy via the command line:

```sh
npm run deploy
```

Or [connect a Github or Gitlab repository(opens in a new tab)](https://developers.cloudflare.com/workers/ci-cd/), and Cloudflare will automatically build and deploy each pull request you merge to your production branch.

## Bindings

[Bindings(opens in a new tab)](https://developers.cloudflare.com/workers/runtime-apis/bindings/) allow your Worker to interact with resources on the Cloudflare Developer Platform. When you declare a binding on your Worker, you grant it a specific capability, such as being able to read and write files to an [R2(opens in a new tab)](https://developers.cloudflare.com/r2/) bucket.

### How to configure your Next.js app so it can access bindings

Install [@opennextjs/cloudflare(opens in a new tab)](https://www.npmjs.com/package/@opennextjs/cloudflare), and then add a [wrangler configuration file(opens in a new tab)](https://developers.cloudflare.com/workers/wrangler/configuration/) in the root directory of your Next.js app, as described in [Get Started](/cloudflare/get-started#3-create-a-wranglerjson-file).

### How to access bindings in your Next.js app

You can access [bindings(opens in a new tab)](https://developers.cloudflare.com/workers/runtime-apis/bindings/) from any route of your Next.js app via `getCloudflareContext`:

```js
import { getCloudflareContext } from "@opennextjs/cloudflare";

export async function GET(request) {
  let responseText = "Hello World";

  const myKv = getCloudflareContext().env.MY_KV_NAMESPACE;
  await myKv.put("foo", "bar");
  const foo = await myKv.get("foo");

  return new Response(foo);
}
```

ℹ️ `getCloudflareContext` can only be used in SSG routes in "async mode" (making it return a promise), to run the function in such a way simply provide an options argument with `async` set to `true`:

```js
const context = await getCloudflareContext({ async: true });
```

**WARNING**: During SSG caution is advised since secrets (stored in `.dev.vars` files) and local development values from bindings (like values saved in a local KV) will be used for the pages static generation.

### How to add bindings to your Worker

Add bindings to your Worker by adding them to your [wrangler configuration file(opens in a new tab)](https://developers.cloudflare.com/workers/wrangler/configuration/).

### TypeScript type declarations for bindings

To ensure that the `env` object from `getCloudflareContext().env` above has accurate TypeScript types, run the following Wrangler command to [generate types that match your Worker's configuration(opens in a new tab)](https://developers.cloudflare.com/workers/languages/typescript/#generate-types-that-match-your-workers-configuration-experimental):

```text
npx wrangler types --experimental-include-runtime
```

This will generate a `d.ts` file and (by default) save it to `.wrangler/types/runtime.d.ts`. You will be prompted in the command's output to add that file to your `tsconfig.json`'s `compilerOptions.types` array.

If you would like to commit the file to git, you can provide a custom path. Here, for instance, the `runtime.d.ts` file will be saved to the root of your project:

```bash
npx wrangler types --experimental-include-runtime="./runtime.d.ts"
```

To ensure that your types are always up-to-date, make sure to run `wrangler types --experimental-include-runtime` after any changes to your config file.

### Other Cloudflare APIs (`cf`, `ctx`)

You can access context about the incoming request from the [`cf` object(opens in a new tab)](https://developers.cloudflare.com/workers/runtime-apis/request/#the-cf-property-requestinitcfproperties), as well as lifecycle methods from the [`ctx` object(opens in a new tab)](https://developers.cloudflare.com/workers/runtime-apis/context) from the return value of [`getCloudflareContext()`(opens in a new tab)](https://github.com/opennextjs/opennextjs-cloudflare/blob/main/packages/cloudflare/src/api/get-cloudflare-context.ts):

```js
import { getCloudflareContext } from "@opennextjs/cloudflare";

export async function GET(request) {
  const { env, cf, ctx } = getCloudflareContext();

  // ...
}
```

## Caching

`@opennextjs/cloudflare` supports [caching(opens in a new tab)](https://nextjs.org/docs/app/building-your-application/data-fetching/fetching-caching-and-revalidating#caching-data).

Next.js primes the cache at build time. The build time values are serverd by the [Workers Assets(opens in a new tab)](https://developers.cloudflare.com/workers/static-assets/).

💡 Workers KV is eventually consistent, which means that it can take up to 60 seconds for updates to be reflected globally, when using the default TTL of 60 seconds.

### How to enable caching

`@opennextjs/cloudflare` supports multiple caching mechanisms through a project's OpenNext configuration.

#### Incremental Static Regeneration (ISR)

The ISR adapter for Cloudflare uses [Workers KV(opens in a new tab)](https://developers.cloudflare.com/kv/) as the cache for your Next.js app. Workers KV is [fast(opens in a new tab)](https://blog.cloudflare.com/faster-workers-kv) and uses Cloudflare's [Tiered Cache(opens in a new tab)](https://developers.cloudflare.com/cache/how-to/tiered-cache/) to increase cache hit rates. When you write cached data to Workers KV, you write to storage that can be read by any Cloudflare location. This means your app can fetch data, cache it in KV, and then subsequent requests anywhere around the world can read from this cache. Pricing information can be found in the Cloudflare [docs(opens in a new tab)](https://developers.cloudflare.com/workers/platform/pricing/#workers-kv).

##### 1. Create a KV namespace

```text
npx wrangler@latest kv namespace create <YOUR_NAMESPACE_NAME>
```

##### 2. Add the KV namespace and Service Binding to your Worker

The binding name used in your app's worker is `NEXT_CACHE_WORKERS_KV`. The service binding should be a self reference to your worker where `<WORKER_NAME>` is the name in your wrangler configuration file.

```jsonc
// wrangler.jsonc
{
  // ...
  "kv_namespaces": [
    {
      "binding": "NEXT_CACHE_WORKERS_KV",
      "id": "<BINDING_ID>",
    },
  ],
  "services": [
    {
      "binding": "NEXT_CACHE_REVALIDATION_WORKER",
      "service": "<WORKER_NAME>",
    },
  ],
}
```

#### 3. Configure the cache

In your project's OpenNext config, enable the KV cache and set up a queue.

The memory queue will send revalidation requests to a page when needed, and offers support for de-duplicating requests on a per-isolate basis. There might still be duplicate requests under high traffic or across regions.

⚠️ The memory queue provided by `@opennextjs/cloudflare` is not fully suitable for production deployments, you can use it at your own risk!

```ts
// open-next.config.ts
import { defineCloudflareConfig } from "@opennextjs/cloudflare";
import kvIncrementalCache from "@opennextjs/cloudflare/kv-cache";
import memoryQueue from "@opennextjs/cloudflare/memory-queue";

export default defineCloudflareConfig({
  incrementalCache: kvIncrementalCache,
  queue: memoryQueue,
});
```

💡 The `direct` mode for the queue is intended for debugging purposes and is not recommended for use in production. We are actively working on a solution that will be suitable for production.

#### On-Demand Revalidation

The tag revalidation mechanism uses a [Cloudflare D1(opens in a new tab)](https://developers.cloudflare.com/d1/) database as its backing store for information about tags, paths, and revalidation times.

To use on-demand revalidation, you should also follow the [ISR setup steps](/cloudflare/caching#incremental-static-regeneration-isr).

💡 If your app **only** uses the pages router, it does not need to have a tag cache and should skip this step.

##### 1. Create a D1 database and Service Binding

The binding name used in your app's worker is `NEXT_CACHE_D1`. The service binding should be a self reference to your worker where `<WORKER_NAME>` is the name in your wrangler configuration file.

```jsonc
// wrangler.jsonc
{
  // ...
  "d1_databases": [
    {
      "binding": "NEXT_CACHE_D1",
      "database_id": "<DATABASE_ID>",
      "database_name": "<DATABASE_NAME>",
    },
  ],
  "services": [
    {
      "binding": "NEXT_CACHE_REVALIDATION_WORKER",
      "service": "<WORKER_NAME>",
    },
  ],
}
```

##### 2. Create tables for tag revalidations

The D1 tag cache requires two tables; one that keeps a record of the tag/path mappings, and another that tracks revalidation times.

For the tag mappings, the default table name is `tags`, and can be configured by setting the `NEXT_CACHE_D1_TAGS_TABLE` environment variable to a string.

For the revalidation times, the default table name is `revalidations` and can be configured by setting the `NEXT_CACHE_D1_REVALIDATIONS_TABLE` environment variable to a string.

Wrangler can be used to create a table with it's [execute(opens in a new tab)](https://developers.cloudflare.com/d1/wrangler-commands/#d1-execute) option. Ensure that you create a table for both your local dev database and your remote database.

```sh
wrangler d1 execute NEXT_CACHE_D1 --command "CREATE TABLE IF NOT EXISTS tags (tag TEXT NOT NULL, path TEXT NOT NULL, UNIQUE(tag, path) ON CONFLICT REPLACE)"
wrangler d1 execute NEXT_CACHE_D1 --command "CREATE TABLE IF NOT EXISTS revalidations (tag TEXT NOT NULL, revalidatedAt INTEGER NOT NULL, UNIQUE(tag) ON CONFLICT REPLACE)"
```

#### 3. Configure the cache

In your project's OpenNext config, enable the KV cache and set up a queue. The queue will send a revalidation request to a page when needed, but it will not dedupe requests.

```ts
// open-next.config.ts
import { defineCloudflareConfig } from "@opennextjs/cloudflare";
import kvIncrementalCache from "@opennextjs/cloudflare/kv-cache";
import d1TagCache from "@opennextjs/cloudflare/d1-tag-cache";
import memoryQueue from "@opennextjs/cloudflare/memory-queue";

export default defineCloudflareConfig({
  incrementalCache: kvIncrementalCache,
  tagCache: d1TagCache,
  queue: memoryQueue,
});
```

#### 4. Initialise the cache during deployments

In order for the cache to be properly initialised with the build-time revalidation data, you need to setup a command that runs as part of your deploy step.

OpenNext will generate an SQL file during the build that can be used to setup your D1 database.

```sh
wrangler d1 execute NEXT_CACHE_D1 --file .open-next/cloudflare/cache-assets-manifest.sql
```

This should be run as part of each deployment to ensure that the cache is being populated with each build's initial revalidation data.

### [NextAuth.js(opens in a new tab)](https://next-auth.js.org/)

💡 Starting from wrangler 4.4.0, the implementation of `createCipheriv` is available and NextAuth.js should work out of the box.

NextAuth.js is an open-source authentication solution for Next.js applications.

### Solving a broken build

💡 This section offers a solution for NextAuth.js v4. Use of v5 is currently blocked by the lack of `createCipheriv` implementation.

NextAuth.js relies on [`createCipheriv`(opens in a new tab)](https://nodejs.org/docs/v22.13.1/api/crypto.html#cryptocreatecipherivalgorithm-key-iv-options) from [`node:crypto`(opens in a new tab)](https://nodejs.org/docs/v22.13.1/api/crypto.html).

`createCipheriv` is not currently implemented by the workerd runtime so apps using NextAuth.js with the default configuration break at build time.

However you can configure NextAuth.js to use custom implementations of the `encode` and `decode` functions that do not use the unimplemented Node APIs. Implementations built on top of [`SubtleCrypto`(opens in a new tab)](https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto) can run on workerd.

The NextAuth.js configuration file should look like:

```js
import { encode, decode } from "@/lib/webcrypto";

export const NEXT_AUTH_CONFIG = {
  // ...
  jwt: {
    encode,
    decode,
  },
};
```

Kudos to Arnav Gupta ([`@arnavgupta00`(opens in a new tab)](https://github.com/arnavgupta00)) for coming up with the solution.
You can find an example of this on his [example repository(opens in a new tab)](https://github.com/arnavgupta00/deployment-cf-workers-prisma-nextauth).

Related issues: [`workers-sdk#206`(opens in a new tab)](https://github.com/opennextjs/opennextjs-cloudflare/issues/206) and [`workerd#3277`(opens in a new tab)](https://github.com/cloudflare/workerd/issues/3277).

### Solving issues in local dev

When trying to access Cloudflare bindings depending on your implementation, you might run into:

`ERROR: getCloudflareContext has been called without having called initOpenNextCloudflareForDev from the Next.js config file.`

You can resolve this issue, by calling `getCloudflareContext` in your `NextAuth` callback, for example like so:

```js
export const { handlers, auth, signIn, signOut } = NextAuth(async _ => {
	let { env } = await getCloudflareContext({async: true})
        ...
}

### [Stripe API(opens in a new tab)](https://www.npmjs.com/package/stripe)

When [creating a Stripe object(opens in a new tab)](https://docs.stripe.com/js/initializing), the default http client implementation is based on `node:https` which is not implemented on Workers.

However you can use an http client based on fetch ([`FetchHttpClient`(opens in a new tab)](https://github.com/stripe/stripe-node/blob/54d423e5d1118dc35c4b76260889826003e00e9f/src/net/FetchHttpClient.ts)) via the `httpClient` option:

```ts
import Stripe from "stripe";

const stripe = Stripe(STRIPE_API_KEY, {
  // Cloudflare Workers use the Fetch API for their API requests.
  httpClient: Stripe.createFetchHttpClient(),
});

### Development Workflow

The primary purpose of `@opennextjs/cloudflare` is to take a Next.js application, built with standard Next.js tooling, and convert it into a format compatible with Cloudflare Workers.

This code transformation process takes some time, making the adapter less than ideal for active application development, where a very fast feedback loop and other quality-of-life features, such as Hot Module Replacement (HMR), are crucial. Fortunately, Vercel already provides excellent tooling for this workflow, which Next.js developers are likely already familiar with.

We recommend that developers continue using the tools they are already comfortable with for local development and then use `@opennextjs/cloudflare` when they are ready to deploy their applications to the Cloudflare platform.

Let's explore, in more detail, the application development workflow we recommend for the best developer experience.

### Create a new application based on a template

To create a new Next.js app, pre-configured to run on Cloudflare using `@opennextjs/cloudflare`, run:

```bash
npm create cloudflare@latest -- my-next-app --framework=next --experimental
```

### Develop locally using `next dev`

We believe that the best development workflow uses the `next dev` command provided by Next.js.

To access Cloudflare resources using the `getCloudflareContext` API while running `next dev`, you will need to update the Next.js configuration to call `initOpenNextCloudflareForDev`, as shown in the following example:

```ts
// next.config.ts
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  /* config options here */
};

export default nextConfig;

import { initOpenNextCloudflareForDev } from "@opennextjs/cloudflare";
initOpenNextCloudflareForDev();
```

### `wrangler dev` and `wrangler deploy`

After you've finished iterating on your Next.js application with `next dev`, you can convert it to a Cloudflare Worker by running the `opennextjs-cloudflare` command. This will generate the Worker code in the `.open-next` directory.

You can then preview the app locally in the Cloudflare Workers runtime or deploy it to the Cloudflare network.

To preview your worker locally, run the `wrangler dev` command. This creates a local server that runs your worker in the Cloudflare Workers runtime. Testing your worker is important to ensure that it has been properly built and is working as expected.

Once you've tested your worker, You can run `wrangler deploy`, and in a matter of seconds, your masterpiece will be available to all your users around the world.

### Environment variables

This entry describe the most sensible way to handle your environment variables which works well both during local development and once your application is deployed to Cloudflare Workers.

On the Cloudflare platform, your environment variables can be stored in either ["Enviroment variables"(opens in a new tab)](https://developers.cloudflare.com/workers/configuration/environment-variables/) or ["Secrets"(opens in a new tab)](https://developers.cloudflare.com/workers/configuration/secrets/). The difference being that Secrets can not be read back from either the dashboard or the CLI after being created.

### Local development

While there are multiple ways to set environment variables for local development on the Cloudflare platform (adding them to to your [wrangler configuration(opens in a new tab)](https://developers.cloudflare.com/workers/configuration/secrets/) or to a [.dev.vars(opens in a new tab)](https://developers.cloudflare.com/workers/configuration/secrets/) file) that does not play well with the recommended development workflow as they would not be available while using `next dev`.

What you should do instead is to use the Next.js [`.env` files(opens in a new tab)](https://nextjs.org/docs/pages/building-your-application/configuring/environment-variables). By doing so the environment variables will be available on `process.env` both while running `next dev` and when running your app locally on a Worker with `wrangler dev`.

Next.js `.env` files are environment specific. That is a `.env.development` will take precedence over a `.env` file when you use the "development" environment. See the Next.js site for a detailed explanation of the [loading order(opens in a new tab)](https://nextjs.org/docs/pages/building-your-application/configuring/environment-variables).

You should use the `NEXTJS_ENV` environment variable to select the environment to use when running your app locally on a worker, that's how you would select the "development" environment:

```plain
# .dev.vars
NEXTJS_ENV=development
```

The "production" environment is used by default when `NEXTJS_ENV` is not explicitly set.

### Production

`.env` and `.dev.vars` are local files that should not be added to source control. You should instead use the cloudflare dashboard to set your environment variables for production.

Next.js has [2 kinds(opens in a new tab)](https://nextjs.org/docs/pages/building-your-application/configuring/environment-variables#bundling-environment-variables-for-the-browser) of environment variables:

- non-`NEXT_PUBLIC_` variables which are only available on the server
- `NEXT_PUBLIC_` variables on the other hand are available to the browser

Those are called runtime environment variables (non-`NEXT_PUBLIC_`) and buildtime environment variables (`NEXT_PUBLIC_`) on the Cloudflare paltform.

You can set the runtime environment variables (non-`NEXT_PUBLIC_`) by following those [instructions(opens in a new tab)](https://developers.cloudflare.com/workers/configuration/environment-variables/#add-environment-variables-via-the-dashboard). The build time environment variables (`NEXT_PUBLIC_`) should be set in the [Builds Configuration(opens in a new tab)](https://developers.cloudflare.com/workers/ci-cd/builds/configuration/) so that their value can be inlined by the Workers Builds.

### Examples

To create a new Next.js app, pre-configured to run on Cloudflare using `@opennextjs/cloudflare`, run:

```text
npm create cloudflare@latest -- my-next-app --framework=next --experimental
```

### Basic starter projects

Basic example apps are included in the repository for `@opennextjs/cloudflare` package:

- [create-next-app(opens in a new tab)](https://github.com/opennextjs/opennextjs-cloudflare/tree/main/examples/create-next-app) — a Next.js project bootstrapped with [create-next-app(opens in a new tab)](https://nextjs.org/docs/app/api-reference/cli/create-next-app).
- [api(opens in a new tab)](https://github.com/opennextjs/opennextjs-cloudflare/tree/main/examples/api) — a minimal Next.js project with a single API route
- [middleware(opens in a new tab)](https://github.com/opennextjs/opennextjs-cloudflare/tree/main/examples/middleware) — a minimal Next.js project using middleware
- [vercel-blog-starter(opens in a new tab)](https://github.com/opennextjs/opennextjs-cloudflare/tree/main/examples/vercel-blog-starter) — a blog project using SSG

You can use these to understand how to configure your Next.js app to use `@opennextjs/cloudflare`, or refer to [Get Started](/cloudflare/get-started).

### Next.js Commerce Demo

The [Next.js Commerce demo app(opens in a new tab)](https://github.com/vercel/commerce/tree/v1) works with `@opennextjs/cloudflare`. You can view a deployed version of it [here(opens in a new tab)](https://vercel-commerce-on-workers.web-experiments.workers.dev/).

## Projects from the community

### Fully-featured SaaS starter kit

The [Cloudflare Workers SaaS template(opens in a new tab)](https://github.com/LubomirGeorgiev/cloudflare-workers-nextjs-saas-template) is a fully-featured SaaS app that works with `@opennextjs/cloudflare` and utilizes lots of other Cloudflare services.
Here is a list of some of the features it includes:

- Custom authentication with password and Google SSO
  - Session storage in Cloudflare KV
  - Forgot password
  - Change password
  - Change user settings
- Security
  - Protection with Cloudflare Turnstile Captcha
  - Rate Limiting with Cloudflare KV to prevent abuse
  - Validation for all user actions with react-zsa and zod
- Database: Drizzle ORM and Cloudflare D1
- UI: Shadcn, Tailwind CSS and Hero UI (formely NextUI)
- Transactional email templates with react-email and integration with Resend and Brevo
- Dev Experience
  - Completely type safe
  - Comprehensive eslint config
  - Integrated with Cursor AI
    - .cursorrules
    - A markdown project documentation that Cursor can refer to for more context and better responses
  - Detailed documentation for local development and production deployment
  - Automatic deployment using Github Actions and the Wrangler CLI
- And more...

### Troubleshooting

#### Trying to deploy to Cloudflare Pages, instead of Cloudflare Workers?

`@opennextjs/cloudflare` is specifically built for deploying Next.js apps to [Cloudflare Workers(opens in a new tab)](https://developers.cloudflare.com/workers/)

Cloudflare Workers now support the majority of functionality from Cloudflare Pages, and have features that are not yet supported by Cloudflare Pages. Refer to the [Compatibility Matrix(opens in a new tab)](https://developers.cloudflare.com/workers/static-assets/compatibility-matrix/) in the Cloudflare Workers docs.

If you need to deploy to Cloudflare Pages, you can use `@cloudflare/next-on-pages`, and follow the [Cloudflare Pages guides for deploying Next.js apps(opens in a new tab)](https://developers.cloudflare.com/pages/framework-guides/nextjs/).

### "Your Worker exceeded the size limit of 3 MiB"

The Cloudflare Account you are deploying to is on the Workers Free plan, which [limits the size of each Worker to 3 MiB(opens in a new tab)](https://developers.cloudflare.com/workers/platform/limits/#worker-size). When you subscribe to the Workers Paid plan, each Worker can be up to 10 MiB.

### My app fails to build when I import a specific NPM package

First, make sure that the `nodejs_compat` compatibility flag is enabled, and your compatibility date is set to on or after "2024-09-23", in your [wrangler configuration file(opens in a new tab)](https://developers.cloudflare.com/workers/wrangler/configuration/).
Refer to the [Node.js Workers docs(opens in a new tab)](https://developers.cloudflare.com/workers/runtime-apis/nodejs/) for more details on Node.js support in Cloudflare Workers.

Some NPM packages define multiple exports. For example:

```text
"exports": {
    "other": "./src/other.js",
    "node": "./src/node.js",
    "browser": "./src/browser.js",
    "default": "./src/default.js"
},
```

When you use `@opennextjs/cloudflare`, [Wrangler(opens in a new tab)](https://developers.cloudflare.com/workers/wrangler/) bundles your code before running it locally, or deploying it to Cloudflare. Wrangler has to choose which export to use, when you import a module. By default, Wrangler, which uses [esbuild(opens in a new tab)](https://esbuild.github.io/), handles this in a way that is not compatible with some NPM packages.

You may want to modify how Wrangler resolves multiple exports, such that when you import packages, the `node` export, if present, is used. You can do do by defining the following variables in a `.env` file within the root directory of your Next.js app:

```text
WRANGLER_BUILD_CONDITIONS=""
WRANGLER_BUILD_PLATFORM="node"
```

### Migrate from 0.4 (to 0.5)

`@opennextjs/cloudflare@0.5.0` did not introduce any breaking change, meaning that there are no migration steps from `0.4` to `0.5`.
