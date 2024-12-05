[← Back to main page](../README.md#6-sdk)

# Python SDK

## Installation

```bash
pip install charactr-api-sdk
```

## SDK initialization

Before you use the SDK, you have to initialize it first with your account's credentials. To get them, please login to the [Gemelo Platform](https://app.gemelo.ai/).

```python
from charactr_api import CharactrAPISDK, Credentials

sdk = CharactrAPISDK(Credentials(client_key="xxx", api_key="yyy"))
```

## Text to speech

### Getting available voices

```python
tts_voices = sdk.tts.get_voices()
```

You can also browse the voices more conveniently in the [Gemelo Platform](https://app.gemelo.ai/).

### Making TTS requests

```python
result = sdk.tts.convert(tts_voices[0]["id"], "Hello world")
```

### TTS Simplex Streaming

**Simplex streaming** allows you to send the text to the server and receive the converted audio response as a stream through **WebSockets**. It greatly reduces the latency compared to the traditional REST endpoint and it's not limited by the text's length.

To use it, call `sdk.tts.start_simplex_stream` method:

```python
def on_data(data: bytes) -> None:
    # do anything with the incoming binary data

def on_close(code: int, message: str) -> None:
    if code != 1000:
        # stream was closed with an error
    else:
        # stream was ended successfully

stream = sdk.tts.start_simplex_stream(
    voice_id, text, on_data=on_data, on_close=on_close
)
```

### TTS Duplex Streaming

This more advanced technique allows you to start a _two-way_, _WebSockets_-based stream, where you can asynchronously send text and receive the converted audio. You control when the stream ends.

To use, it, call the `sdk.tts.start_duplex_stream` method. You may pass callbacks to know when the stream ends/errors and to receive data:

```python

start_duplex_stream(voice_id: int,
                    on_data: (bytes) -> None | None = None,
                    on_close: (int, str) -> None | None = None) -> TTSStreamDuplex
```

The method also returns a set of functions to control the stream:

```python
def convert(text: str) -> None:
    """Convert text to speech and receive audio response as stream,
    using the `on_data` callback."""

def wait() -> None:
    """Blocks execution until there was 5 seconds of stream inactivity."""

def terminate() -> None:
    """Ends the stream immediately. In most cases we advise to use `close()` instead."""

def close() -> None:
    """Requests the server to close the connection gracefully."""
```

Example usage:

```python
def on_data(data: bytes) -> None:
    # do anything with the incoming binary data

def on_close(code: int, message: str) -> None:
    if code != 1000:
        # the stream was closed because of an error
    else:
        # the stream ended successfully

stream = sdk.tts.start_duplex_stream(
    voice_id, on_data=on_data, on_close=on_close
)
stream.convert(text1)
stream.convert(text2)
stream.close()
```

## Speech to speech (Voice Conversion)

### Getting available voices

```python
tts_voices = sdk.vc.get_voices()
```

You can also browse the voices more conveniently in the [Gemelo Platform](https://app.gemelo.ai/).

### Making VC requests

```python
result = sdk.vc.convert(vc_voices[0]["id"], audio_input)
```

## Voice Cloning

Voice cloning API allows you to create voice clones from audio, list all created clones, change their name and delete them, as well as using them with tts & vc endpoints.

The only notable difference is, that cloned_voice option needs to be set to True when doing the convert request:

```python
{ "cloned_voice": True }
```

For more, see the relevant code example: [Full Voice Cloning Example](https://github.com/charactr-platform/charactr-api-sdk-python/blob/main/examples/clones.py)

## Examples

Our SDK comes with a bunch of examples to help you start working with it. Please, refer to the [SDK's README page](https://github.com/charactr-platform/charactr-api-sdk-python) if you wish to run them.
