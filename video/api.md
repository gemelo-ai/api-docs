[← Back to main page](../README.md#3-api-reference)

# Gemelo Video API Docs

## Quick API Flow Overview

1.  **Create a Video**:
    - Use the `/v2/videos` endpoint to create a video.
      - You can provide audio in three ways:
      - **Text-to-Speech**: Submit text to be converted into speech.
      - **Audio URL**: Provide a link to an audio file.
      - **Recording ID**: Use an existing recording.
2.  **Check Video Status**:

    - Use `/v2/videos/{video_id}/status` to track the status of a video.

3.  **Retrieve a Video**:

    - Use `/v2/videos/{video_id}` to get the video details once it’s processed.

4.  **Manage Videos**:

    - Use `/v2/videos` to retrieve all videos.
    - Use `/v2/videos/{video_id}` to delete a specific video.

5.  **Additional Resources**:

    - Use `/v2/twins` to retrieve available twin avatars.
    - Use `/v2/voices` to retrieve available voices.

6.  **Download Video**:
    - Use `/v3/files/video_{video_id}` to download a video file.

## Video API Documentation

This API allows users to create, retrieve, and manage video resources, including generating videos with specific voices and audio sources.

#### Authentication

All API requests require the following headers:

- `X-Client-Key`: The client secret key.
- `X-API-Key`: The API key for authentication.

#### 1. Create a Video

You can create a video in three different ways depending on the audio source:

**Option 1: Using Text-to-Speech** Submit text that will be converted into speech.

```bash
curl -X POST 'https://api.gemelo.ai/v2/videos' \
  -H 'X-Client-Key: SECRET' \
  -H 'X-API-Key: APIK' \
  -H 'Content-type: application/json' \
  --data '{"twin_id": 20, "voice": {"voice_id": 181}, "audio": {"text": "Hello, welcome to our service!"}, "consent": true}'
```

**Option 2: Using an Audio URL** Provide a URL pointing to an audio file that will be used for the video.

```bash
curl -X POST 'https://api.gemelo.ai/v2/videos' \
  -H 'X-Client-Key: SECRET' \
  -H 'X-API-Key: APIK' \
  -H 'Content-type: application/json' \
  --data '{"twin_id": 20, "voice": {"voice_id": 181}, "audio": {"audio_url": "ADD_URL"}, "consent": true}'
```

**Option 3: Using a Pre-recorded Audio (Recording ID)** Specify a pre-existing recording by its `recording_id`.

```bash
curl -X POST 'https://api.gemelo.ai/v2/videos' \
  -H 'X-Client-Key: SECRET' \
  -H 'X-API-Key: APIK' \
  -H 'Content-type: application/json' \
  --data '{"twin_id": 20, "voice": {"voice_id": 181}, "audio": {"recording_id": 12}, "consent": true}'
```

**Common parameters** for all three options:

- `twin_id`: ID of the twin avatar.
- `voice_id`: ID of the voice to use.
- `consent`: Boolean indicating consent for video generation.

---

#### 2. Check Video Status

Check the current status of a specific video using its `video_id`. Video generation takes a while, check the ready field to know, when it’s ready.

**Request**:

```bash
curl -X GET 'https://api.gemelo.ai/v2/videos/123/status' \
  -H 'X-Client-Key: SECRET' \
  -H 'X-API-Key: APIK'
```

**Parameters**:

- `123`: Replace with the actual video ID.

---

#### 3. Retrieve All Videos

Fetch a list of all generated videos.

**Request**:

```bash
curl -X GET 'https://api.gemelo.ai/v2/videos' \
  -H 'X-Client-Key: SECRET' \
  -H 'X-API-Key: APIK'
```

---

#### 4. Retrieve a Video

Retrieve the details of a specific video using its `video_id`.

**Request**:

```bash
curl -X GET 'https://api.gemelo.ai/v2/videos/123' \
  -H 'X-Client-Key: SECRET' \
  -H 'X-API-Key: APIK'
```

---

#### 5. Delete a Video

Delete a specific video using its `video_id`.

**Request**:

```bash
curl -X DELETE 'https://api.gemelo.ai/v2/videos/123' \
  -H 'X-Client-Key: SECRET' \
  -H 'X-API-Key: APIK'
```

---

#### 6. Retrieve All Twins

Fetch a list of all available twin avatars.

**Request**:

```bash
curl -X GET 'https://api.gemelo.ai/v2/twins' \
  -H 'X-Client-Key: SECRET' \
  -H 'X-API-Key: APIK'
```

---

#### 7. Retrieve All Voices

Fetch a list of all available voices.

**Request**:

```bash
curl -X GET 'https://api.gemelo.ai/v2/voices' \
  -H 'X-Client-Key: SECRET' \
  -H 'X-API-Key: APIK'
```

---

#### 8. Download a Video File

Download a video file using its `video_id`.

**Request**:

```bash
curl -X GET 'https://api.gemelo.ai/v3/files/video_123' \
  -H 'X-Client-Key: SECRET' \
  -H 'X-API-Key: APIK' \
  --output video123.mp4
```

**Parameters**:

- `123`: Replace with the actual video ID.

---

### Notes

- Always ensure that you have the correct API keys and permissions to access these endpoints.

For further inquiries, feel free to contact us.

## curl reference

```bash
curl -X POST   'https://api.gemelo.ai/v2/videos'            -H 'X-Client-Key: SECRET' -H 'X-API-Key: APIK' -H 'Content-type: application/json' --data '{"twin_id": 20, "voice": {"voice_id": 181}, "audio": {"text": "Hello, welcome to our service!"}, "consent": true}'
curl -X POST   'https://api.gemelo.ai/v2/videos'            -H 'X-Client-Key: SECRET' -H 'X-API-Key: APIK' -H 'Content-type: application/json' --data '{"twin_id": 20, "voice": {"voice_id": 181}, "audio": {"audio_url": "ADD_URL"}, "consent": true}'
curl -X POST   'https://api.gemelo.ai/v2/videos'            -H 'X-Client-Key: SECRET' -H 'X-API-Key: APIK' -H 'Content-type: application/json' --data '{"twin_id": 20, "voice": {"voice_id": 181}, "audio": {"recording_id": 12}, "consent": true}'
curl -X GET    'https://api.gemelo.ai/v2/videos'            -H 'X-Client-Key: SECRET' -H 'X-API-Key: APIK'
curl -X GET    'https://api.gemelo.ai/v2/videos/123/status' -H 'X-Client-Key: SECRET' -H 'X-API-Key: APIK'
curl -X GET    'https://api.gemelo.ai/v2/videos/123'        -H 'X-Client-Key: SECRET' -H 'X-API-Key: APIK'
curl -X DELETE 'https://api.gemelo.ai/v2/videos/123'        -H 'X-Client-Key: SECRET' -H 'X-API-Key: APIK'
curl -X GET    'https://api.gemelo.ai/v2/twins'             -H 'X-Client-Key: SECRET' -H 'X-API-Key: APIK'
curl -X GET    'https://api.gemelo.ai/v2/voices'            -H 'X-Client-Key: SECRET' -H 'X-API-Key: APIK'
curl -X GET    'https://api.gemelo.ai/v3/files/video_123'   -H 'X-Client-Key: SECRET' -H 'X-API-Key: APIK' --output video123.mp4
```
