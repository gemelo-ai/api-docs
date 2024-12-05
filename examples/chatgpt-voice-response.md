[← Back to main page](../README.md#7-examples)

# Voice Response

## How to combine ChatGPT and Charactr API to get a voice response?

The main idea is to send a request to ChatGPT to get a text response and then use this output as input to Charactr TTS. Follow these small steps:

### Step 1: Install required libraries

To use the gemelo.ai API for TTS and OpenAI API for Whisper and ChatGPT, install the following:

```bash
pip install charactr-api-sdk openai
pip install requests ipython
```

### Step 2: Load required libraries

```python
import json
from typing import Dict, List

import IPython.display
import openai
import requests
from charactr_api import CharactrAPISDK, Credentials
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

### Step 7: Define a function to generate a text response

```python
def generate(request: str) -> str:
    """Generate a text response with ChatGPT."""
    messages = [{'role': 'user', 'content': request}]
    result = openai.ChatCompletion.create(model=model,
                                          messages=messages,
                                          **parameters)
    try:
        response = result['choices'][0]['message']['content'].strip()
    except Exception as e:
        raise Exception(e)
    return response
```

### Step 8: Generate a text response

```python
text = 'Tell me a joke'
response = generate(text)
```

### Step 9: Generate a voice response

```python
tts_result = charactr_api.tts.convert(voice_id, response)
```

### Step 10: Listen to the output voice response

To listen to the voice response in a notebook:

```python
IPython.display.Audio(tts_result['data'])
```

### Step 11: Save the output voice response as a file

```python
with open('output.wav', 'wb') as f:
    f.write(tts_result['data'])
```
