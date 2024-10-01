[← Back to main page](../README.md#6-sdk)

# Go SDK

## Installation

```shell
go get github.com/charactr-platform/charactr-api-sdk-go/v2
```

## SDK initialization

Before you use the SDK, you have to initialize it first with your account's credentials. To get them, please login to the [Gemelo Platform](https://app.gemelo.ai/).

```go
import "github.com/charactr-platform/charactr-api-sdk-go/v2"

func main() {
  sdk := charactr.New(&charactr.Credentials{ClientKey: "", APIKey: ""})
}
```

## Text to speech

### Getting available voices

```go
voices, err := sdk.TTS.GetVoices(context.TODO())
if err != nil {
  panic(err)
}
```

You can also browse the voices more conveniently in the [Gemelo Platform](https://app.gemelo.ai/).

### Making TTS requests

```go
result, err := sdk.TTS.Convert(context.TODO(), voices[0].ID, "Hello world")
if err != nil {
  panic(err)
}
```

### TTS Simplex Streaming

**Simplex streaming** allows you to send the text to the server and receive the converted audio response as a stream through **WebSockets**. It greatly reduces the latency compared to the traditional REST endpoint and it's not limited by the text's length.

To use it, call `sdk.TTS.StartSimplexStream` method:

```go
stream, err := sdk.TTS.StartSimplexStream(ctx, voiceID, text)
if err != nil {
  panic(err)
}

g, ctx := errgroup.WithContext(ctx)

g.Go(func() error {
  for {
    audioBytes, err := stream.Read()
    if err != nil {
      if err == io.EOF { // normal closure
        return nil
      }
      return err
    }

    // do anything with the incoming binary data
  }
})

err = g.Wait()
if err != nil {
  // stream was closed with error
  panic(err)
} else {
  // stream has ended successfully
}
```

### TTS Duplex Streaming

This more advanced technique allows you to start a **two-way**, **WebSockets**-based stream, where you can asynchronously send text and receive the converted audio. You control when the stream ends.

To use, it, call the `sdk.TTS.StartDuplexStream` method:

```go
func StartDuplexStream(ctx context.Context, voiceID int) (\*DuplexStream, error)
```

The method also returns a set of functions to control the stream:

```go
// Read returns the raw audio data converted by the server
func (v \*DuplexStream) Read() ([]byte, error)

// Convert asynchronously feeds the stream with the text to be converted to audio
func (v \*DuplexStream) Convert(text string) error

// Wait will block the execution until there was 5 seconds of stream inactivity
func (v \*DuplexStream) Wait()

// Close requests the server to close the connection gracefully
func (v \*DuplexStream) Close()

// Terminate ends the stream immediately. In most cases we advise to use Close() instead.
func (v \*DuplexStream) Terminate()
```

Example usage:

```go
stream, err := sdk.TTS.StartDuplexStream(ctx, voiceID)
if err != nil {
  panic(err)
}

g, ctx := errgroup.WithContext(ctx)

g.Go(func() error {
  for {
    audioBytes, err := stream.Read()
    if err != nil {
      if err == io.EOF { // normal closure
        return nil
      }
      return err
    }

    // do anything with the incoming binary data
  }
})

stream.Convert(text1)
stream.Convert(text2)

err = stream.Close()
if err != nil {
  // there was a problem with closing the stream
  panic(err)
}

err = g.Wait()
if err != nil {
  // stream was closed with error
  panic(err)
} else {
  // stream ended successfully
}
```

## Speech to speech (Voice Conversion)

Getting available voices

```go
voices, err := sdk.VC.GetVoices(context.TODO())
if err != nil {
  panic(err)
}
```

You can also browse the voices more conveniently in the [Gemelo Platform](https://app.gemelo.ai/).

### Making VC requests

```go
result, err := sdk.VC.Convert(context.TODO(), voices[0].ID, file)
if err != nil {
  panic(err)
}
```

## Voice Cloning

Voice cloning API allows you to create voice clones from audio, list all created clones, change their name and delete them, as well as using them with tts & vc endpoints.

The only notable difference is, that `ClonedVoice` option needs to be set to `true` when doing the convert request:

```go
&charactr.TTSRequestOptions{ClonedVoice: true}
```

For more, see the relevant code example: [Full Voice Cloning Example](https://github.com/charactr-platform/charactr-api-sdk-go/blob/main/example/clones/main.go)

## Examples

Our SDK comes with a bunch of examples to help you start working with it. Please, refer to the SDK's [README page](https://github.com/charactr-platform/charactr-api-sdk-go) if you wish to run them.
