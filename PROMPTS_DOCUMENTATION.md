# SUSI.AI Android Application - Prompts Documentation

## Overview

This document contains all AI-related prompts, user interface prompts, and conversational templates used in the SUSI.AI Android application. The SUSI.AI Android app is a client application that communicates with the SUSI.AI server backend. While the AI intelligence resides on the server side, the app contains various prompts for user interaction, guidance, and communication with the SUSI assistant.

**Note:** All prompts are defined in `/app/src/main/res/values/strings.xml` and related resource files, with translations available in multiple languages.

---

## 1. Voice Interaction Prompts

These prompts are used for voice-based interaction with SUSI. They guide users through voice input and provide audio feedback.

### 1.1 Welcome Greeting (First Voice Interaction)

**Purpose:** This prompt is played via Text-to-Speech when a user uses voice interaction for the first time. It introduces SUSI and offers help.

**Usage Context:**
- Triggered in `STTfragment.kt:68` when the user hasn't used voice before
- Played automatically using TextToSpeech engine
- Only plays once per installation (controlled by preference flag `used_voice`)

**Prompt:**
```
Hi! How can I help you?
```

**Resource ID:** `voice_welcome`
**File Location:** `/app/src/main/res/values/strings.xml:14`

---

### 1.2 Voice Input Hint

**Purpose:** Placeholder text displayed in the voice input interface to prompt the user to speak.

**Usage Context:**
- Shown in the speech-to-text fragment UI
- Indicates that SUSI is ready to receive voice input

**Prompt:**
```
Say Something…
```

**Resource ID:** `hint_voice_input`
**File Location:** `/app/src/main/res/values/strings.xml:40`

---

### 1.3 Speech Prompt

**Purpose:** Alternative prompt for voice input, used in different contexts within the speech interface.

**Usage Context:**
- Generic speech prompt for voice input screens
- Used as a placeholder or instruction text

**Prompt:**
```
Say something…
```

**Resource ID:** `speech_prompt`
**File Location:** `/app/src/main/res/values/strings.xml:128`

---

### 1.4 Microphone Tap Instruction

**Purpose:** Instructs users on how to activate voice input.

**Usage Context:**
- Displayed to guide users who may not know how to start voice interaction
- Helps first-time users understand the voice interface

**Prompt:**
```
Tap the mic icon to speak
```

**Resource ID:** `tap_on_mic`
**File Location:** `/app/src/main/res/values/strings.xml:129`

---

## 2. User Input Prompts

These prompts guide users on how to interact with SUSI through text chat.

### 2.1 Chat Input Hint

**Purpose:** Placeholder text in the main chat input field that prompts users to ask SUSI a question.

**Usage Context:**
- Displayed in the text input field of the main chat interface
- Encourages users to interact with SUSI
- Provides context about what users can do

**Prompt:**
```
Ask SUSI something…
```

**Resource ID:** `send_msg_hint`
**File Location:** `/app/src/main/res/values/strings.xml:84`

---

### 2.2 Help Offer

**Purpose:** A friendly prompt asking how SUSI can assist the user.

**Usage Context:**
- Used in various UI contexts where SUSI offers help
- General assistance prompt

**Prompt:**
```
How can I help you?
```

**Resource ID:** `how_can_i_help`
**File Location:** `/app/src/main/res/values/strings.xml:388`

---

### 2.3 Unknown Answer Response

**Purpose:** Default response when SUSI doesn't know how to answer a question or when there's no server response available.

**Usage Context:**
- Fallback response for unrecognized queries
- Displayed when the AI cannot provide a meaningful answer
- Used as a client-side placeholder when server communication fails

**Prompt:**
```
I don't know.
```

**Resource ID:** `unknown_answer`
**File Location:** `/app/src/main/res/values/strings.xml:213`

---

## 3. Skill Discovery Prompts (Metric Queries)

These are pre-defined query prompts that help users discover SUSI's capabilities and skills. They are displayed as clickable suggestions in the skills browsing interface to help users explore different skill categories.

### 3.1 Highest Rated Skills Query

**Purpose:** Example query to help users discover the top-rated skills available in SUSI.

**Usage Context:**
- Displayed as a suggested query in the skills section
- When clicked or spoken, it triggers a search for highest rated skills
- Used in `SkillListingPresenter.kt` for UI display

**Prompt:**
```
SUSI, What are your highest rated skills?
```

**Resource ID:** `metric_rating`
**File Location:** `/app/src/main/res/values/strings.xml:487`

---

### 3.2 Most Used Skills Query

**Purpose:** Example query to help users discover the most popular skills based on usage statistics.

**Usage Context:**
- Displayed as a suggested query in the skills section
- Helps users find skills that are frequently used by the community

**Prompt:**
```
SUSI, what are your most used skills?
```

**Resource ID:** `metric_usage`
**File Location:** `/app/src/main/res/values/strings.xml:488`

---

### 3.3 Recently Updated Skills Query

**Purpose:** Example query to help users discover skills that have been recently updated or improved.

**Usage Context:**
- Displayed as a suggested query in the skills section
- Helps users find the latest improvements and updates

**Prompt:**
```
SUSI, what are the recently updated skills?
```

**Resource ID:** `metric_latest`
**File Location:** `/app/src/main/res/values/strings.xml:489`

---

### 3.4 Skills with Most Feedback Query

**Purpose:** Example query to help users discover skills that have received the most user feedback.

**Usage Context:**
- Displayed as a suggested query in the skills section
- Helps users find skills that are actively discussed and reviewed

**Prompt:**
```
SUSI, what are the skills with most feedback?
```

**Resource ID:** `metric_feedback`
**File Location:** `/app/src/main/res/values/strings.xml:490`

---

### 3.5 Newest Skills Query

**Purpose:** Example query to help users discover the newest skills added to SUSI.

**Usage Context:**
- Displayed as a suggested query in the skills section
- Helps users explore recently added functionality

**Prompt:**
```
SUSI, what are the newest skills?
```

**Resource ID:** `metric_newest`
**File Location:** `/app/src/main/res/values/strings.xml:491`

---

### 3.6 Top Games Query

**Purpose:** Example query to help users discover gaming skills available in SUSI.

**Usage Context:**
- Displayed as a suggested query in the skills section
- Specifically targets entertainment and gaming capabilities

**Prompt:**
```
SUSI, what are your top games?
```

**Resource ID:** `metrics_top_games`
**File Location:** `/app/src/main/res/values/strings.xml:492`

---

## 4. Example Voice Commands

These are sample commands displayed to users to demonstrate SUSI's capabilities. They are shown as clickable suggestions in the voice interface to help users understand what kinds of questions they can ask.

**Usage Context:**
- Displayed horizontally in the voice input screen (`STTfragment.kt:82-85`)
- Shown as clickable chips/buttons that users can tap to quickly send a command
- Help new users understand SUSI's capabilities
- Defined as an array in `/app/src/main/res/values/array.xml:40-48`

### Example Commands List:

#### 4.1 Time Query
**Prompt:**
```
What is the time now?
```
**Category:** Time & Date
**Purpose:** Demonstrates SUSI can provide current time information

---

#### 4.2 Geography/Trivia Query
**Prompt:**
```
Which is the largest country in the world?
```
**Category:** General Knowledge
**Purpose:** Demonstrates SUSI can answer general knowledge questions

---

#### 4.3 App Control Command
**Prompt:**
```
Open WhatsApp
```
**Category:** Device Control
**Purpose:** Demonstrates SUSI can launch applications on the device

---

#### 4.4 Location Query
**Prompt:**
```
Where is Singapore?
```
**Category:** Geography
**Purpose:** Demonstrates SUSI can provide location and geographic information

---

#### 4.5 News Query
**Prompt:**
```
Show me news headlines
```
**Category:** News & Information
**Purpose:** Demonstrates SUSI can fetch and display current news

---

#### 4.6 Information Query
**Prompt:**
```
Who is the president of India?
```
**Category:** Current Affairs / General Knowledge
**Purpose:** Demonstrates SUSI can answer factual questions about world leaders

---

**Array Resource ID:** `voiceCommands`
**File Location:** `/app/src/main/res/values/array.xml:40-48`

---

## 5. Error Handling Prompts

These prompts inform users about errors and problems that occur during interaction with SUSI.

### 5.1 General Error Message

**Purpose:** Generic error message displayed when an unexpected error occurs.

**Usage Context:**
- Displayed when API calls fail
- Shown when unexpected errors occur during operation
- General fallback error message

**Prompt:**
```
An error occurred. please try again.
```

**Resource ID:** `error_occurred_try_again`
**File Location:** `/app/src/main/res/values/strings.xml:142`

---

### 5.2 Internet Connectivity Error

**Purpose:** Informs users that there's a problem with their internet connection.

**Usage Context:**
- Displayed when the app cannot reach the SUSI server
- Shown when network requests fail due to connectivity issues
- Helps users understand they need to check their internet connection

**Prompt:**
```
Internet Connectivity Problem.
```

**Resource ID:** `error_internet_connectivity`
**File Location:** `/app/src/main/res/values/strings.xml:140`

---

### 5.3 No Internet Connection

**Purpose:** Alternative message for internet connectivity problems.

**Usage Context:**
- More explicit message about lack of internet connection
- Used in different contexts from the connectivity problem message

**Prompt:**
```
Internet Connection Not Available.
```

**Resource ID:** `no_internet_connection`
**File Location:** `/app/src/main/res/values/strings.xml:65`

---

### 5.4 Hotword Detection Error

**Purpose:** Informs users that hotword detection (wake word "SUSI") is not supported on their device.

**Usage Context:**
- Displayed when device hardware/software doesn't support hotword detection
- Shown when trying to enable hotword detection on incompatible devices

**Prompt:**
```
Sorry, hotword detection is not supported on your device.
```

**Resource ID:** `error_hotword`
**File Location:** `/app/src/main/res/values/strings.xml:139`

---

### 5.5 Hotword Detected Success

**Purpose:** Confirmation message when the hotword "SUSI" is successfully detected.

**Usage Context:**
- Displayed when the wake word is recognized
- Provides feedback that voice interaction is starting

**Prompt:**
```
Hotword detected
```

**Resource ID:** `hotword_success`
**File Location:** `/app/src/main/res/values/strings.xml:42`

---

### 5.6 Voice Search Not Supported

**Purpose:** Informs users that their device doesn't support voice actions.

**Usage Context:**
- Displayed when device lacks voice recognition capabilities
- Shown when trying to use voice features on incompatible devices

**Prompt:**
```
Sorry, your device does not support voice actions.
```

**Resource ID:** `error_voice_search`
**File Location:** `/app/src/main/res/values/strings.xml:150`

---

### 5.7 Speech Not Supported

**Purpose:** Informs users that their device doesn't support speech input.

**Usage Context:**
- Displayed when device lacks speech recognition hardware/software
- Alternative to voice search error for speech input specifically

**Prompt:**
```
Sorry, your device doesn't support speech input
```

**Resource ID:** `speech_not_supported`
**File Location:** `/app/src/main/res/values/strings.xml:127`

---

### 5.8 Voice Chat Search Error

**Purpose:** Error message for failures during voice-based search in chat.

**Usage Context:**
- Displayed when voice search within chat history fails
- Specific to voice search functionality errors

**Prompt:**
```
Error occured while doing voice search.
```

**Resource ID:** `error_voice_chat_search`
**File Location:** `/app/src/main/res/values/strings.xml:151`

---

### 5.9 Chat Search Not Found

**Purpose:** Informs users that their search query didn't match any messages.

**Usage Context:**
- Displayed when searching chat history returns no results
- Helps users understand their search term wasn't found

**Prompt:**
```
Search Not Found
```

**Resource ID:** `chat_search_status`
**File Location:** `/app/src/main/res/values/strings.xml:43`

---

### 5.10 No Search Results (Upward Search)

**Purpose:** Message when searching upward in chat finds no matches.

**Usage Context:**
- Displayed when user searches above current position in chat
- Indicates no matching messages exist above

**Prompt:**
```
Nothing above matching your query
```

**Resource ID:** `nothing_up_matches_your_query`
**File Location:** `/app/src/main/res/values/strings.xml:70`

---

### 5.11 No Search Results (Downward Search)

**Purpose:** Message when searching downward in chat finds no matches.

**Usage Context:**
- Displayed when user searches below current position in chat
- Indicates no matching messages exist below

**Prompt:**
```
Nothing below matching your query
```

**Resource ID:** `nothing_down_matches_your_query`
**File Location:** `/app/src/main/res/values/strings.xml:69`

---
