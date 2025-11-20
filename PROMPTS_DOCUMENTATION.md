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

## 6. Device Setup Prompts

These prompts guide users through the process of setting up SUSI.AI smart speaker devices. The setup process involves connecting to the device's Wi-Fi hotspot, configuring home Wi-Fi credentials, and linking to a SUSI account.

### 6.1 Device Setup Tutorial

**Purpose:** Initial instructions for setting up a new SUSI.AI device.

**Usage Context:**
- Displayed when user hasn't connected any devices yet
- Provides guidance before starting setup

**Prompt:**
```
If you're setting up a new device, make sure it's
 nearby and plugged into a wall outlet
```

**Resource ID:** `setup_tut`
**File Location:** `/app/src/main/res/values/strings.xml:315`

---

### 6.2 Connection Instructions

**Purpose:** Detailed step-by-step instructions for connecting to SUSI.AI device.

**Usage Context:**
- Shown during the initial connection step of device setup
- Guides users through connecting their phone to the device's hotspot

**Prompt:**
```
To start the configuration process, plug in your SUSI.AI smart device and wait for the "Ping" sound. The SUSI.AI device will automatically open a Wi-Fi hotspot. Please connect your phone to the wireless network with the name "SUSI.AI".
```

**Resource ID:** `connect_to_susi`
**File Location:** `/app/src/main/res/values/strings.xml:333`

---

### 6.3 Connection Troubleshooting

**Purpose:** Help users when they cannot connect to the SUSI.AI hotspot.

**Usage Context:**
- Displayed when user taps "I cannot connect" button
- Provides troubleshooting guidance

**Prompt:**
```
Please check that your SUSI.AI smart device is plugged in and the latest SUSI.AI software is installed. Ensure you are disconnected from other Wi-Fi networks.
```

**Resource ID:** `cannot_connect_details`
**File Location:** `/app/src/main/res/values/strings.xml:336`

---

### 6.4 Wi-Fi Setup Help

**Purpose:** Instructions when no Wi-Fi networks are found during setup.

**Usage Context:**
- Displayed when Wi-Fi scanning returns no results
- Helps users troubleshoot Wi-Fi availability issues

**Prompt:**
```
Make sure there are Wi-Fi connections available
 near you.
```

**Resource ID:** `setup_wifi`
**File Location:** `/app/src/main/res/values/strings.xml:317`

---

### 6.5 Device Help Instructions

**Purpose:** Instructions for entering Wi-Fi credentials for the device.

**Usage Context:**
- Displayed during Wi-Fi configuration step
- Explains how to provide home network credentials

**Prompt:**
```
Choose your WiFi home network from the list below and enter the password of your network. Leave blank for networks without a password.
```

**Resource ID:** `device_help`
**File Location:** `/app/src/main/res/values/strings.xml:454`

---

### 6.6 Anonymous Mode Warning

**Purpose:** Explains the limitations of using anonymous mode without account linkage.

**Usage Context:**
- Displayed when user chooses anonymous mode during setup
- Warns about functionality limitations

**Prompt:**
```
If you do not link the SUSI.AI smart device with an online account, you will not be able to reconfigure your speaker with the mobile app afterwards and add personal skills of services like music providers. To change anything you will need a hard reset of the device.
```

**Resource ID:** `anonymous_details`
**File Location:** `/app/src/main/res/values/strings.xml:343`

---

### 6.7 Final Setup Instructions

**Purpose:** Explains what happens when finishing the setup process.

**Usage Context:**
- Displayed when user is about to complete setup
- Sets expectations for final configuration steps

**Prompt:**
```
The SUSI.AI smart device will now shut down the hotspot and connect with your Wi-Fi home network and online account. If SUSI.AI is not able to connect to your home Wi-Fi network after 3 times, it will restart the SUSI.AI hotspot again and you can start the setup process again. If SUSI.AI connects successfully with the Internet and your account, it will show up in your list of connected devices and you can continue to configure its settings.
```

**Resource ID:** `finish_setup_details`
**File Location:** `/app/src/main/res/values/strings.xml:347`

---

### 6.8 Successful Setup Message (With Account)

**Purpose:** Success message when device is configured with an account.

**Usage Context:**
- Displayed after successful setup completion with account linkage
- Guides user on next steps

**Prompt:**
```
You have configured a SUSI.AI smart device. Please wait a moment to check if the process was successful and go to the list of "Smart Devices" in your settings to check and continue to configure your device settings. Please connect your phone with the Internet now.
```

**Resource ID:** `succesfully_setup`
**File Location:** `/app/src/main/res/values/strings.xml:350`

---

### 6.9 Successful Setup Message (Anonymous)

**Purpose:** Success message when device is configured in anonymous mode.

**Usage Context:**
- Displayed after successful setup completion without account linkage
- Simpler message for anonymous mode

**Prompt:**
```
You have configured a SUSI.AI smart device in anonymous mode. Please wait a moment to check if the process was successful. Please connect your phone with the Internet now.
```

**Resource ID:** `success_setup_anonymous`
**File Location:** `/app/src/main/res/values/strings.xml:351`

---

### 6.10 Status Messages

Various status messages during the setup process:

#### Scanning for Devices
**Prompt:** `Scanning for devices...`
**Resource ID:** `scan_devices` | **Location:** `/app/src/main/res/values/strings.xml:313`

#### No Devices Found
**Prompt:** `No devices found`
**Resource ID:** `no_device_found` | **Location:** `/app/src/main/res/values/strings.xml:314`

#### Scanning for Wi-Fi
**Prompt:** `Scanning for available Wi-Fi's`
**Resource ID:** `scan_available_wifi` | **Location:** `/app/src/main/res/values/strings.xml:339`

#### No Wi-Fi Found
**Prompt:** `No Wi-Fi-Networks found`
**Resource ID:** `no_wifi_found` | **Location:** `/app/src/main/res/values/strings.xml:316`

#### Device Setting Up
**Prompt:** `Setting up your device`
**Resource ID:** `device_setting_up` | **Location:** `/app/src/main/res/values/strings.xml:324`

#### Connecting to Wi-Fi
**Prompt:** `Connecting to your wifi...`
**Resource ID:** `connecting_device` | **Location:** `/app/src/main/res/values/strings.xml:455`

#### Connection Success
**Prompt:** `Connected Successfully !`
**Resource ID:** `connection_success` | **Location:** `/app/src/main/res/values/strings.xml:457`

#### Device Connected
**Prompt:** `Device Connected successfully!`
**Resource ID:** `connect_success` | **Location:** `/app/src/main/res/values/strings.xml:304`

#### Wi-Fi Credentials Sent
**Prompt:** `Wi-Fi credentials sent successfully!`
**Resource ID:** `wifi_success` | **Location:** `/app/src/main/res/values/strings.xml:306`

#### Authentication Credentials Sent
**Prompt:** `Authentication credentials sent successfully!`
**Resource ID:** `auth_success` | **Location:** `/app/src/main/res/values/strings.xml:305`

---

## 7. Onboarding & Tutorial Prompts

These prompts are shown to new users during the app's onboarding slides to introduce key features.

### 7.1 Slide 1: Login

**Purpose:** Introduces users to login options.

**Usage Context:**
- First onboarding slide
- Explains authentication options

**Title:**
```
Log in
```

**Description:**
```
Log in on the app using a SUSI.AI account, sign up to create a new account or just skip logging in altogether.
```

**Resource IDs:** `slide_1_title`, `slide_1_desc`
**File Location:** `/app/src/main/res/values/strings.xml:411-412`

---

### 7.2 Slide 2: Chat Interface

**Purpose:** Introduces the main chat interface and voice interaction.

**Usage Context:**
- Second onboarding slide
- Explains how to interact with SUSI

**Title:**
```
Chat Interface
```

**Description:**
```
Interact with SUSI.AI by asking questions. Use the microphone button for voice interaction.
```

**Resource IDs:** `slide_2_title`, `slide_2_desc`
**File Location:** `/app/src/main/res/values/strings.xml:413-414`

---

### 7.3 Slide 3: Skill List

**Purpose:** Introduces the skills browsing feature.

**Usage Context:**
- Third onboarding slide
- Explains skills discovery

**Title:**
```
Skill List
```

**Description:**
```
Browse and try your favorite SUSI.AI skill
```

**Resource IDs:** `slide_3_title`, `slide_3_desc`
**File Location:** `/app/src/main/res/values/strings.xml:415-416`

---

### 7.4 Slide 4: Chat Settings

**Purpose:** Introduces personalization options.

**Usage Context:**
- Fourth onboarding slide
- Explains settings customization

**Title:**
```
Chat Settings
```

**Description:**
```
Personalize your chat settings for better experience.
```

**Resource IDs:** `slide_4_title`, `slide_4_desc`
**File Location:** `/app/src/main/res/values/strings.xml:417-418`

---

## 8. Help & Instruction Prompts

These prompts provide detailed instructions for common user tasks and questions.

### 8.1 How to Use SUSI Voice

**Question:**
```
How to use SUSI voice?
```

**Answer:**
```
To use SUSI voice click on the microphone symbol present in the bottom right corner of the chats page. You can ask anything to SUSI here. To change various voice settings, go to Settings page and configure SUSI voice accordingly.
```

**Resource IDs:** `help_susi_voice`, `help_susi_voice_desc`
**File Location:** `/app/src/main/res/values/strings.xml:514-515`

---

### 8.2 What Skills Does SUSI Have

**Question:**
```
What skills do SUSI have?
```

**Answer:**
```
To know about the skills that SUSI posses move to 'SUSI.AI Skills' page. You can reach there just by clicking on the 'SUSI' logo present at the top right corner of the Chat page.
```

**Resource IDs:** `help_susi_skills`, `help_susi_skills_desc`
**File Location:** `/app/src/main/res/values/strings.xml:516-517`

---

### 8.3 How to Change SUSI Voice

**Question:**
```
How to change SUSI voice ?
```

**Answer:**
```
In order to change voice and pitch of Susi, go to 'Settings ' and scroll down until you find 'Click here to change SUSI voice' and change voice according to your preference.
```

**Resource IDs:** `help_susi_changevoice`, `help_susi_changevoice_desc`
**File Location:** `/app/src/main/res/values/strings.xml:518-519`

---

### 8.4 How to Open Settings

**Question:**
```
How to open 'Settings'?
```

**Answer:**
```
In the main page click on SUSI logo present at the top right corner. This will open a page. Click on the 3 dot present at top right corner. From the menu that open click on 'Settings'
```

**Resource IDs:** `help_open_setting`, `help_open_setting_desc`
**File Location:** `/app/src/main/res/values/strings.xml:504-505`

---

### 8.5 How to Sign Up

**Question:**
```
How to SignUp in SUSI.AI?
```

**Answer:**
```
In the Login page click on 'Sign Up for SUSI'.
 * Enter your email address.
 * Enter your password and confirm it
 After this click on 'SIGN UP'
```

**Resource IDs:** `help_signup`, `help_signup_desc`
**File Location:** `/app/src/main/res/values/strings.xml:506-507`

---

### 8.6 How to Change Password

**Question:**
```
How to change Password?
```

**Answer:**
```
 * Open the 'Settings' page.
 * Scroll down and click on 'Reset Password'.
 * Enter your current password and after that entire your new password.
 * Click on 'OK'.
```

**Resource IDs:** `help_change_password`, `help_change_password_desc`
**File Location:** `/app/src/main/res/values/strings.xml:508-509`

---

### 8.7 How to Report an Issue

**Question:**
```
How to report an issue?
```

**Answer:**
```
Open the 'About Us' page. Under the 'Report Issues' section click on  'Github Repository Issue Tracker'. This will redirect you to a webpage where you can put up your issue by clicking on 'New Issue'.
```

**Resource IDs:** `help_report_issue`, `help_report_issue_desc`
**File Location:** `/app/src/main/res/values/strings.xml:510-511`

---

### 8.8 How to Login (After Skipping)

**Question:**
```
How to login, if you have opened SUSI without logging in?
```

**Answer:**
```
Open 'Settings' page and scroll down to the bottom. Click on 'Login'. This will redirect you to the login page.
```

**Resource IDs:** `help_login`, `help_login_desc`
**File Location:** `/app/src/main/res/values/strings.xml:512-513`

---

## 9. About SUSI Description

This is the main description of what SUSI.AI is, used in the About section of the app.

**Purpose:** Explains SUSI.AI's capabilities and nature to users.

**Usage Context:**
- Displayed in the About Us section
- Helps users understand what SUSI can do

**Prompt:**
```
SUSI.AI is an intelligent libre software personal assistant. It is capable of chat and voice interaction by using APIs to perform actions such as music playback, making to-do lists, setting alarms, streaming podcasts, playing audiobooks, and providing weather, traffic, and other real time information. Additional functionalities can be added as console services using external APIs. SUSI.AI is able to answer questions and depending on the context will ask for additional information in order to perform the desired outcome. The core of the assistant is the SUSI.AI server that holds the "intelligence" and "personality" of SUSI.AI.
```

**Resource ID:** `susi_about`
**File Location:** `/app/src/main/res/values/strings.xml:355`

---

## Summary

This documentation covers all user-facing prompts in the SUSI.AI Android application organized into 9 main categories:

1. **Voice Interaction Prompts** - Voice greetings and input instructions
2. **User Input Prompts** - Chat interface guidance
3. **Skill Discovery Prompts** - Metric-based skill queries
4. **Example Voice Commands** - Sample commands to demonstrate capabilities
5. **Error Handling Prompts** - Error messages and troubleshooting
6. **Device Setup Prompts** - Smart speaker configuration instructions
7. **Onboarding & Tutorial Prompts** - First-time user guidance
8. **Help & Instruction Prompts** - Detailed how-to guides
9. **About SUSI Description** - Core product description

All prompts are localized and available in multiple languages including English, German, Spanish, Italian, Hindi, Tamil, Malayalam, and Portuguese (Brazilian).
