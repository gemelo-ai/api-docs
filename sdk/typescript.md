[← Back to main page](../README.md#6-sdk)

# Typescript/Javascript SDK

> ### 📘 Disclaimer
>
> We highly encourage you to use the API through our SDK. If you still want to make requests on your own, there's a caveat to remember. Most of the JavasScript tools to make HTTP requests, interpret the server's response as `plaintext` or `json` by default, but our TTS/VC endpoints return the binary data. You have take that into account to be able to use it.
> In case of [Axios](https://axios-http.com/docs/req_config), you have to configure the `responseType` as `arraybuffer`.
> In (Fetch)[https://developer.mozilla.org/en-US/docs/Web/API/Response/arrayBuffer], you have to use the `response.arrayBuffer()` method.
> For other tools, please refer to the corresponding documentation.
> If you use our SDK, you don't have to worry about it.

> ### ⚠️ Security of using the SDK in browser
>
> The `ClientKey` and `ApiKey` required to authenticate the SDK are meant to be private.
> Packaging them with the frontend application effectively leaks them publicly and allows anyone to use your gemelo.ai Account directly.
> We strongly advise to proxy those requests via your own backend or use the browser SDK only for internal usage.

## Requirements

The JS/TS SDK works both in all of the modern browsers and Node.js >=18.0.0. We support CommonJS and ESModules.

## Installation

`yarn add @charactr/api-sdk`

## SDK initialization

Before you use the SDK, you have to initialize it first with your account's credentials. To get them, please log in to the (Gemelo Platform)[https://app.gemelo.ai/).

## Text-to-Speech

**Getting available voices**

```typescript
const voices = await sdk.tts.getVoices();
```

Refer [here](https://github.com/charactr-platform/charactr-api-sdk-ts/blob/main/lib/types.ts#L6) for the response type.

You can also browse the voices more conveniently on the (Gemelo Platform)[https://app.gemelo.ai/).

**Making TTS requests**

```typescript
const result = await sdk.tts.convert(voiceId, "Hello world");
```

Refer [here](https://github.com/charactr-platform/charactr-api-sdk-ts/blob/main/lib/types.ts#L14) for the response type.

**TTS Simplex Streaming**

Simplex streaming allows you to send the text to the server and receive the converted audio response as a stream through WebSockets. It greatly reduces the latency compared to the traditional REST endpoint and it's not limited by the text's length. To use it, call `sdk.tts.convertStreamSimplex` method:

```typescript
try {
  await sdk.tts.convertStreamSimplex(voiceId, text, {
    onData: (data: ArrayBuffer) => {
      // do anything with the incoming binary data
    },
  });
} catch (error) {
  console.error(error);
}
```

**TTS Duplex Streaming**
This more advanced technique allows you to start a two-way, WebSockets-based stream, where you can asynchronously send text and receive the converted audio. You control when the stream ends.

To use, it, call the sdk.tts.convertStreamDuplex method:

```typescript
async convertStreamDuplex(
  voice: number | Voice,
  cb: TTSStreamDuplexCallbacks
): Promise<TTSStreamDuplex>
```

You may pass callbacks to know when the stream ends/errors and to receive data:

```typescript
export interface TTSStreamDuplexCallbacks {
  onData?: (data: ArrayBuffer) => void;
  onClose?: (event: CloseEvent) => void;
}
```

The method also returns a set of functions to control the stream:

```typescript
export interface TTSStreamDuplex {
  /**
   * sends to the server the text to be converted into audio
   */
  convert: (text: string) => void;
  /**
   * resolves when there were 5 seconds of stream inactivity
   */
  wait: () => Promise<void>;
  /**
   * terminates the websocket connection immediately
   * in most use cases we advise to use the `close()` method instead
   */
  terminate: () => void;
  /**
   * requests the server to close the connection gracefully
   */
  close: () => void;
}
```

Example usage:

```typescript
const stream = await sdk.tts.convertStreamDuplex(voiceId, {
  onData: (data: ArrayBuffer) => {
    // do anything with the incoming binary data
  },
  onClose: (event: CloseEvent) => {
    if (event.code === 1000) {
      // the stream ended successfully
    } else {
      // the stream was closed because of an error
    }
  },
});
stream.convert(text);
stream.close();
```

## Speech to speech (Voice Conversion)

**Getting available voices**

```typescript
const voices = await sdk.vc.getVoices();
```

Refer [here](https://github.com/charactr-platform/charactr-api-sdk-ts/blob/main/lib/types.ts#L6) for the response type.

You can also browse the voices more conveniently on the (Gemelo Platform)[https://app.gemelo.ai/).

**Making VC requests**

```typescript
const inputAudioBlob = new Blob();
const result = await sdk.vc.convert(inputAudioBlob, "Hello world");
```

Refer [here](https://github.com/charactr-platform/charactr-api-sdk-ts/blob/main/lib/types.ts#L14) for the response type.

## Voice Cloning

Voice cloning API allows you to create voice clones from audio, list all created clones, change their name and delete them, as well as using them with tts & vc endpoints.

The only notable difference is, that `voiceType` option needs to be set to `cloned` when doing the convert request:

```typescript
{
  voiceType: "cloned";
}
```

For more, see the relevant code example: [Full Voice Cloning Example](https://github.com/charactr-platform/charactr-api-sdk-ts/blob/main/examples/nodejs/clones.ts).

### Examples

Our SDK comes with a bunch of examples to help you start working with it. Please, refer to the [SDK's README page](https://github.com/charactr-platform/charactr-api-sdk-ts) if you wish to run them.
