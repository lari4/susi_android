# SUSI.AI Android Application - Agent Pipelines Documentation

## Overview

This document describes all the agent workflows and data pipelines in the SUSI.AI Android application. It shows how data flows from user input through various processing stages to final output, what prompts are used at each stage, and how different components interact.

**Architecture:** The app follows MVP (Model-View-Presenter) pattern where:
- **Model** handles data and business logic
- **View** handles UI display
- **Presenter** mediates between Model and View

**Key Components:**
- `ChatPresenter.kt` - Main presenter for chat functionality
- `ChatModel.kt` - Handles API calls and data processing
- `ParseSusiResponseHelper.kt` - Parses server responses into actionable data
- `SusiService.kt` - Retrofit API interface to SUSI backend

---

## Pipeline 1: Text Query Pipeline

This is the core pipeline for handling text-based user queries to SUSI.

### Flow Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          TEXT QUERY PIPELINE                             │
└─────────────────────────────────────────────────────────────────────────┘

User Action                Presenter                 Model                Server
    │                          │                       │                     │
    │  1. Types message        │                       │                     │
    │     "Ask SUSI something…"│                       │                     │
    ├─────────────────────────>│                       │                     │
    │  (send_msg_hint prompt)  │                       │                     │
    │                          │                       │                     │
    │                          │ 2. Validate input     │                     │
    │                          │    Check network      │                     │
    │                          │                       │                     │
    │                          │ 3. Build query params │                     │
    │                          │    - Get location     │                     │
    │                          │    - Get timezone     │                     │
    │                          │    - Get language     │                     │
    │                          │    - Add user query   │                     │
    │                          │                       │                     │
    │                          │ 4. Save to DB         │                     │
    │                          │    (user message)     │                     │
    │                          │                       │                     │
    │  5. Show waiting dots    │                       │                     │
    │<─────────────────────────┤                       │                     │
    │                          │                       │                     │
    │                          │ 6. Request API call   │                     │
    │                          ├──────────────────────>│                     │
    │                          │                       │                     │
    │                          │                       │ 7. Send HTTP GET    │
    │                          │                       ├────────────────────>│
    │                          │                       │  /susi/chat.json    │
    │                          │                       │  ?q=<query>         │
    │                          │                       │  &latitude=...      │
    │                          │                       │  &longitude=...     │
    │                          │                       │  &timezoneOffset=...│
    │                          │                       │  &language=...      │
    │                          │                       │  &device_type=...   │
    │                          │                       │                     │
    │                          │                       │ 8. AI processes     │
    │                          │                       │    (server-side)    │
    │                          │                       │                     │
    │                          │                       │ 9. Returns response │
    │                          │                       │<────────────────────┤
    │                          │                       │  SusiResponse JSON  │
    │                          │                       │                     │
    │                          │ 10. Callback success  │                     │
    │                          │<──────────────────────┤                     │
    │                          │    onSusiMessageReceivedSuccess()           │
    │                          │                       │                     │
    │                          │ 11. Parse response    │                     │
    │                          │     ParseSusiResponseHelper.parseSusiResponse()
    │                          │                       │                     │
    │                          │ 12. Save to DB        │                     │
    │                          │     (SUSI response)   │                     │
    │                          │                       │                     │
    │  13. Update UI           │                       │                     │
    │      Display message     │                       │                     │
    │<─────────────────────────┤                       │                     │
    │  14. Hide waiting dots   │                       │                     │
    │<─────────────────────────┤                       │                     │
    │                          │                       │                     │
    │  15. Execute action      │                       │                     │
    │      (based on type)     │                       │                     │
    │<─────────────────────────┤                       │                     │
    │                          │                       │                     │
```

### Data Flow Details

#### Step 1: User Input
- **Location:** `ChatActivity.kt`
- **Prompt Used:** `send_msg_hint` - "Ask SUSI something…"
- **User Action:** Types message in EditText field

#### Step 2-3: Prepare Query
- **Location:** `ChatPresenter.kt:330` - `sendMessage()`
- **Data Collected:**
  - `q` - User's query text
  - `timezoneOffset` - Calculated from device timezone
  - `latitude` - From LocationHelper (GPS or IP-based)
  - `longitude` - From LocationHelper (GPS or IP-based)
  - `geosource` - "GPS" or "IP"
  - `language` - User's preferred language or device default
  - `country_code` - User's country code
  - `country_name` - User's country name
  - `device_type` - "Android"

#### Step 4: Save User Message
- **Location:** `DatabaseRepository.kt`
- **Action:** Message saved to Realm database with timestamp
- **Purpose:** Persist chat history locally

#### Step 5-7: Send Request
- **Location:** `ChatPresenter.kt:393-402` - `computeOtherMessage()`
- **API Endpoint:** `GET /susi/chat.json`
- **Network Layer:** Retrofit + OkHttp
- **Service Interface:** `SusiService.kt:61-62`

#### Step 8: Server Processing
- **Server Side:** SUSI.AI backend processes query
- **AI Processing:** Natural language understanding, intent recognition, action generation
- **Not visible to client:** All AI intelligence is server-side

#### Step 9-10: Receive Response
- **Response Format:** JSON - `SusiResponse` object
```json
{
  "answers": [{
    "data": [...],
    "actions": [{
      "type": "answer|table|map|videoplay|audioplay|...",
      "expression": "...",
      ...
    }]
  }],
  "answerDate": "..."
}
```

#### Step 11: Parse Response
- **Location:** `ParseSusiResponseHelper.kt:30` - `parseSusiResponse()`
- **Action Types Handled:**
  - `ANSWER` - Text response
  - `TABLE` - Tabular data
  - `MAP` - Location/map display
  - `VIDEOPLAY` - Video playback
  - `AUDIOPLAY` - Audio playback
  - `ANCHOR` - Web links
  - `STOP` - Stop action

#### Step 12-15: Display & Execute
- **Location:** `ChatActivity.kt`
- **Actions:**
  - Add message to chat RecyclerView
  - Execute action based on type
  - Update UI accordingly

### Error Handling

**Network Error:**
```
User → Presenter (no network) → Display error
Prompt: "Internet Connection Not Available."
```

**Server Error:**
```
User → Presenter → Model → Server (error) → Callback failure
→ Display error: "Internet Connectivity Problem."
```

**Unknown Response:**
```
Server returns empty/invalid → Display fallback
Prompt: "I don't know."
```

### Key Files
- `/app/src/main/java/org/fossasia/susi/ai/chat/ChatPresenter.kt`
- `/app/src/main/java/org/fossasia/susi/ai/chat/ChatModel.kt`
- `/app/src/main/java/org/fossasia/susi/ai/chat/ParseSusiResponseHelper.kt`
- `/app/src/main/java/org/fossasia/susi/ai/rest/services/SusiService.kt`

---

## Pipeline 2: Voice Query Pipeline

This pipeline handles voice-based interaction with SUSI, including speech recognition and text-to-speech output.

### Flow Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         VOICE QUERY PIPELINE                             │
└─────────────────────────────────────────────────────────────────────────┘

User Action              STTFragment           SpeechRecognizer      TextToSpeech
    │                         │                        │                   │
    │  1. Tap mic button      │                        │                   │
    ├────────────────────────>│                        │                   │
    │                         │                        │                   │
    │                         │ 2. First time?         │                   │
    │                         │    Check preference    │                   │
    │                         │    (used_voice)        │                   │
    │                         │                        │                   │
    │  [First Time Only]      │                        │                   │
    │  3. Play welcome        │                        │                   │
    │     "Hi! How can I      │──────────────────────────────────────────>│
    │      help you?"         │  speak(voice_welcome)  │                   │
    │<────────────────────────────────────────────────────────────────────┤
    │  (voice_welcome prompt) │                        │                   │
    │                         │                        │                   │
    │  4. Show voice commands │                        │                   │
    │     "What is the time   │                        │                   │
    │      now?"              │                        │                   │
    │     "Where is Singapore"│                        │                   │
    │     etc.                │                        │                   │
    │<─────────────────────────                        │                   │
    │  (voiceCommands array)  │                        │                   │
    │                         │                        │                   │
    │  5. Show "Say Something"│                        │                   │
    │<─────────────────────────                        │                   │
    │  (speech_prompt)        │                        │                   │
    │                         │                        │                   │
    │                         │ 6. Start listening     │                   │
    │                         ├───────────────────────>│                   │
    │                         │  ACTION_RECOGNIZE_SPEECH                   │
    │                         │  LANGUAGE_MODEL_FREE_FORM                  │
    │                         │  EXTRA_PARTIAL_RESULTS│                   │
    │                         │                        │                   │
    │  7. User speaks         │                        │                   │
    ├─────────────────────────────────────────────────>│                   │
    │  "What is the weather?" │                        │                   │
    │                         │                        │                   │
    │                         │                        │ 8. Process audio  │
    │                         │                        │    (Google STT)   │
    │                         │                        │                   │
    │  9. Show partial results│ 10. Partial callback   │                   │
    │     "What is the..."    │<───────────────────────┤                   │
    │<─────────────────────────  onPartialResults()    │                   │
    │                         │                        │                   │
    │  11. Show final text    │ 12. Final results      │                   │
    │      "What is the       │<───────────────────────┤                   │
    │       weather?"         │  onResults()           │                   │
    │<─────────────────────────                        │                   │
    │                         │                        │                   │
    │                         │ 13. Send to chat       │                   │
    │                         │     setText(text)      │                   │
    │                         │                        │                   │
    │                         └──────────────────┐     │                   │
    │                                            │     │                   │
    │                     [CONTINUES TO TEXT QUERY PIPELINE]              │
    │                                            │     │                   │
    │                                            ▼     │                   │
    │                         Process as text query (see Pipeline 1)      │
    │                                                  │                   │
    │                         ┌────────────────────────┘                   │
    │                         │                        │                   │
    │                         │ 14. Get SUSI response  │                   │
    │                         │     (from server)      │                   │
    │                         │                        │                   │
    │  [If Speech Output ON]  │                        │                   │
    │                         │ 15. Check TTS setting  │                   │
    │                         │     speech_output or   │                   │
    │                         │     speech_always      │                   │
    │                         │                        │                   │
    │  16. Speak response     │                        │                   │
    │<────────────────────────────────────────────────────────────────────┤
    │  "The weather is..."    │  speak(response)       │                   │
    │                         │                        │                   │
```

### Data Flow Details

#### Step 1: Voice Activation
- **Location:** `ChatActivity.kt` - mic button click
- **Trigger:** User taps microphone icon or uses hotword detection
- **Fragment:** Opens `STTFragment`

#### Step 2-3: First-Time Welcome
- **Location:** `STTfragment.kt:67-69`
- **Check:** `PrefManager.getBoolean(R.string.used_voice, false)`
- **Prompt:** `voice_welcome` - "Hi! How can I help you?"
- **Action:** Play welcome message via TextToSpeech
- **Persistence:** Set preference to prevent repeated welcomes

#### Step 4: Display Example Commands
- **Location:** `STTfragment.kt:82-85`
- **Prompts:** `voiceCommands` array
  - "What is the time now?"
  - "Which is the largest country in the world?"
  - "Open WhatsApp"
  - "Where is Singapore?"
  - "Show me news headlines"
  - "Who is the president of India?"
- **Display:** Horizontal scrolling chips/buttons

#### Step 5-6: Start Speech Recognition
- **Location:** `STTfragment.kt:88-170`
- **Service:** Android `SpeechRecognizer`
- **Intent:** `RecognizerIntent.ACTION_RECOGNIZE_SPEECH`
- **Parameters:**
  - `LANGUAGE_MODEL_FREE_FORM` - Open-ended speech
  - `EXTRA_LANGUAGE` - Device locale
  - `EXTRA_PARTIAL_RESULTS` - Show real-time transcription
  - `EXTRA_SPEECH_INPUT_COMPLETE_SILENCE_LENGTH_MILLIS` - 5000ms timeout

#### Step 7-8: Audio Capture & Processing
- **Service Provider:** Google Speech Recognition (device-dependent)
- **Processing:** Audio converted to text using cloud-based STT
- **Real-time:** Partial results shown as user speaks

#### Step 9-10: Partial Results Display
- **Callback:** `onPartialResults()` in `RecognitionListener`
- **Location:** `STTfragment.kt:159-163`
- **Action:** Update UI with partial transcription
- **Purpose:** Visual feedback during speech

#### Step 11-12: Final Text Recognition
- **Callback:** `onResults()` in `RecognitionListener`
- **Location:** `STTfragment.kt:100-124`
- **Action:** Extract best match from results array
- **Data:** `RESULTS_RECOGNITION` string array

#### Step 13: Hand Off to Text Pipeline
- **Location:** `STTfragment.kt:114`
- **Action:** `thisActivity.setText(voiceResults[0])`
- **Next:** Continues to **Pipeline 1: Text Query Pipeline**

#### Step 14-16: Text-to-Speech Response
- **Settings Check:**
  - `speech_output` - TTS only for voice input
  - `speech_always` - TTS for all input types
- **Engine:** Android TextToSpeech
- **Action:** Speak SUSI's text response
- **Language:** Matches user's TTS preference

### Hotword Detection (Optional)

```
User Environment        Hotword Detector        App
       │                      │                  │
       │  Background audio    │                  │
       ├─────────────────────>│                  │
       │  "SUSI..."           │                  │
       │                      │ Pattern match    │
       │                      │ (Snowboy lib)    │
       │                      │                  │
       │                      │ Detected!        │
       │                      ├─────────────────>│
       │                      │                  │
       │                      │                  │ Trigger STT
       │                      │                  │ Fragment
       │                      │                  │
       │  Auto-start voice    │                  │
       │<──────────────────────────────────────────
       │  (hotword_success)   │                  │
```

- **Library:** Snowboy hotword detection
- **Model Files:** `/app/src/main/assets/snowboy/`
- **Trigger Word:** "SUSI"
- **Preference:** `hotword_detection` setting
- **Success Prompt:** "Hotword detected"

### Error Handling

**Speech Recognition Not Supported:**
```
User → Check capability → Device lacks STT
→ Display: "Sorry, your device doesn't support speech input"
```

**Speech Recognition Error:**
```
User → Speak → STT Error → onError() callback
→ Toast: "Could not recognize speech, try again."
```

**Hotword Not Supported:**
```
User → Enable hotword → Device incompatible
→ Display: "Sorry, hotword detection is not supported on your device."
```

### Settings Impact

**Mic Input Setting:** (`setting_mic_key`)
- Controls whether microphone icon is visible
- Preference: `mic_input`

**Speech Output Setting:** (`settings_speechPreference_key`)
- TTS only for voice input
- Preference: `speech_output`

**Speech Always Setting:** (`settings_speechAlways_key`)
- TTS for all input types
- Preference: `speech_always`

**Hotword Detection:** (`setting_hotword_key`)
- Background listening for wake word
- Preference: `hotword_detection`

### Key Files
- `/app/src/main/java/org/fossasia/susi/ai/chat/STTfragment.kt`
- `/app/src/main/java/org/fossasia/susi/ai/chat/ChatActivity.kt`
- `/app/src/main/assets/snowboy/` - Hotword detection models

---
