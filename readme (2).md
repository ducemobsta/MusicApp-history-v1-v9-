# LLM Chained Orchestrator Hub AI Music Generation App

This application leverages a chained Large Language Model (LLM) orchestration to generate music based on user-defined parameters.

## Project Build

This project is a single-page application built with React and TypeScript, styled with Tailwind CSS. It runs entirely in the browser.

### Dependencies
- React & ReactDOM
- Tailwind CSS (via CDN)
- Font Awesome (via CDN)

## Build Instructions

There is no complex build process required to run this application locally. A modern browser with JavaScript enabled is sufficient.

1.  **Download Files**: Ensure you have `index.html` and `index.tsx` (and its compiled JS output, usually `index.js`) in the same directory.
2.  **Run**: Simply open the `index.html` file in your web browser.

## Deployment

To deploy this application, you can host the static files (`index.html`, and the compiled `index.js`) on any static web hosting service like GitHub Pages, Netlify, or Vercel.

## How It Works

The application uses a visual pipeline of "agents". The user provides a prompt, and the app simulates sending this to a backend which processes the request through each agent in the chain. The results are displayed in the UI.

## Features

-   **Music Generation**: Simulate creating music in various genres.
-   **Parameter Control**: Adjust genre, key, tempo, and mood.
-   **Visual Pipeline**: See the status of each agent in the generation process.
-   **Save/Load Project**: Save your current settings and generated data to a JSON file, and load it back later to continue your work.