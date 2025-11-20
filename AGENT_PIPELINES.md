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

## Pipeline 3: Response Processing Pipeline

This pipeline shows how SUSI server responses are parsed and rendered based on their action types.

### Flow Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    RESPONSE PROCESSING PIPELINE                          │
└─────────────────────────────────────────────────────────────────────────┘

Server Response         ParseSusiResponseHelper      ChatActivity (UI)
       │                          │                           │
       │  SusiResponse JSON       │                           │
       ├─────────────────────────>│                           │
       │  {                       │                           │
       │    "answers": [{         │                           │
       │      "actions": [        │ 1. Extract action type    │
       │        {"type": "..."}   │    from response          │
       │      ]                   │                           │
       │    }]                    │                           │
       │  }                       │                           │
       │                          │                           │
       │                          │ 2. Switch on actionType   │
       │                          │                           │
       ├──────────────────────────┴───────────────────────────┤
       │                                                       │
       │  ACTION TYPE ROUTING:                                │
       │  ┌─────────────────────────────────────────────────┐ │
       │  │                                                 │ │
       │  │  type: "answer"                                 │ │
       │  │  ├──> Extract expression                        │ │
       │  │  ├──> Extract URLs from data                    │ │
       │  │  └──> Display text + links                      │ │
       │  │       → ChatMessageViewHolder (text bubble)     │ │
       │  │                                                 │ │
       │  │  type: "table"                                  │ │
       │  │  ├──> Extract columns map                       │ │
       │  │  ├──> Extract data rows                         │ │
       │  │  ├──> Build TableItem                           │ │
       │  │  └──> Display table view                        │ │
       │  │       → ChatMessageViewHolder (table layout)    │ │
       │  │                                                 │ │
       │  │  type: "map"                                    │ │
       │  │  ├──> Extract latitude                          │ │
       │  │  ├──> Extract longitude                         │ │
       │  │  ├──> Extract zoom level                        │ │
       │  │  └──> Display map                               │ │
       │  │       → MapData → MapView                       │ │
       │  │                                                 │ │
       │  │  type: "videoplay"                              │ │
       │  │  ├──> Extract video identifier (YouTube ID)     │ │
       │  │  └──> Display video player                      │ │
       │  │       → YouTubePlayerView                       │ │
       │  │                                                 │ │
       │  │  type: "audioplay"                              │ │
       │  │  ├──> Extract audio identifier (URL)            │ │
       │  │  └──> Display audio player                      │ │
       │  │       → MediaPlayer controls                    │ │
       │  │                                                 │ │
       │  │  type: "anchor"                                 │ │
       │  │  ├──> Extract link URL                          │ │
       │  │  ├──> Extract link text                         │ │
       │  │  └──> Display clickable link                    │ │
       │  │       → HTML anchor <a href="...">              │ │
       │  │                                                 │ │
       │  │  type: "stop"                                   │ │
       │  │  └──> Stop ongoing playback                     │ │
       │  │       → Stop MediaPlayer / YouTube              │ │
       │  │                                                 │ │
       │  └─────────────────────────────────────────────────┘ │
       │                          │                           │
       │                          │ 3. Create ChatArgs        │
       │                          │    with parsed data       │
       │                          │                           │
       │                          │ 4. Save to database       │
       │                          │    DatabaseRepository     │
       │                          │                           │
       │                          │ 5. Notify UI to update    │
       │                          ├──────────────────────────>│
       │                          │                           │
       │                          │                           │ 6. Render in
       │                          │                           │    RecyclerView
       │                          │                           │
```

### Action Type Details

#### ANSWER - Text Response
**Purpose:** Display simple text answers from SUSI

**Parsing Logic:** (`ParseSusiResponseHelper.kt:42-60`)
```kotlin
answer = susiResponse.answers[0].actions[i].expression
// Extract embedded URLs
val text = susiResponse.answers[0].data[0].get("object")
val urlList = extractUrls(text)
if (urlList.isNotEmpty()) {
    answer += "\n" + urlList[0]
}
```

**Example Response:**
```json
{
  "expression": "The weather is sunny today.",
  "data": [{"object": "Check https://weather.com for details"}]
}
```

**Rendered As:** Text message with clickable URL

---

#### TABLE - Tabular Data
**Purpose:** Display structured data in table format

**Parsing Logic:** (`ParseSusiResponseHelper.kt:92-115`)
```kotlin
val listColumn = ArrayList<String>()      // Column names
val listColVal = ArrayList<String>()      // Column display values
val listTableData = ArrayList<String>()   // Row data

susiResponse.answers.forEach { answer ->
    answer.actions.forEach { action ->
        action.columns?.forEach { entry ->
            listColumn.add(entry.key)
            listColVal.add(entry.value.toString())
        }
    }
    answer.data.forEach {
        listColumn.forEach { i ->
            listTableData.add(it[i].toString())
        }
    }
}
tableData = TableItem(listColVal, listTableData)
```

**Example Use Case:**
- Query: "Show me top programming languages"
- Response: Table with columns [Language, Popularity, Year]

**Rendered As:** RecyclerView with table layout

---

#### MAP - Geographic Location
**Purpose:** Display location on a map

**Parsing Logic:** (`ParseSusiResponseHelper.kt:62-70`)
```kotlin
val latitude = susiResponse.answers[0].actions[i].latitude
val longitude = susiResponse.answers[0].actions[i].longitude
val zoom = susiResponse.answers[0].actions[i].zoom
MapData(latitude, longitude, zoom)
```

**Example Use Case:**
- Query: "Where is Singapore?"
- Response: Map centered on Singapore coordinates

**Rendered As:** Google Maps view with marker

---

#### VIDEOPLAY - Video Playback
**Purpose:** Play YouTube videos inline

**Parsing Logic:** (`ParseSusiResponseHelper.kt:78-83`)
```kotlin
identifier = susiResponse.answers[0].actions[i].identifier
// identifier is YouTube video ID (e.g., "dQw4w9WgXcQ")
```

**Example Use Case:**
- Query: "Show me cat videos"
- Response: YouTube player with video ID

**Rendered As:** Embedded YouTube player

**File Reference:** `YoutubeVid.kt` - YouTube player handler

---

#### AUDIOPLAY - Audio Playback
**Purpose:** Play audio files or streams

**Parsing Logic:** (`ParseSusiResponseHelper.kt:85-90`)
```kotlin
identifier = susiResponse.answers[0].actions[i].identifier
// identifier is audio URL or stream
```

**Example Use Case:**
- Query: "Play some music"
- Response: Audio stream URL

**Rendered As:** Audio player controls with play/pause/stop

---

#### ANCHOR - Web Links
**Purpose:** Display clickable hyperlinks

**Parsing Logic:** (`ParseSusiResponseHelper.kt:35-40`)
```kotlin
answer = "<a href=\"" +
         susiResponse.answers[0].actions[i].anchorLink +
         "\">" +
         susiResponse.answers[0].actions[i].anchorText +
         "</a>"
```

**Example:**
- Link: `https://example.com`
- Text: "Read more"
- Rendered: [Read more](https://example.com)

---

#### STOP - Playback Control
**Purpose:** Stop current media playback

**Parsing Logic:** (`ParseSusiResponseHelper.kt:72-76`)
```kotlin
stop = susiResponse.answers[0].actions[1].type
```

**Example Use Case:**
- Query: "Stop playing"
- Response: Stops active audio/video

---

### Multi-Action Responses

Server can return multiple actions in a single response:

```json
{
  "answers": [{
    "actions": [
      {"type": "answer", "expression": "Here's a video about that:"},
      {"type": "videoplay", "identifier": "dQw4w9WgXcQ"}
    ]
  }]
}
```

**Processing:**
```kotlin
val actionSize = susiResponse.answers[0].actions.size
for (i in 0 until actionSize) {
    parseSusiResponse(susiResponse, i, error)
    // Each action rendered sequentially
}
```

**Result:** Text message followed by video player

---

### Error Handling

**Empty Response:**
```kotlin
if (susiResponse.answers.isEmpty()) {
    // Display: "Internet Connectivity Problem."
}
```

**Parse Error:**
```kotlin
catch (e: Exception) {
    Timber.e(e)
    answer = error  // "An error occurred. please try again."
}
```

**Unknown Action Type:**
```kotlin
else -> answer = error
// Display error message for unrecognized types
```

### Database Persistence

All parsed responses are saved to local Realm database:

**ChatArgs Data Structure:**
- `prevId` - ID of previous message (for linking)
- `message` - Parsed text content
- `date` - Response date from server
- `timeStamp` - Local timestamp
- `actionType` - Type of action (ANSWER, TABLE, MAP, etc.)
- `mapData` - Map coordinates (if applicable)
- `identifier` - Video/audio ID (if applicable)
- `tableData` - Table structure (if applicable)

**Purpose:**
- Offline access to chat history
- Fast scroll through previous conversations
- Restore state after app restart

### Key Files
- `/app/src/main/java/org/fossasia/susi/ai/chat/ParseSusiResponseHelper.kt`
- `/app/src/main/java/org/fossasia/susi/ai/data/db/DatabaseRepository.kt`
- `/app/src/main/java/org/fossasia/susi/ai/chat/adapters/viewholders/ChatMessageViewHolder.kt`

---

## Pipeline 4: Skill Discovery & Listing Pipeline

This pipeline shows how users discover and browse SUSI skills through the skills marketplace.

### Flow Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                   SKILL DISCOVERY & LISTING PIPELINE                     │
└─────────────────────────────────────────────────────────────────────────┘

User Action              SkillListingActivity      Server
    │                            │                   │
    │  1. Open Skills page       │                   │
    ├───────────────────────────>│                   │
    │                            │                   │
    │  2. Show metric queries    │                   │
    │     (suggested searches)   │                   │
    │<───────────────────────────┤                   │
    │                            │                   │
    │  Prompts displayed:        │                   │
    │  • "SUSI, What are your    │                   │
    │     highest rated skills?" │                   │
    │  • "SUSI, what are your    │                   │
    │     most used skills?"     │                   │
    │  • "SUSI, what are the     │                   │
    │     recently updated       │                   │
    │     skills?"               │                   │
    │  • "SUSI, what are the     │                   │
    │     skills with most       │                   │
    │     feedback?"             │                   │
    │  • "SUSI, what are the     │                   │
    │     newest skills?"        │                   │
    │  • "SUSI, what are your    │                   │
    │     top games?"            │                   │
    │                            │                   │
    │  3. Select category/metric │                   │
    ├───────────────────────────>│                   │
    │     (e.g., "Top Rated")    │                   │
    │                            │                   │
    │                            │ 4. Fetch skills   │
    │                            ├──────────────────>│
    │                            │  GET /cms/        │
    │                            │  getSkillList.json│
    │                            │  ?model=general   │
    │                            │  &group=Knowledge │
    │                            │  &language=en     │
    │                            │                   │
    │                            │ 5. Return list    │
    │                            │<──────────────────┤
    │                            │  ListSkillsResponse
    │                            │                   │
    │  6. Display skills         │                   │
    │     • Skill name           │                   │
    │     • Description          │                   │
    │     • Author               │                   │
    │     • Rating ★★★★☆         │                   │
    │     • Examples             │                   │
    │<───────────────────────────┤                   │
    │                            │                   │
    │  7. Tap on skill           │                   │
    ├───────────────────────────>│                   │
    │                            │                   │
    │  8. Open skill details     │                   │
    │<───────────────────────────┤                   │
    │     • Full description     │                   │
    │     • Example queries      │                   │
    │     • Average rating       │                   │
    │     • Total ratings        │                   │
    │     • Feedback section     │                   │
    │     • "Try It" button      │                   │
    │                            │                   │
    │  9. Tap "Try It"           │                   │
    ├───────────────────────────>│                   │
    │                            │                   │
    │  10. Send example query    │                   │
    │      to chat               │                   │
    │<───────────────────────────┤                   │
    │                            │                   │
    │  [Continues to TEXT QUERY PIPELINE]           │
    │                            │                   │
```

### Skill Listing Endpoints

#### Get Skill Groups
**API:** `GET /cms/getGroups.json`
**Purpose:** Fetch available skill categories
**Response:** List of groups (Knowledge, Entertainment, Shopping, etc.)

#### Get Skills by Group
**API:** `GET /cms/getSkillList.json`
**Parameters:**
- `model` - Model type (e.g., "general")
- `group` - Skill group (e.g., "Knowledge")
- `language` - Language code (e.g., "en")

**Response:**
```json
{
  "skills": [{
    "skill_name": "Weather",
    "author": "John Doe",
    "description": "Get weather information",
    "examples": ["What's the weather?", "Weather in Paris"],
    "skill_rating": {
      "stars": { "avg_star": 4.5, "total_star": 100 }
    }
  }]
}
```

#### Get Skills by Metrics
**API:** `GET /cms/getSkillMetricsData.json`
**Parameters:**
- `model` - Model type
- `language` - Language code

**Metrics Available:**
- `rating` - Highest rated skills
- `usage` - Most used skills
- `latest` - Recently updated skills
- `feedback_count` - Most feedback received
- `creation_date` - Newest skills
- `top_games` - Top game skills

---

### Skill Rating System

```
User → Skill Details → Rate Skill (1-5 stars) → Submit
                              │
                              ▼
                    POST /cms/fiveStarRateSkill.json
                    Parameters:
                    • model, group, language, skill
                    • stars (1-5)
                    • accessToken (user auth)
                              │
                              ▼
                    Server updates rating
                              │
                              ▼
                    Display: "Thank you for rating this skill"
```

**Prompt:** `toast_thank_for_rating`
**Anonymous Users:** Cannot rate - "Skill not rated yet. Please login to rate the skill."

---

### Skill Feedback System

```
User → Skill Details → Write Feedback → Post
                              │
                              ▼
                    POST /cms/feedbackSkill.json
                    Parameters:
                    • model, group, language, skill
                    • feedback (text)
                    • accessToken
                              │
                              ▼
                    Server saves feedback
                              │
                              ▼
                    Display: "Skill feedback updated"
```

**Prompts:**
- Input hint: `hint_feedback` - "Skill feedback"
- Empty check: `toast_empty_feedback` - "Please enter a feedback to post"
- Success: `toast_feedback_updated` - "Skill feedback updated"
- Anonymous: `post_feedback_for_anonymous_user` - "Please login to post feedback"

---

### Skill Reporting

```
User → Skill Details → Flag → Report → Send
                              │
                              ▼
                    POST /cms/reportSkill.json
                    Parameters:
                    • model, group, language, skill
                    • feedback (report reason)
                    • accessToken
                              │
                              ▼
                    Server logs report
                              │
                              ▼
                    Display result
```

**Prompts:**
- Button: `report_skill` - "Flag as inappropriate"
- Send: `report_send` - "Send report"
- Success: `report_send_success` - "Skill reported successfully"
- Already reported: `reported_already` - "Skill already reported"
- Error: `report_error` - "Error reporting skill. Please try again later"

---

### Error States

**No Skills Found:**
```
Server returns empty → Display: "No skills found."
Prompt: message_no_skills_found
```

**Network Error:**
```
Request fails → Display: "Unable to fetch skills. Please try again later."
Prompt: error_skill_listing
```

**Missing Data:**
```
Skill has no name → Display: "Name not available"
Skill has no description → Display: "No description available"
Skill not rated → Display: "Skill not rated yet"
```

### Key Files
- `/app/src/main/java/org/fossasia/susi/ai/skills/skilllisting/SkillListingPresenter.kt`
- `/app/src/main/java/org/fossasia/susi/ai/skills/skilldetails/SkillDetailsFragment.kt`
- `/app/src/main/java/org/fossasia/susi/ai/rest/services/SusiService.kt`

---

## Pipeline 5: Device Setup Pipeline

This pipeline shows the complete flow for setting up a SUSI.AI smart speaker device.

### Flow Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                       DEVICE SETUP PIPELINE                              │
└─────────────────────────────────────────────────────────────────────────┘

User                    App                    Device              Server
 │                       │                       │                   │
 │ 1. Start setup        │                       │                   │
 ├──────────────────────>│                       │                   │
 │                       │                       │                   │
 │ 2. Show instructions  │                       │                   │
 │    "If you're setting │                       │                   │
 │     up a new device,  │                       │                   │
 │     make sure it's    │                       │                   │
 │     nearby and        │                       │                   │
 │     plugged in"       │                       │                   │
 │<──────────────────────┤                       │                   │
 │                       │                       │                   │
 │ 3. Plug in device     │                       │                   │
 ├───────────────────────────────────────────────>│                   │
 │                       │                       │ Play "Ping"       │
 │                       │                       │ sound             │
 │<───────────────────────────────────────────────┤                   │
 │                       │                       │                   │
 │                       │                       │ Open Wi-Fi        │
 │                       │                       │ hotspot           │
 │                       │                       │ "SUSI.AI"         │
 │                       │                       │                   │
 │ 4. Connect phone      │                       │                   │
 │    to "SUSI.AI" WiFi  │                       │                   │
 ├───────────────────────────────────────────────>│                   │
 │                       │                       │                   │
 │ 5. Connection check   │                       │                   │
 │    "Connected to      │                       │                   │
 │     SUSI.AI WiFi"     │                       │                   │
 │<──────────────────────┤                       │                   │
 │                       │                       │                   │
 │ 6. Scan WiFi networks │                       │                   │
 │    "Scanning for      │                       │                   │
 │     available WiFi's" │                       │                   │
 │<──────────────────────┤                       │                   │
 │                       │                       │                   │
 │                       │ 7. Request WiFi list  │                   │
 │                       ├──────────────────────>│                   │
 │                       │                       │ Scan networks     │
 │                       │                       │                   │
 │                       │ 8. Return WiFi list   │                   │
 │                       │<──────────────────────┤                   │
 │                       │                       │                   │
 │ 9. Select home WiFi   │                       │                   │
 │    Enter password     │                       │                   │
 ├──────────────────────>│                       │                   │
 │                       │                       │                   │
 │                       │ 10. Send credentials  │                   │
 │                       ├──────────────────────>│                   │
 │                       │   SSID + password     │                   │
 │                       │                       │                   │
 │ 11. "Wi-Fi credentials│                       │                   │
 │      sent             │                       │                   │
 │      successfully!"   │                       │                   │
 │<──────────────────────┤                       │                   │
 │                       │                       │                   │
 │ 12. Choose room       │                       │                   │
 │     • Kitchen         │                       │                   │
 │     • Bedroom         │                       │                   │
 │     • Create new      │                       │                   │
 ├──────────────────────>│                       │                   │
 │                       │                       │                   │
 │ 13. Link account      │                       │                   │
 │     Options:          │                       │                   │
 │     • Enter password  │                       │                   │
 │     • Anonymous mode  │                       │                   │
 ├──────────────────────>│                       │                   │
 │                       │                       │                   │
 │ [If Anonymous]        │                       │                   │
 │ 14. Warning shown:    │                       │                   │
 │     "If you do not    │                       │                   │
 │      link the device  │                       │                   │
 │      with an online   │                       │                   │
 │      account, you     │                       │                   │
 │      will not be able │                       │                   │
 │      to reconfigure   │                       │                   │
 │      your speaker..." │                       │                   │
 │<──────────────────────┤                       │                   │
 │                       │                       │                   │
 │ 15. Send account info │                       │                   │
 ├──────────────────────>│                       │                   │
 │                       │ 16. Configure device  │                   │
 │                       ├──────────────────────>│                   │
 │                       │   password/anonymous  │                   │
 │                       │                       │                   │
 │ 17. "Authentication   │                       │                   │
 │      credentials sent │                       │                   │
 │      successfully!"   │                       │                   │
 │<──────────────────────┤                       │                   │
 │                       │                       │                   │
 │ 18. Finish setup      │                       │                   │
 │     "The SUSI.AI      │                       │                   │
 │      smart device     │                       │                   │
 │      will now shut    │                       │                   │
 │      down the hotspot │                       │                   │
 │      and connect..."  │                       │                   │
 │<──────────────────────┤                       │                   │
 │                       │                       │                   │
 │                       │                       │ 19. Close hotspot │
 │                       │                       │     Connect to    │
 │                       │                       │     home WiFi     │
 │                       │                       │                   │
 │                       │                       │ 20. Register with │
 │                       │                       │     server        │
 │                       │                       ├──────────────────>│
 │                       │                       │                   │
 │ 21. Reconnect phone   │                       │                   │
 │     to home WiFi      │                       │                   │
 │                       │                       │                   │
 │ 22. Verify device     │                       │                   │
 │     in app            │ GET /aaa/             │                   │
 ├──────────────────────>│ listUserDevices.json  │                   │
 │                       ├───────────────────────────────────────────>│
 │                       │                       │                   │
 │                       │ 23. Device list       │                   │
 │                       │<───────────────────────────────────────────┤
 │                       │                       │                   │
 │ 24. Success!          │                       │                   │
 │     "You have         │                       │                   │
 │      configured a     │                       │                   │
 │      SUSI.AI smart    │                       │                   │
 │      device."         │                       │                   │
 │<──────────────────────┤                       │                   │
 │                       │                       │                   │
```

### Setup Steps with Prompts

#### Step 1: Initial Tutorial
**Prompt:** `setup_tut`
```
If you're setting up a new device, make sure it's
 nearby and plugged into a wall outlet
```

#### Step 2: Connection Instructions
**Prompt:** `connect_to_susi`
```
To start the configuration process, plug in your SUSI.AI smart device and wait
for the "Ping" sound. The SUSI.AI device will automatically open a Wi-Fi hotspot.
Please connect your phone to the wireless network with the name "SUSI.AI".
```

#### Step 3: Connection Verification
**Status:** `connected_to_susiai` - "Connected to SUSI.AI WiFi"
**If failed:** `not_connected_susiai_details`

**Troubleshooting:** `cannot_connect_details`
```
Please check that your SUSI.AI smart device is plugged in and the latest SUSI.AI
software is installed. Ensure you are disconnected from other Wi-Fi networks.
```

#### Step 4: WiFi Configuration
**Scanning:** `scan_available_wifi` - "Scanning for available Wi-Fi's"
**Help:** `device_help`
```
Choose your WiFi home network from the list below and enter the password of your
network. Leave blank for networks without a password.
```

#### Step 5: Room Selection
**Prompt:** `device_room` - "Where is this device?"
**Description:** `device_room_exp`
```
Choose a location for your SUSI smart speaker. This will help name and organize
your devices.
```

#### Step 6: Account Linking
**Prompt:** `password_title`
```
Please enter your account password or choose the anonymous mode
```

**Anonymous Warning:** `anonymous_details`
```
If you do not link the SUSI.AI smart device with an online account, you will not
be able to reconfigure your speaker with the mobile app afterwards and add personal
skills of services like music providers. To change anything you will need a hard
reset of the device.
```

#### Step 7: Completion
**Prompt:** `finish_setup_details`
```
The SUSI.AI smart device will now shut down the hotspot and connect with your Wi-Fi
home network and online account. If SUSI.AI is not able to connect to your home Wi-Fi
network after 3 times, it will restart the SUSI.AI hotspot again and you can start
the setup process again.
```

**Success (with account):** `succesfully_setup`
**Success (anonymous):** `success_setup_anonymous`

### Error Handling

**No Devices Found:**
```
Scan completes → No devices
→ Display: "No devices found"
```

**WiFi Connection Failed:**
```
WiFi connect fails → Display: "Wi-Fi connection failed"
Prompt: wifi_connection_failed
```

**Configuration Error:**
```
Setup fails → Display: "Configuration of the speaker failed."
Prompt: config_error
```

### API Endpoints

**Add Device:**
```
GET /aaa/addNewDevice.json
Parameters:
- name: Device name
- room: Room location
- latitude, longitude: Location
- macid: Device MAC address
```

**List Devices:**
```
GET /aaa/listUserDevices.json
Returns: Array of connected devices
```

### Key Files
- `/app/src/main/java/org/fossasia/susi/ai/device/` - Device setup activities
- `/app/src/main/java/org/fossasia/susi/ai/rest/services/SusiService.kt`

---

## Summary

This document has covered 5 major agent pipelines in the SUSI.AI Android application:

1. **Text Query Pipeline** - Core query processing from text input to response
2. **Voice Query Pipeline** - Speech-to-text, voice commands, and text-to-speech
3. **Response Processing Pipeline** - Parsing and rendering different action types
4. **Skill Discovery & Listing Pipeline** - Browsing and rating SUSI skills
5. **Device Setup Pipeline** - Configuring SUSI smart speaker devices

Each pipeline shows:
- Complete data flow with ASCII diagrams
- Prompts used at each step
- Error handling scenarios
- API endpoints and parameters
- Key files and code references

All pipelines work together to create a cohesive user experience, with prompts guiding users through each interaction.
