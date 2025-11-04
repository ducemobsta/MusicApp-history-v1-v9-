
# Hybrid AI Music Studio

## 🎵 Description

The Hybrid AI Music Studio is a cutting-edge, browser-based music creation tool that combines procedural, rule-based music generation with the power of Google's Gemini API for AI-driven singing synthesis. Users can craft unique instrumentals by selecting genres, keys, and complexity, enhance them with custom audio samples and high-quality SoundFonts (.sf2), and then bring them to life with AI-generated vocals based on their own lyrics.

This project is self-contained and designed to run directly from static files, with all dependencies loaded from a CDN.

## ✨ Features

- **Procedural Music Generation:** Create beats and melodies in various genres (Trap, Trap Soul, Lo-fi, Drill).
- **AI Singing Synthesis:** Use the Gemini API to generate sung vocals from text prompts.
- **Custom Sample Support:** Upload your own `.wav`, `.mp3`, or `.ogg` files for drums (kick, snare, hi-hat).
- **SoundFont Integration:** Upload `.sf2` files to use high-quality, realistic instruments for melodies and chords.
- **Real-time Playback:** Audition your creations instantly with Tone.js.
- **WYSIWYG Recording:** Download your final track (instrumentals + vocals) as a high-quality `.wav` file.
- **Dynamic UI:** A futuristic, responsive interface with a real-time audio visualizer.
- **Zero Build-Step Required:** Runs directly from an `index.html` file.

## 🛠️ Tech Stack

- **Frontend:** React, TypeScript
- **Styling:** Tailwind CSS (via CDN)
- **Audio:** Tone.js, soundfont-player
- **AI Model:** Google Gemini API (`gemini-2.5-flash-preview-tts`)
- **Icons:** Lucide React

## 🚀 Getting Started / Rebuilding the App

Follow these instructions to set up and run the project locally or integrate it into your own environment.

### Prerequisites

1.  A modern web browser (Chrome, Firefox, Safari, Edge).
2.  A local web server to serve the files. You can use the `serve` package from npm: `npm install -g serve`.
3.  A Google Gemini API key.

### File Structure

Ensure you have the following files organized in a single project directory:

```
/
├── index.html              # Main HTML entry point, loads all scripts
├── index.tsx               # React application root
├── App.tsx                 # Main application component and logic
├── constants.ts            # Genre definitions, scales, and music theory constants
├── types.ts                # TypeScript type definitions for the application
├── metadata.json           # Application metadata
│
├── components/
│   └── icons.tsx           # Lucide icon imports
│
├── services/
│   ├── musicService.ts     # Rule-based instrumental generation logic
│   └── geminiService.ts    # Handles communication with the Gemini API
│
└── utils/
    └── audioUtils.ts       # Audio decoding and WAV encoding utilities
```

### Setup Instructions

1.  **Place Files:** Arrange all the files listed above into a single directory according to the specified structure.

2.  **Configure API Key:**
    The application requires a Google Gemini API key. The `geminiService.ts` file is hardcoded to look for `process.env.API_KEY`. In a development environment or a platform like AI Studio, this environment variable is typically injected for you. If running a simple static server, you will need to replace `process.env.API_KEY` in `services/geminiService.ts` directly with your key string for it to work.
    
    *Open `services/geminiService.ts` and find this line:*
    ```typescript
    const ai = new GoogleGenAI({ apiKey: process.env.API_KEY });
    ```
    *Replace it with your key if needed for local testing:*
    ```typescript
    const ai = new GoogleGenAI({ apiKey: "YOUR_API_KEY_HERE" });
    ```
    **Note:** Be careful not to expose your API key in public repositories.

3.  **Serve the Project:**
    Navigate to your project directory in your terminal and start a local server.
    ```bash
    # If you have the 'serve' package installed
    serve .
    ```
    The server will give you a local URL, usually `http://localhost:3000`.

4.  **Run the App:**
    Open the local URL in your web browser. The Hybrid AI Music Studio should now be running.

## ⚙️ How It Works

The application follows a simple, creative workflow:

1.  **Customize Parameters:** Select a genre, root note, and complexity level.
2.  **Upload Samples (Optional):** Upload your own drum samples or a SoundFont file to customize the sound palette.
3.  **Generate Instrumental:** Click "Generate Instrumental" to create and listen to a procedurally generated music loop. You can stop and re-generate as many times as you like.
4.  **Write Lyrics:** Enter lyrics into the text area.
5.  **Add AI Vocals:** Once you're happy with the instrumental, click "Add AI Vocals & Play". The app sends your lyrics to the Gemini API, which returns a sung vocal track that is then layered over your instrumental.
6.  **Download:** Click "Record & Download" to save the complete track as a `.wav` file.

## 🎨 Customization

You can easily extend the app by adding new genres.

1.  Open `constants.ts`.
2.  Add a new genre key to the `GENRE_MUSIC_LOGIC` object, following the existing structure for tempo, scales, chord progressions, and rhythm patterns.
3.  Add the new genre name to the `ALL_GENRES` array.
4.  Update the `Genre` type in `types.ts` to include your new genre.

The UI will automatically update to include your new genre in the dropdown menu.
