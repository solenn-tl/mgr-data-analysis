# Conferences analysis platform: High-performance data streaming pipeline
This repository contains a high-performance, single-file web application built with Vue 3, Tailwind CSS, and PapaParse. The application is specifically designed to index, align, filter, and inspect ultra-large discourse transcripts and multi-tiered document collections without crashing the web browser or causing interface lag.

## Local setup and use
Because the application relies on advanced streaming network features (ReadableStream), web browsers require the file to be delivered via an HTTP server environment rather than being opened directly from disk.

### Step 1: Place Files Together
Ensure your files are arranged in your project folder as follows:
```plaintext
app/
│
├── index.html                       # The downloaded application file
├── merged_no_transcriptions.json    # Structural dataset
└── merged_sentences.csv             # Transcription mapping table
```
### Step 2: Spin Up a Local Server
Open your terminal inside that directory and start a web server using one of the following commands or a software of you choice (Live Server in VsCode is also a possible choice). 
Go the local address of your server to open the application.

## Use of AI
This plateform has been produced with the assistance of a generative AI (Gemini Flash 3.5 - Extended).

## Data sources
The conferences and transcription have been provided by the Philharmonie of Paris - Cité de la Musique. 
Transcriptions have been created using the Whisper X framework.