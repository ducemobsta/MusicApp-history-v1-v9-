# Orchestrator AI Music Studio

An advanced AI music generation application that uses a pipeline of specialized agents to create original music in various genres. It features procedural generation, real-time audio playback with visualization, custom sample support, and WAV export capabilities.

## Project Structure

This project consists of four essential files:
- `index.html`: The main HTML file that sets up the page and imports the necessary scripts.
- `index.tsx`: The entry point for the React application.
- `App.tsx`: The main React component containing all the application logic and UI.
- `metadata.json`: Configuration for the application environment.

## Build Instructions

This is a static, client-side application that runs entirely in the browser. To rebuild or run this project from scratch, follow these steps:

### Step 1: Set up the files

Create the following files in a single directory with the exact content provided below.

<details>
<summary><code>index.html</code></summary>

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Orchestrator AI Music Studio</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/tone@14.7.77/build/Tone.js"></script>
  <script type="importmap">
{
  "imports": {
    "react/": "https://aistudiocdn.com/react@^19.2.0/",
    "react": "https://aistudiocdn.com/react@^19.2.0",
    "react-dom/": "https://aistudiocdn.com/react-dom@^19.2.0/",
    "tone": "https://aistudiocdn.com/tone@^15.1.22",
    "@google/genai": "https://aistudiocdn.com/@google/genai@^0.14.0"
  }
}
</script>
</head>
  <body class="bg-gray-900">
    <div id="root"></div>
  </body>
</html>
```
</details>

<details>
<summary><code>index.tsx</code></summary>

```tsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';

const rootElement = document.getElementById('root');
if (!rootElement) {
  throw new Error("Could not find root element to mount to");
}

const root = ReactDOM.createRoot(rootElement);
root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```
</details>

<details>
<summary><code>App.tsx</code></summary>

```tsx
// The full content of the App.tsx file you are currently running.
// You can get the latest version by saving the file in your development environment.
```
</details>

<details>
<summary><code>metadata.json</code></summary>

```json
{
  "name": "Orchestrator AI Music Studio",
  "description": "An advanced AI music generation application that uses a pipeline of specialized agents to create original music in various genres. It features procedural generation, real-time audio playback with visualization, custom sample support, and WAV export capabilities.",
  "requestFramePermissions": []
}
```
</details>

### Step 2: Set up your Gemini API Key

This application requires a Google Gemini API key to function.

1.  Obtain your API key from [Google AI Studio](https://aistudio.google.com/app/apikey).
2.  The application is configured to read the API key from an environment variable (`process.env.API_KEY`). You will need to set this variable in the environment where you are running the application.

### Step 3: Run the Application

Since this is a client-side application with no build step, you can run it by simply opening the `index.html` file in a modern web browser that supports ES modules. For best results, serve the directory using a simple local web server.

- **Using Python:**
  ```bash
  # For Python 3
  python -m http.server
  ```
- **Using Node.js:**
  Install `serve` globally:
  ```bash
  npm install -g serve
  ```
  Then run it in your project directory:
  ```bash
  serve
  ```
Then open `http://localhost:8000` (or the port specified by your server) in your browser.