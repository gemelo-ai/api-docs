# Gemelo API

## Table of contents

- [1. Introduction](#1-introduction)
- [2. Obtaining your API Key](#2-obtaining-your-api-key)
- [3. API Reference](#3-api-reference)
- [4. Making your first requests](#4-making-your-first-requests)
- [5. Text-to-Speech streaming](#5-text-to-speech-streaming)
- [6. SDK](#6-sdk)
- [7. Examples](#7-examples)
- [8. Support](#8-support)

## 1. Introduction

Gemelo API empowers you to create personalized video twins, allowing you to clone yourself and seamlessly generate custom content with your digital double. In addition to video cloning, we offer advanced voice cloning technology, enabling your digital twin to speak naturally and expressively. Our AI also provides access to the world’s most expressive, unique, and lifelike synthetic voices, perfect for enhancing any app, game, website, or media project.

## 2. Obtaining your API Key

To use the API, you need first to obtain the API Key.

To get your key, please visit our [Gemelo Platform](https://app.gemelo.ai) and make a new account. If you have any questions, feel free to [contact us](https://gemelo.ai/contact).

> ⚠️ API Keys are meant to be private. You should not disclose them publicly or package them with your frontend/mobile application.

## 3. API Reference

Our complete API reference is available through Swagger and hosted on GitHub Pages. This documentation provides detailed information about the endpoints, request and response structures, error codes, and examples for integrating with our API.

To explore the API and try out requests directly in the browser, visit the API reference at:

[API Documentation](https://gemelo-ai.github.io/api-docs/)

This interactive documentation allows you to:

- View all available API endpoints and their descriptions.
- Test requests directly from the browser.
- Get sample request and response payloads.
- Understand possible errors and troubleshooting tips.

For a quick start guide or more examples of how to use the API, check out the detailed guides and code snippets below in this README.

## 4. Making your first requests

Once you have acquired your keys, you can begin making requests. For each request you make, you have to include your Client ID in the X-Client-ID header, and the API Key in the X-API-Key header respectively:

```bash
curl -i -X POST \
   -H "X-Client-Key:xxxxxxxxxx-xxxxxxxxxx-xxxxxxxxxx" \
   -H "X-Api-Key:yyyyyyyyyy-yyyyyyyyyy-yyyyyyyyyy" \
 'https://api.gemelo.ai/v1/endpoint'
```

Before you request TTS or VC, you have to query the available voices first, and include the chosen voice's ID in the subsequent request.

### 4.1. Video API Examples

TBA

### 4.2. Text-to-Speech Example

**Querying TTS voices**

```bash
curl -i -X GET \
   -H "X-Client-Key:xxxxxxxxxx-xxxxxxxxxx-xxxxxxxxxx" \
   -H "X-Api-Key:yyyyyyyyyy-yyyyyyyyyy-yyyyyyyyyy" \
   'https://api.gemelo.ai/v1/tts/voices'
```

**Available voices response (example)**

```json
[
  {
    "id": 24,
    "name": "John Doe",
    "description": "Hoarse voice of an elderly man."
  },
  {
    "id": 35,
    "name": "Jane Doe",
    "description": "Warm and friendly female voice."
  }
]
```

**Actual Text-to-Speech Request**

```bash
curl -X POST \
   -H "X-Client-Key:xxxxxxxxxx-xxxxxxxxxx-xxxxxxxxxx" \
   -H "X-Api-Key:yyyyyyyyyy-yyyyyyyyyy-yyyyyyyyyy" \
   -d '{"voiceId": 35, "text": "Hello world!"}' \
   -o output.wav \
   'https://api.gemelo.ai/v1/tts/convert'
```

Running this will save the response as `output.wav` file in the active location.

### 4.3. Speech-to-Speech Example

**Querying TTS voices**

```bash
curl -i -X GET \
   -H "X-Client-Key:xxxxxxxxxx-xxxxxxxxxx-xxxxxxxxxx" \
   -H "X-Api-Key:yyyyyyyyyy-yyyyyyyyyy-yyyyyyyyyy" \
   'https://api.gemelo.ai/v1/vc/voices'
```

**Available voices response (example)**

```json
[
  {
    "id": 24,
    "name": "John Doe",
    "description": "Hoarse voice of an elderly man."
  },
  {
    "id": 35,
    "name": "Jane Doe",
    "description": "Warm and friendly female voice."
  }
]
```

**Actual VC request**

Make sure that the following params are correctly filled:

- Client Key
- API Key
- Input file location (`audio.wav` in the example)
- voiceID (in the URL query params)

```bash
curl -X POST \
     -H "Content-Type: multipart/form-data" \
     -H "X-Client-Key: xxx" \
     -H "X-Api-Key: yyy" \
     -F "file=@audio.wav" \
     -o output.wav \
     "https://api.gemelo.ai/v1/vc/convert?voiceId=133"
```

Running this will save the response as output.wav file in the active location.

## 5. Text-to-Speech streaming

Our WebSockets-based TTS Streaming solution allows you to reduce the latency of Text-to-Speech conversion significantly, and it's not limited by the text's length.

We support two kinds of streaming:

- TTS Simplex streaming - allows you to send the text to the server once and receive the converted audio response as a stream.
- TTS Duplex streaming - allows you to asynchronously send text to the server multiple times and receive back the converted audio. You control when the stream ends.

Both kinds are supported in our SDKs. As long as you use them, you don't need to worry about the low-level details presented in this article below.

### 5.1. TTS Simplex streaming

First, connect with the WebSocket using the endpoint below. You need to specify the voice you want to use in the query parameter.

`wss://api.gemelo.ai/v1/tts/stream/simplex/ws?voiceId=xxx`

Next, you send the following message to authenticate in the service:

```json
{
  "type": "authApiKey",
  "clientKey": "xxx",
  "apiKey": "yyy"
}
```

Then, you send another message with the text to convert:

```json
{
  "type": "convert",
  "text": "xxx"
}
```

Now you listen for the incoming raw binary data. You should not close the connection by yourself; the server will do so as soon as it finishes converting the text.

### 5.2. TTS Duplex streaming

Similarly to the simplex streaming, you need to connect with the WebSocket and choose the voice, however the endpoint is different:

`wss://api.gemelo.ai/v1/tts/stream/duplex/ws?voiceId=xxx`

Now, send the authentication command:

```json
{
  "type": "authApiKey",
  "clientKey": "xxx",
  "apiKey": "yyy"
}
```

From now on, you can asynchronously listen for incoming raw binary data and send the convert command to the server as many times as you want:

```
{
  "type": "convert",
  "text": "xxx"
}
```

When you are done, you can finish the stream by sending the `close` command. The server will flush the buffer and gracefully close the connection (you will receive a WebSocket close event when it does):

```
{
  "type": "close"
}
```

Alternatively, you can close the WebSocket connection manually (depending on the language you are using).

## 6. SDK

> ℹ️ Our current SDK supports only voice features, including AI-generated voices and voice cloning. Video twin functionality is not yet available within the SDK but is planned for a future release.

- [Typescript/Javascript SDK](sdk/typescript.md)
- [Python SDK](sdk/python.md)
- [Go SDK](sdk/go.md)

## 7. Examples

- [ChatGPT Voice Response](examples/chatgpt-voice-response.md)
- [ChatGPT Dialogue System](examples/chatgpt-dialogue-system.md)

## 8. Support

If you have any troubles using our API, please contact us via the form at [gemelo.ai/contact](https://gemelo.ai/contact), or report an issue using GitHub.
