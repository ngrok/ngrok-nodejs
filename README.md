# The ngrok Agent SDK for NodeJS

[![npm.rs][npm-badge]][npm-url]
[![MIT licensed][mit-badge]][mit-url]
[![Apache-2.0 licensed][apache-badge]][apache-url]
[![Continuous integration][ci-badge]][ci-url]

[npm-badge]: https://img.shields.io/npm/v/@ngrok/ngrok.svg
[npm-url]: https://www.npmjs.com/package/@ngrok/ngrok
[mit-badge]: https://img.shields.io/badge/license-MIT-blue.svg
[mit-url]: https://github.com/ngrok/ngrok-rust/blob/main/LICENSE-MIT
[apache-badge]: https://img.shields.io/badge/license-Apache_2.0-blue.svg
[apache-url]: https://github.com/ngrok/ngrok-rust/blob/main/LICENSE-APACHE
[ci-badge]: https://github.com/ngrok/ngrok-nodejs/actions/workflows/ci.yml/badge.svg
[ci-url]: https://github.com/ngrok/ngrok-nodejs/actions/workflows/ci.yml

**Note: This is beta-quality software. Interfaces may change without warning.**

[ngrok](https://ngrok.com) is a globally distributed reverse proxy commonly used for quickly getting a public URL to a
service running inside a private network, such as on your local laptop. The ngrok agent is usually
deployed inside a private network and is used to communicate with the ngrok cloud service.

This is the ngrok agent in library form, suitable for integrating directly into NodeJS
applications. This allows you to quickly build ngrok into your application with no separate process
to manage.

# Installation

The published library is available on
[npm](https://www.npmjs.com/package/@ngrok/ngrok).

```shell
npm install @ngrok/ngrok
```

To verify that the library is correctly installed use the following code, which forwards to `localhost` port `80`:
```jsx
const ngrok = require("@ngrok/ngrok");
(async function() {
  console.log( await ngrok.connect() );
})();
```

# Documentation

A quickstart guide and a full API reference are included in the [ngrok-nodejs API documentation](https://ngrok.github.io/ngrok-nodejs/).

# Quickstart

After you've installed the package, you'll need an Auth Token. Retrieve one on the
[Auth Token page of your ngrok dashboard](https://dashboard.ngrok.com/get-started/your-authtoken)

A minimal code block [using the 'connect' convenience function](https://github.com/ngrok/ngrok-nodejs/blob/main/examples/ngrok-connect-minimal.js), with `authtoken_from_env: true` to use an Auth Token from the `NGROK_AUTHTOKEN` environment variable, and making a connection to `localhost` port `8080`:

```jsx
const ngrok = require("@ngrok/ngrok");
(async function() {
  const url = await ngrok.connect({ addr: 8080, authtoken_from_env: true })
  console.log(`Ingress established at: ${url}`);
})();
```

There are many more examples in [the /examples directory](https://github.com/ngrok/ngrok-nodejs/tree/main/examples).

## Authorization

To use most features of ngrok, you need to obtain an Auth Token. You can get an Auth Token by signing up for free at [ngrok.com](https://dashboard.ngrok.com/signup) and then retrieving it from the [Auth Token page of your ngrok dashboard](https://dashboard.ngrok.com/get-started/your-authtoken). Once you have an Auth Token, you can reference it in several ways.

You can set the authtoken in the environment variable `NGROK_AUTHTOKEN` and then pass `authtoken_from_env: true` to the [connect](https://ngrok.github.io/ngrok-nodejs/functions/connect.html) method:

```jsx
await ngrok.connect({authtoken_from_env: true, ...});
```

You can also set the default Auth Token to use for all connections by calling the [authtoken](https://ngrok.github.io/ngrok-nodejs/functions/authtoken.html) method:

```jsx
await ngrok.authtoken(token);
```

Or the Auth Token can be passed directly to the [connect](https://ngrok.github.io/ngrok-nodejs/functions/connect.html) method:

```jsx
await ngrok.connect({authtoken: token, ...});
```

## Connection

The [connect](https://ngrok.github.io/ngrok-nodejs/functions/connect.html) method is the most common way to use this library. It will start an ngrok session if it is not already running, and then establish a tunnel to the specified address. The [connect](https://ngrok.github.io/ngrok-nodejs/functions/connect.html) method returns a promise that resolves to the public URL of the tunnel.

With just an integer, the `connect` method will forward to `localhost` on the specified port:

```jsx
const url = await ngrok.connect(4242);
```

Or more options can be passed to the `connect` method to customize the connection:

```jsx
const url = await ngrok.connect({addr: 8080, basic_auth: "ngrok:online1line"});
const url = await ngrok.connect({addr: 8080, oauth_provider: "google", oauth_allow_domains: "example.com"});
const url = await ngrok.connect({proto: 'tcp', addr: 25565});
```

## Disconnection

To close a tunnel use the [disconnect](https://ngrok.github.io/ngrok-nodejs/functions/disconnect.html) function with the URL of the tunnel to close:

```jsx
await ngrok.disconnect(url);
```

Or omit the URL to close all tunnels:

```jsx
await ngrok.disconnect();
```

The [close](https://ngrok.github.io/ngrok-nodejs/classes/NgrokTunnel.html#close) method on a tunnel will shut it down, and also stop the ngrok session if it is no longer needed. This method returns a promise that resolves when the tunnel is closed.

```jsx
const tunnel = await ngrok.getTunnelByUrl(url);
await tunnel.close();
```

## Listing Tunnels

To list all current non-closed tunnels use the [tunnels](https://ngrok.github.io/ngrok-nodejs/functions/tunnels.html) function:

```jsx
const tunnels = await ngrok.tunnels();
```

# Full Configuration

The [Config](https://ngrok.github.io/ngrok-nodejs/interfaces/Config.html) interface shows all the available options, or refer to this example of [all the possible configuration items ngrok.connect](https://github.com/ngrok/ngrok-nodejs/blob/main/examples/ngrok-connect-full.js):

```jsx
const url = await ngrok.connect({
  // session configuration
  addr: `localhost:8080`, // or `8080` or `pipe:${UNIX_SOCKET}`
  authtoken: "<authtoken>",
  authtoken_from_env: true,
  on_status_change: (addr, error) => {
    console.log(`disconnected, addr ${addr} error: ${error}`);
  },
  session_metadata: "Online in One Line",
  // tunnel configuration
  basic_auth: ["ngrok:online1line"],
  circuit_breaker: 0.1,
  compression: true,
  domain: "<domain>",
  ip_restriction_allow_cidrs: ["0.0.0.0/0"],
  ip_restriction_deny_cidrs: ["10.1.1.1/32"],
  metadata: "example tunnel metadata from nodejs",
  mutual_tls_cas: [fs.readFileSync('ca.crt', 'utf8')],
  oauth_provider: "google",
  oauth_allow_domains: ["<domain>"],
  oauth_allow_emails: ["<email>"],
  oauth_scopes: ["<scope>"],
  oidc_issuer_url: "<url>",
  oidc_client_id: "<id>",
  oidc_client_secret: "<secret>",
  oidc_allow_domains: ["<domain>"],
  oidc_allow_emails: ["<email>"],
  oidc_scopes: ["<scope>"],
  proxy_proto: "", // One of: "", "1", "2"
  request_header_remove: ["X-Req-Nope"],
  response_header_remove: ["X-Res-Nope"],
  request_header_add: ["X-Req-Yup:true"],
  response_header_add: ["X-Res-Yup:true"],
  schemes: ["HTTPS"],
  verify_webhook_provider: "twilio",
  verify_webhook_secret: "asdf",
  websocket_tcp_converter: true,
});
```

# Examples

Degit can be used for cloning and running an example directory like this:
```bash
npx degit github:ngrok/ngrok-nodejs/examples/<example> <folder-name>
cd <folder-name>
npm i
```
For example:
```bash
npx degit github:ngrok/ngrok-nodejs/examples/express express && cd express && npm i
```

## Frameworks
* [AWS App Runner](https://aws.amazon.com/apprunner/) - See the [ngrok SDK Serverless Example](https://github.com/ngrok/ngrok-sdk-serverless-example) repository
* [Express](https://expressjs.com/) - [Quickstart Example](https://github.com/ngrok/ngrok-nodejs/blob/main/examples/express/ngrok-express-quickstart.js), [Configuration Example](https://github.com/ngrok/ngrok-nodejs/blob/main/examples/express/ngrok-express.js)
* [Fastify](https://www.fastify.io/) - [Example](https://github.com/ngrok/ngrok-nodejs/blob/main/examples/fastify/ngrok-fastify.js)
* [Hapi](https://hapi.dev/) - [Example](https://github.com/ngrok/ngrok-nodejs/blob/main/examples/hapi/ngrok-hapi.js)
* [Koa](https://koajs.com/) - [Example](https://github.com/ngrok/ngrok-nodejs/blob/main/examples/koa/ngrok-koa.js)
* [Nest.js](https://nestjs.com/) - [Example main.ts](https://github.com/ngrok/ngrok-nodejs/blob/main/examples/nestjs/src/main.ts)
* [Next.js](https://nextjs.org/) - [Example next.config.js](https://github.com/ngrok/ngrok-nodejs/blob/main/examples/nextjs/next.config.js) loading [ngrok.config.js](https://github.com/ngrok/ngrok-nodejs/blob/main/examples/nextjs/ngrok.config.js)
* [Remix](https://remix.run/) - [Example remix.config.js](https://github.com/ngrok/ngrok-nodejs/blob/main/examples/remix/remix.config.js) loading [ngrok.config.js](https://github.com/ngrok/ngrok-nodejs/blob/main/examples/remix/ngrok.config.js)
* [Svelte](https://svelte.dev/) - [Example svelte.config.js](https://github.com/ngrok/ngrok-nodejs/blob/main/examples/svelte/svelte.config.js) (works in vite.config.js too) loading [ngrok.config.cjs](https://github.com/ngrok/ngrok-nodejs/blob/main/examples/svelte/ngrok.config.cjs)
* [Typescript](https://www.typescriptlang.org/) - [Example ts-node](https://github.com/ngrok/ngrok-nodejs/blob/main/examples/ngrok-typescript.ts)
* [Vue](https://vuejs.org/) - [Example vite.config.ts](https://github.com/ngrok/ngrok-nodejs/blob/main/examples/vue/vite.config.ts) loading [ngrok.config.ts](https://github.com/ngrok/ngrok-nodejs/blob/main/examples/vue/ngrok.config.ts)
* [Winston](https://github.com/winstonjs/winston#readme) Logging - [Example](https://github.com/ngrok/ngrok-nodejs/blob/main/examples/ngrok-winston.js)

## Tunnel Types
* ngrok.connect - [ngrok.connect Minimal Example](https://github.com/ngrok/ngrok-nodejs/blob/main/examples/ngrok-connect-minimal.js), [Full ngrok.connect Example](https://github.com/ngrok/ngrok-nodejs/blob/main/examples/ngrok-connect-full.js)
* HTTP - [ngrok.listen Example](https://github.com/ngrok/ngrok-nodejs/blob/main/examples/ngrok-listen.js), [Minimal Example](https://github.com/ngrok/ngrok-nodejs/blob/main/examples/ngrok-http-minimum.js), [Full Configuration Example](https://github.com/ngrok/ngrok-nodejs/blob/main/examples/ngrok-http-full.js)
* Labeled - [Example](https://github.com/ngrok/ngrok-nodejs/blob/main/examples/ngrok-labeled.js)
* TCP - [Example](https://github.com/ngrok/ngrok-nodejs/blob/main/examples/ngrok-tcp.js)
* TLS - [Example](https://github.com/ngrok/ngrok-nodejs/blob/main/examples/ngrok-tls.js)
* Windows Pipe - [Example](https://github.com/ngrok/ngrok-nodejs/blob/main/examples/ngrok-windows-pipe.js)

# Builders

For more control over Sessions and Tunnels, the builder classes can be used.

A minimal builder use-case looks like [the following](https://github.com/ngrok/ngrok-nodejs/blob/main/examples/ngrok-http-minimum.js):

```jsx
async function create_tunnel() {
  const session = await new ngrok.NgrokSessionBuilder().authtokenFromEnv().connect();
  const tunnel = await session.httpEndpoint().listen();
  console.log("Ingress established at:", tunnel.url());
  tunnel.forwardTcp("localhost:8081");
}
```

See here for a [Full Configuration Example](https://github.com/ngrok/ngrok-nodejs/blob/main/examples/ngrok-http-full.js)

# Async Programming

All methods return a `Promise` and are suitable for use in asynchronous
programming. You can use callback chaining with `.then()` and `.catch()` syntax
or the `await` keyword to wait for completion of an API call.

### Error Handling

The asynchronous functions will throw an error on failure to set up a session or tunnel,
which can be caught and dealt with using standard then/catch semantics.

```jsx
new ngrok.NgrokSessionBuilder().authtokenFromEnv().connect()
    .then((session) => {
        session.httpEndpoint().listen()
            .then((tun) => {})
            .catch(err => console.log('tunnel setup error: ' + err))
    })
    .catch(err => console.log('session setup error: ' + err))
    .await;
```

# Platform Support

Pre-built binaries are provided on NPM for the following platforms:

| OS         | i686 | x64 | aarch64 | arm |
| ---------- | -----|-----|---------|-----|
| Windows    |   ✓  |  ✓  |    *    |     |
| MacOS      |      |  ✓  |    ✓    |     |
| Linux      |      |  ✓  |    ✓    |  ✓  |
| Linux musl |      |  ✓  |    ✓    |     |
| FreeBSD    |      |  ✓  |         |     |
| Android    |      |     |    ✓    |  ✓  |

ngrok-nodejs, and [ngrok-rust](https://github.com/ngrok/ngrok-rust/) which it depends on, are open source, so it may be possible to build them for other platforms.

* Windows-aarch64 will be supported after the next release of [Ring](https://github.com/briansmith/ring/issues/1167).

# Dependencies

This project relies on [NAPI-RS](https://napi.rs/), an excellent system to ease development and building of Rust plugins for NodeJS.

# License

This project is licensed under either of

 * Apache License, Version 2.0, ([LICENSE-APACHE](LICENSE-APACHE) or
   http://www.apache.org/licenses/LICENSE-2.0)
 * MIT license ([LICENSE-MIT](LICENSE-MIT) or
   http://opensource.org/licenses/MIT)

at your option.

### Contribution

Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in ngrok-nodejs by you, as defined in the Apache-2.0 license, shall be
dual licensed as above, without any additional terms or conditions.
