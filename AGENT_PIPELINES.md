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
