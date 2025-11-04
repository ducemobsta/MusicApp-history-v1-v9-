
# AI Music Studio

This is a React + TypeScript application for AI-powered music generation using Google Gemini and Tone.js.

## Project Setup

### 1. Prerequisites
- Node.js (version 18 or newer)
- npm or yarn package manager

### 2. Create a New React Project
This project was bootstrapped with Vite. You can set up a similar project using:
```bash
npm create vite@latest ai-music-studio -- --template react-ts
cd ai-music-studio
```

### 3. Install Dependencies
The application requires several libraries for audio processing, file handling, and UI.
```bash
npm install tone jszip file-saver @google/genai
```

### 4. Environment Variables
To use the AI generation features, you need a Google Gemini API key. Create a `.env.local` file in the root of your project and add your key:
```
VITE_GEMINI_API_KEY=YOUR_API_KEY_HERE
```
*Note: In the provided code, the key is accessed via `process.env.API_KEY`. For Vite, you must prefix it with `VITE_` and access it via `import.meta.env.VITE_GEMINI_API_KEY`. The generated code assumes a generic `process.env` setup.*

### 5. Start the Development Server
```bash
npm run dev
```
This command will start the Vite development server. Open your web browser and navigate to the local URL provided in the terminal (usually `http://localhost:5173`).

## How to Use

1.  **AI Generation**: Type a prompt into the "Describe your music..." text area (e.g., "a dark trap beat for a villain scene, like Zaytoven"). Click "Generate with AI". The AI will configure the settings for you.
2.  **Manual Generation**: Alternatively, adjust the Genre, Producer, Key, and other sliders and controls manually. Click "Generate".
3.  **Playback & Visualize**: Once generation is complete, the "Play" button will become active. Click it to listen. The audio visualizer will display the waveform in real-time.
4.  **Save Your Creation**: If you like what you hear, click the "Save WAV" button. Your browser will prompt you to download a `.wav` file of the full mix.
