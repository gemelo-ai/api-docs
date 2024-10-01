[← Back to main page](../README.md#7-examples)

# Dialogue System

## How to create a dialogue system using OpenAI and Charactr API?

The main idea is to send three requests:

1. Sending an audio sample to Whisper (ASR) to convert speech to text.
2. Sending this text to ChatGPT to get a text response.
3. Sending this response to gemelo.ai TTS to convert text to speech and get a voice response.

This is one iteration of the dialogue. To have a longer conversation, we can repeat these three steps many times and collect text responses to keep context. We can implement such a dialogue system in a few small steps:

### Step 1: Install required libraries

To use the gemelo.ai API for TTS and OpenAI API for Whisper and ChatGPT, install the following:

```bash
pip install charactr-api-sdk openai
pip install requests ipywebrtc ipython
```

### Step 2: Load required libraries

```python
import json
from typing import Dict, List

import IPython.display
import openai
import requests
from charactr_api import CharactrAPISDK, Credentials
from ipywebrtc import AudioRecorder, CameraStream
```

### Step 3: Set up your API keys

```python
openai_api_key = 'xxxx'
charactr_client_key = 'yyyy'
charactr_api_key = 'zzzz'

openai.api_key = openai_api_key
```

### Step 4: Create `CharactrAPISDK` instance

```python
credentials = Credentials(client_key=charactr_client_key, api_key=charactr_api_key)
charactr_api = CharactrAPISDK(credentials)
```

### Step 5: Get list of available voices

```python
charactr_api.tts.get_voices()
```

You get a list of voices. Choose one and set up the `voice_id`:

```python
voice_id = 136
```

### Step 6: Set up ChatGPT model and parameters

```python
model = 'gpt-3.5-turbo'
parameters = {
    'temperature': 0.8,
    'max_tokens': 150,
    'top_p': 1,
    'presence_penalty': 0,
    'frequency_penalty': 0,
    'stop': None
}
```

### Step 7: Define a function that generates a text response based on conversation history

```python
def update_conversation(request: str, conversation: List[Dict]) -> None:
    """Run a request to ChatGPT to get a response and update the conversation."""
    try:
        user_request = {'role': 'user', 'content': request}
        conversation.append(user_request)
        result = openai.ChatCompletion.create(model=model,
                                              messages=conversation,
                                              **parameters)
        response = result['choices'][0]['message']['content'].strip()
        bot_response = {'role': 'assistant', 'content': response}
        conversation.append(bot_response)
    except Exception as e:
        raise Exception(e)
```

### Step 8: Set up the conversation

```python
conversation = [{'role': 'system', 'content': 'You are AI Assistant.'}]
```

### Step 9: Define a function to convert speech to text using Whisper

```python
def speech2text(audio_path: str) -> str:
    """Run a request to Whisper to convert speech to text."""
    try:
        with open(audio_path, 'rb') as audio_f:
            result = openai.Audio.transcribe('whisper-1', audio_f)
        text = result['text']
    except Exception as e:
        raise Exception(e)
    return text
```

### Step 10: Create an audio recorder in a notebook

```python
camera = CameraStream(constraints={'audio': True, 'video': False})
recorder = AudioRecorder(stream=camera)
recorder
```

### Step 11: Record and save your request

```python
temp_path = 'recording.webm'
with open(temp_path, 'wb') as f:
    f.write(recorder.audio.value)
```

### Step 12: Generate a voice response using Whisper, ChatGPT, and gemelo.ai TTS

```python
# convert the recorded audio to text
input_text = speech2text(temp_path)
# generate a text response and update conversation
update_conversation(input_text, conversation)
# convert a text response to speech
tts_result = charactr_api.tts.convert(voice_id, conversation[-1]['content'])
```

### Step 13: Listen to the output voice response

To listen to the voice response in a notebook:

```python
IPython.display.Audio(tts_result['data'])
```

### Step 14: Save the output voice response as a file

```python
with open('output.wav', 'wb') as f:
    f.write(tts_result['data'])
```

To continue the conversation, repeat steps 8 and 9.
