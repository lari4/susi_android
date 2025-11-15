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
