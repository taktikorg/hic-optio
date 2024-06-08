<p align="center" width="100%">
    <img width="55%" src="./assets/hyperfetch.png">
</p>

> :warning: Currently still tested with SSR

> Latest release 0.1.0: Fetch for Upload File is supported, check our example https://github.com/taktikorg/hic-optio/blob/cbf34afa892e9231e64d5f4c8158c041a9425f36/src/Hyperfetch.test.ts#L50C1-L81C6

Supertiny (4kB minified & 0 dependencies) and strong-typed HTTP client for Deno, Bun, Node.js, Cloudflare Workers and Browsers.

```sh
# Node.js
npm install @taktikorg/hic-optio
# Bun
bun install @taktikorg/hic-optio
```

The idea of this tool is to provide lightweight `fetch` wrapper for Node.js, Bun:

```js
import @taktikorg/hic-optio from "@taktikorg/hic-optio";

const @taktikorg/hic-optioRequest = @taktikorg/hic-optio.createRequest("https://jsonplaceholder.typicode.com"); // Pass true for DEBUG mode

// Example usage of POST method with retry and timeout
const [postErr, postData] = await @taktikorg/hic-optioRequest.post(
  "/posts",
  { retries: 3, timeout: 5000 },
  {
    title: "foo",
    body: "bar",
    userId: 1,
  }
);

if (postErr) {
  console.error("POST Error:", postErr);
} else {
  console.log("POST Data:", postData);
}
```

Cloudflare Workers:

```ts
export default {
  async fetch(
    request: Request,
    env: Env,
    ctx: ExecutionContext
  ): Promise<Response> {
    const @taktikorg/hic-optioInstance = await @taktikorg/hic-optio.createRequest(
      "https://jsonplaceholder.typicode.com"
    );

    const [getErr, getData] = await @taktikorg/hic-optioInstance.get<
      Array<{
        userId: number;
        id: number;
        title: string;
        body: string;
      }>
    >("/posts");

    if (getErr) {
      console.error("GET Error:", getErr);
    }

    return Response.json(getData);
  },
};
```

and Browsers:

```html
<script src="https://unpkg.com/@taktikorg/hic-optio/dist/hyperfetch-browser.min.js"></script>

<script>
  (async () => {
    const request = @taktikorg/hic-optio.default.createRequest(
      "https://jsonplaceholder.typicode.com"
    );
  })();
</script>
```

# Why Hyperfetch?

## Simple Core

We define things easier with composed functions, ofcourse contribute easier.

```js
get: (url, options, data) => httpMethodFunction(url, options)('GET', options, data),
post: (url, options, data) => httpMethodFunction(url, options)('POST', options, data),
put: (url, options, data) => httpMethodFunction(url, options)('PUT', options, data),
delete: (url, options, data) => httpMethodFunction(url, options)('DELETE', options, data),
patch: (url, options, data) => httpMethodFunction(url, options)('PATCH', options, data),
options: (url, options, data) => httpMethodFunction(url, options)('OPTIONS', options, data),
getAbortController,
```

## Error Handling

No need to write `try..catch` ! @taktikorg/hic-optio do it like this:

```js
const @taktikorg/hic-optioRequest = @taktikorg/hic-optio.createRequest("https://jsonplaceholder.typicode.com";

// Example usage of POST method with retry and timeout
const [postErr, postData] = await @taktikorg/hic-optioRequest.post(
    '/posts',
    { retries: 3, timeout: 5000 },
    {
        title: 'foo',
        body: 'bar',
        userId: 1,
    }
);

if (postErr) {
    console.error('POST Error:', postErr);
} else {
    console.log('POST Data:', postData);
}
```

## Hooks

Hooks is supported and expected to not modifying the original result by design.

```js
const hooks = {
  preRequest: (url, options) => {
    console.log(`Preparing to send request to: ${url}`);
    // You can perform actions before the request here
  },
  postRequest: (url, options, data, response) => {
    console.log(
      `Request to ${url} completed with status: ${
        response?.[0] ? "error" : "success"
      }`
    );
    // You can perform actions after the request here, including handling errors
  },
};

const requestWithHooks = @taktikorg/hic-optio.createRequest(
  "https://jsonplaceholder.typicode.com",
  hooks,
  true
); // pass true for DEBUG mode

// Example usage of POST method with retry and timeout
const [postErr, postData] = await requestWithHooks.post(
  "/posts",
  { retries: 3, timeout: 5000 },
  {
    title: "foo",
    body: "bar",
    userId: 1,
  }
);
```

List of Hooks:

```ts
export interface Hooks {
  preRequest?: (url: string, options: RequestOptions) => void;
  postRequest?: <T, U>(
    url: string,
    options: RequestOptions,
    data?: T,
    response?: [Error | null, U]
  ) => void;
  preRetry?: (
    url: string,
    options: RequestOptions,
    retryCount: number,
    retryLeft: number
  ) => void;
  postRetry?: <T, U>(
    url: string,
    options: RequestOptions,
    data?: T,
    response?: [Error | null, U],
    retryCount?: number,
    retryLeft?: number
  ) => void;
  preTimeout?: (url: string, options: RequestOptions) => void;
  postTimeout?: (url: string, options: RequestOptions) => void;
}
```

## Retry Mechanism

You can retry your request once it's failed!

```js
const [postErr, postData] = await requestWithHooks.post(
  "/posts",
  { retries: 3, timeout: 5000 },
  {
    title: "foo",
    body: "bar",
    userId: 1,
  }
);
```

Jitter and backoff also supported. 😎

```js
const [postErr, postData] = await requestWithHooks.post(
  "/posts",
  { retries: 3, timeout: 5000, jitter: true }, // false `jitter` to use backoff
  {
    title: "foo",
    body: "bar",
    userId: 1,
  }
);
```

You can modify backoff and jitter factor as well.

```js
const [postErr, postData] = await requestWithHooks.post(
  "/posts",
  { retries: 3, timeout: 5000, jitter: true, jitterFactor: 10000 }, // false `jitter` to use backoff
  {
    title: "foo",
    body: "bar",
    userId: 1,
  }
);

// or backoff

const [postErr, postData] = await requestWithHooks.post(
  "/posts",
  { retries: 3, timeout: 5000, jitter: false, backoffFactor: 10000 }, // false `jitter` to use backoff
  {
    title: "foo",
    body: "bar",
    userId: 1,
  }
);
```

Retry on timeout also supported.

```js
const [postErr, postData] = await requestWithHooks.post(
  "/posts",
  { retries: 3, timeout: 5000, retryOnTimeout: true },
  {
    title: "foo",
    body: "bar",
    userId: 1,
  }
);
```

## Infer Response Types

```ts
const [getErr, getData] = await @taktikorg/hic-optioRequest.get<
  Array<{
    userId: number;
    id: number;
    title: string;
    body: string;
  }>
>("/posts", {
  retries: 3,
  timeout: 5000,
});

getData?.[0]?.id; // number | undefined
```

### URLSearchParams

```ts
const [getErr, getData] = await @taktikorg/hic-optioRequest.get("/posts", {
  retries: 3,
  timeout: 5000,
  params: {
    id: 1,
  },
}); // /posts?id=1
```

### Form Data

Example usecase for Upload File:

```ts
export async function postImportFile(formData: FormData) {
  const [postErr, postData] = await @taktikorg/hic-optioRequest.post(
    `/api/upload-file/import`,
    {
      body: formData,
      credentials:
        import.meta.env.VITE_USER_NODE_ENV === "development"
          ? "same-origin"
          : "include",
    }
  );

  if (postErr) {
    throw postErr;
  }

  return postData;
}

```

### Applications Knowledges

#### Constant

```js
const DEFAULT_MAX_TIMEOUT = 2147483647;
const DEFAULT_BACKOFF_FACTOR = 0.3;
const DEFAULT_JITTER_FACTOR = 1;
```

#### AbortController

We expose abort controller, you can cancel next request anytime.

```js
// DELETE will not work if you uncomment this
const controller = requestWithHooks.getAbortController();

controller.abort();

// Example usage of DELETE method with retry and timeout
const [deleteErr, deleteData] = await requestWithHooks.delete("/posts/1", {
  retries: 3,
  timeout: 5000,
});

if (deleteErr) {
  console.error("DELETE Error:", deleteErr);
} else {
  console.log("DELETE Data:", deleteData);
}
```

---

License MIT 2024
