---
date: 2018-02-24
title: Network connections
description: How to communicate your scene with external servers and APIs.
categories:
  - development-guide
type: Document
---

Your scene can leverage external services that expose APIs, you can use this to obtain updated price data, weather data or any other kind of information exposed by an API.

You can also set up your own external server to aid your scene and serve to synchronize data between your players. This can either be done with a server that exposes a REST API, or with a server that uses WebSockets.

## Call a REST API

Your scene's code can send calls to a REST API to fetch data.

Since the server might take time to send its response, you must execute this command as an [asynchronous function]({{ site.baseurl }}{% post_url /development-guide/2018-02-25-async-functions %}), using `executeTask()`.

```ts
executeTask(async () => {
  try {
    let response = await fetch(callUrl)
    let json = await response.json()
    log(json)
  } catch {
    log("failed to reach URL")
  }
})
```

The fetch command can also include a second optional argument that bundles headers, HTTP method and HTTP body into a single object.

```ts
executeTask(async () => {
  try {
    let response = await fetch(callUrl, {
      headers: { "Content-Type": "application/json" },
      method: "POST",
      body: JSON.stringify(myBody),
    })
    let json = await response.json()
    log(json)
  } catch {
    log("failed to reach URL")
  }
})
```

> Note: The body must be sent as a stringified JSON object.

The fetch command returns a `response` object with the following data:

- `headers`: A `ReadOnlyHeaders` object. Call the `get()` method to obtain a specific header, or the `has()` method to check if a header is present.
- `ok`: Boolean
- `redirected`: Boolean
- `status`: Status code number
- `statusText`: Text for the status code
- `type`: Will have one of the following values: _basic_, _cors_, _default_, _error_, _opaque_, _opaqueredirect_
- `url`: URL that was sent

- `json()`: Obtain the body in JSON format.
- `text()`: Obtain the body as text.

> Note: `json()` and `text()` are mutually exclusive. If you obtain the body of the response in one of the two formats, you can no longer obtain the other from the `response` object.

### Signed requests

You can employ an extra security measure to certify that a request is originating from a player session inside Decentraland. You can send your requests with an additional signature, that is signed using an ephemeral key that the Decentraland session generates for each player based on the player's address. The server receiving the request can then verify that the signed message indeed matches an address that is currently active in-world.

These kinds of security measures are especially valuable when there may be an incentive for a player to abuse the system, to farm tokens or points in a game.

To send a signed request, all you need to do is use the `signedFetch()` function, in exactly the same way as you would use the `fetch()` function. All the same parameters are valid.

```ts
executeTask(async () => {
  try {
    let response = await signedFetch(callUrl, {
      headers: { "Content-Type": "application/json" },
      method: "POST",
      body: JSON.stringify(myBody),
    })
    let json = await response.json()
    log(json)
  } catch {
    log("failed to reach URL")
  }
})
```

The request will include an additional series of headers, containing a signed message and a set of metadata to interpret that. The signed message consists of all the contents of the request encrypted using the player's ephemeral key.

#### Validating a signed request

To make make use of signed requests, the server receiving these should to validate that the signatures match the rest of the request, and that the timestamp that's encoded within the signed message is current.

You can find a simple example of a server performing this task in the following example scene:

[Validate player authenticity](https://github.com/decentraland-scenes/validate-player-authenticity)

## Use WebSockets

You can also send and obtain data from a WebSocket server, as long as this server uses a secured connection with _wss_.

```ts
var socket = new WebSocket("url")

socket.onmessage = function (event) {
  log("WebSocket message received:", event)
}
```

The syntax to use WebSockets is no different from that implemented natively by JavaScript. See the documentation from [Mozilla Web API](https://developer.mozilla.org/en-US/docs/Web/API/WebSocket) for details on how to catch and send messages over WebSockets.

> TIP: One library that simplifies the use of websocket connections and has been proven to work very well with Decentraland is [Colyseus](https://colyseus.io/). It builds a layer of abstraction on top of the websocket connections that makes reacting to changes and storing a consistent game state remotely in the server super easy. You can see it in action in [these examples](https://github.com/decentraland-scenes/Awesome-Repository#colyseus). Several other websocket libraries aren't compatible with the Decentraland SDK.
