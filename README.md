# live-captions
script that captures live audio and provides real-time speech-to-text transcription.

![](https://miro.medium.com/v2/resize:fit:1400/0*yJ9CwrhodY5Asjn4)

---

**Building a Real-Time Speech-to-Text Application with Deepgram and Python**

Real-time speech transcription has become a valuable tool in industries like customer service, content creation, and accessibility. Using Deepgram’s API and Python, you can build a real-time transcription tool that processes audio from your microphone and outputs the text dynamically. In this blog post, we'll walk you through a Python script that achieves this with features like speaker diarization and punctuation.

---

### Prerequisites
Before diving in, ensure you have the following:

1. **Python 3.7 or higher**: Install Python from [python.org](https://www.python.org/).
2. **Deepgram API Key**: Sign up at [Deepgram](https://deepgram.com/) and obtain your API key.
3. **Dependencies**: Install required Python libraries using the command below:

   ```bash
   pip install sounddevice websockets
   ```

4. **Audio Input Device**: A functional microphone for capturing audio.

---

### Key Features of the Script

The script is designed to:
- **Select Audio Input Devices**: Allows you to choose from available microphone devices.
- **Stream Audio**: Captures real-time audio and sends it to Deepgram’s WebSocket API.
- **Receive Transcriptions**: Processes responses from Deepgram, including speaker identification and punctuation.
- **Save the Transcript**: Outputs the transcription to a text file for later use.

---

### Step-by-Step Explanation

#### 1. **Set Up the Environment**
Start by importing the necessary libraries and setting up the Deepgram API key.

```python
import asyncio
import sounddevice as sd
import websockets
import json
import os
import sys

DEEPGRAM_API_KEY = os.getenv('DEEPGRAM_API_KEY') or "your_api_key_here"
if not DEEPGRAM_API_KEY:
    print("Deepgram API Key not found. Please set the DEEPGRAM_API_KEY environment variable.")
    sys.exit(1)
```

Here, the `DEEPGRAM_API_KEY` is fetched from the environment or manually defined.

#### 2. **Select Audio Input Device**
The script lists available audio input devices and prompts the user to select one:

```python
devices = sd.query_devices()
input_devices = [idx for idx, dev in enumerate(devices) if dev['max_input_channels'] > 0]
print("Available audio input devices:")
for idx in input_devices:
    print(f"{idx}: {devices[idx]['name']}")

while True:
    try:
        device_id = int(input("Select the input device ID: "))
        if device_id in input_devices:
            break
        else:
            print("Invalid device ID. Please try again.")
    except ValueError:
        print("Please enter a valid integer.")
```

#### 3. **Configure Audio Streaming**
Set up parameters like sample rate and buffer size. The WebSocket URL includes query parameters for Deepgram’s features like speaker diarization and punctuation.

```python
SAMPLE_RATE = 16000
BLOCK_SIZE = 1024
CHANNELS = 1

url = f"wss://api.deepgram.com/v1/listen?diarize=true&punctuate=true&encoding=linear16&sample_rate={SAMPLE_RATE}&channels={CHANNELS}"
```

#### 4. **Send and Receive Audio Data**
Two coroutines handle the core tasks:
- **Send Audio**: Captures microphone input and streams it to Deepgram.
- **Receive Transcriptions**: Processes responses and displays them in real time.

```python
async def send_audio():
    def callback(indata, frames, time, status):
        if status:
            print(status, file=sys.stderr)
        asyncio.run_coroutine_threadsafe(ws.send(indata.tobytes()), loop)

    with sd.InputStream(samplerate=SAMPLE_RATE, blocksize=BLOCK_SIZE, device=device_id,
                        channels=CHANNELS, dtype='int16', callback=callback):
        print("Recording... Press Ctrl+C to stop.")
        await stop_signal.wait()

async def receive_transcript():
    async for message in ws:
        response = json.loads(message)
        if 'channel' in response:
            words = response['channel'].get('alternatives', [])[0].get('words', [])
            for word_info in words:
                print(word_info.get('word', ''), end=' ', flush=True)
```

#### 5. **Handle Transcription Output**
The transcription is continuously displayed in real-time and saved to a text file when the program exits.

```python
if transcript:
    with open("real_time_transcript.txt", "w", encoding="utf-8") as f:
        f.write(transcript)
    print("\nTranscript saved to real_time_transcript.txt")
```

---

### Running the Script
To run the script, execute:

```bash
python your_script_name.py
```

Select your audio input device when prompted, and start speaking. The transcription appears in real-time and is saved to a text file upon completion.

---

### Customization and Extensions

- **Language Support**: Modify the WebSocket URL to specify a language.
- **Advanced Features**: Enable additional Deepgram options like language models or custom vocabularies.
- **Error Handling**: Improve error handling for better robustness.
- **Integration**: Combine this script with applications like Zoom or a chatbot for live captioning.

---

### Conclusion
This tutorial demonstrates how to build a real-time speech-to-text application using Deepgram and Python. With just a few lines of code, you can unlock the potential of real-time transcription, opening doors to various applications in accessibility, automation, and more.

Feel free to experiment and adapt the script to your needs. Happy coding!

