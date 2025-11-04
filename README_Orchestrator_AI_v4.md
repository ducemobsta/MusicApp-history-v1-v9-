# Orchestrator AI Music Studio v4 - Build Instructions

This document provides complete instructions to rebuild this application from scratch using the files provided.

## Project Overview

Orchestrator AI Music Studio is an AI-powered music generation application that uses a chained orchestration of Large Language Model (LLM) agents to create musical compositions from user prompts. Users can customize genres, keys, tempos, and even provide their own audio samples for instruments. The application visualizes the entire creation process and allows for in-browser playback and WAV export of the final track.

## File Structure

```
/
|-- index.html
|-- index.tsx
|-- metadata.json
|-- App.tsx
|-- types.ts
|-- constants.ts
|-- components/
|   |-- ControlPanel.tsx
|   |-- OutputDisplay.tsx
|   |-- ChainFlow.tsx
|   |-- AudioControls.tsx
|   |-- AgentModelSelector.tsx
|   |-- SettingsMenu.tsx
|-- services/
    |-- geminiService.ts
    |-- audioService.ts
    |-- readmeService.ts

```

## File Contents

### `index.html`

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF--8" />
    <link rel="icon" type="image/svg+xml" href="/vite.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Orchestrator AI Music Studio</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/tone/14.7.77/Tone.js"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css" integrity="sha512-DTOQO9RWCH3ppGqcWaEA1BIZOC6xxalwEsw9c2QQeAIftl+Vegovlnee1c9QX4TctnWMn13TZye+giMm8e2LwA==" crossorigin="anonymous" referrerpolicy="no-referrer" />
  <script type="importmap">
{
  "imports": {
    "react": "https://aistudiocdn.com/react@^19.2.0",
    "react-dom/": "https://aistudiocdn.com/react-dom@^19.2.0/",
    "react/": "https://aistudiocdn.com/react@^19.2.0/",
    "@google/genai": "https://aistudiocdn.com/@google/genai@^1.28.0",
    "tone": "https://aistudiocdn.com/tone@^15.1.22"
  }
}
</script>
</head>
  <body class="bg-gray-900 text-white font-sans">
    <div id="root"></div>
    <script type="module" src="/index.tsx"></script>
  </body>
</html>
```

### `index.tsx`

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

### `metadata.json`

```json
{
  "name": "Orchestrator AI Music Studio",
  "description": "An AI-powered music generation application that uses a chained orchestration of LLM agents to create musical compositions from user prompts. Visualize the entire creation process and listen to the final output synthesized in your browser.",
  "requestFramePermissions": []
}
```

### `App.tsx`

```tsx
import React, { useState, useCallback, useRef, useEffect } from 'react';
import { ControlPanel } from './components/ControlPanel';
import { OutputDisplay } from './components/OutputDisplay';
import { ChainFlow } from './components/ChainFlow';
import { AudioControls } from './components/AudioControls';
import { SettingsMenu } from './components/SettingsMenu';
import { orchestrateMusicGeneration } from './services/geminiService';
import { AudioService } from './services/audioService';
import { Agent, GenerationSettings, AgentName, FinalComposition, SampleMap, AIModel } from './types';
import { AGENT_PIPELINE } from './constants';

const App: React.FC = () => {
    const [settings, setSettings] = useState<GenerationSettings>({
        genre: 'Trap',
        key: 'A minor',
        tempo: 120,
        mood: 'dark and atmospheric',
        prompt: 'Create a beat for a late-night coding session.',
    });
    const [agents, setAgents] = useState<Agent[]>(
        AGENT_PIPELINE.map(name => ({ 
            name, 
            status: 'idle', 
            output: null,
            model: 'gemini-2.5-flash' // Default model
        }))
    );
    const [samples, setSamples] = useState<SampleMap>({
        kick: null,
        snare: null,
        melody: null,
        pads: null,
        counterMelody: null,
    });
    const [isGenerating, setIsGenerating] = useState<boolean>(false);
    const [isPlaying, setIsPlaying] = useState<boolean>(false);
    const [finalComposition, setFinalComposition] = useState<FinalComposition | null>(null);

    const audioServiceRef = useRef<AudioService | null>(null);

    useEffect(() => {
        audioServiceRef.current = new AudioService();
        return () => {
            audioServiceRef.current?.dispose();
        };
    }, []);

    const handleGenerate = useCallback(async () => {
        setIsGenerating(true);
        setIsPlaying(false);
        setFinalComposition(null);
        audioServiceRef.current?.stop();

        setAgents(prev => prev.map(a => ({ ...a, status: 'idle', output: null })));

        const onAgentComplete = (name: AgentName, output: any) => {
            setAgents(prev =>
                prev.map(agent =>
                    agent.name === name ? { ...agent, status: 'complete', output } : agent
                )
            );
        };
        
        const onAgentStart = (name: AgentName) => {
             setAgents(prev =>
                prev.map(agent =>
                    agent.name === name ? { ...agent, status: 'processing', output: null } : agent
                )
            );
        };

        try {
            const result = await orchestrateMusicGeneration(settings, agents, onAgentStart, onAgentComplete);
            setFinalComposition(result);
            await audioServiceRef.current?.initialize();
            await audioServiceRef.current?.scheduleMusic(result, samples);
        } catch (error) {
            console.error('Error during music generation:', error);
            setAgents(prev => prev.map(a => ({...a, status: a.status === 'processing' ? 'error' : a.status})));
        } finally {
            setIsGenerating(false);
        }
    }, [settings, agents, samples]);
    
    const handlePlay = useCallback(async () => {
        if (!finalComposition) return;
        await audioServiceRef.current?.initialize();
        audioServiceRef.current?.play();
        setIsPlaying(true);
    }, [finalComposition]);

    const handleStop = useCallback(() => {
        audioServiceRef.current?.stop();
        setIsPlaying(false);
    }, []);
    
    const handleExport = useCallback(async () => {
        if (!finalComposition) return;
        await audioServiceRef.current?.exportWav();
    }, [finalComposition]);


    return (
        <div className="min-h-screen bg-gray-900 text-gray-200 p-4 sm:p-6 lg:p-8 flex flex-col">
            <header className="text-center mb-6 relative">
                <h1 className="text-4xl sm:text-5xl font-bold tracking-tight text-transparent bg-clip-text bg-gradient-to-r from-purple-400 to-pink-500">
                    Orchestrator AI Music Studio
                </h1>
                 <div className="absolute top-0 right-0">
                    <SettingsMenu />
                </div>
            </header>

            <main className="flex-grow flex flex-col gap-6">
                <ChainFlow agents={agents} />

                <div className="flex flex-col lg:flex-row gap-6 flex-grow">
                    <div className="lg:w-1/3 xl:w-1/4 flex flex-col gap-6">
                         <ControlPanel
                            settings={settings}
                            setSettings={setSettings}
                            onGenerate={handleGenerate}
                            isGenerating={isGenerating}
                            agents={agents}
                            setAgents={setAgents}
                            samples={samples}
                            setSamples={setSamples}
                        />
                        <AudioControls 
                            onPlay={handlePlay}
                            onStop={handleStop}
                            onExport={handleExport}
                            isPlaying={isPlaying}
                            isGenerating={isGenerating}
                            isCompositionReady={!!finalComposition}
                            audioService={audioServiceRef.current}
                        />
                    </div>
                    <div className="lg:w-2/3 xl:w-3/4">
                        <OutputDisplay agents={agents} />
                    </div>
                </div>
            </main>
        </div>
    );
};

export default App;
```

### `types.ts`

```ts
export type AgentName =
  | 'Theme Director'
  | 'Harmony Designer'
  | 'Rhythm Architect'
  | 'Melody Composer'
  | 'Audio Synthesizer';

export type AgentStatus = 'idle' | 'processing' | 'complete' | 'error';

export type AIModel = 'gemini-2.5-pro' | 'gemini-2.5-flash';

export interface Agent {
  name: AgentName;
  status: AgentStatus;
  output: any | null;
  model: AIModel;
}

export interface GenerationSettings {
  genre: string;
  key: string;
  tempo: number;
  mood: string;
  prompt: string;
}

export interface SampleMap {
  kick: string | null;
  snare: string | null;
  melody: string | null;
  pads: string | null;
  counterMelody: string | null;
}

// Types for musical composition
interface Note {
  time: string;
  note: string;
  duration: string;
  velocity: number;
}

interface KickPattern { time: string; }
interface SnarePattern { time: string; }
interface HihatPattern { time: string; }
interface PercussionPattern { time: string; }

export interface Theme {
  musicalTheme: string;
  lyricalConcepts: string[];
  songStructure: string[];
}

export interface Harmony {
  chordProgression: Note[];
  bassline: Note[];
}

export interface Rhythm {
  kick: KickPattern[];
  snare: SnarePattern[];
  hihat: HihatPattern[];
  percussion: PercussionPattern[];
}

export interface Melody {
  lead: Note[];
  counter: Note[];
  pads: Note[];
}

export interface MixParams {
    kick: { volume: number; attack: number; decay: number; };
    snare: { volume: number; attack: number; decay: number; noiseType: 'white' | 'pink' | 'brown'; };
    hihat: { volume: number; attack: number; decay: number; };
    melody: { volume: number; attack: number; decay: number; sustain: number; release: number; oscillator: 'sine' | 'square' | 'triangle' | 'sawtooth'; };
    chords: { volume: number; attack: number; decay: number; sustain: number; release: number; oscillator: 'sine' | 'square' | 'triangle' | 'sawtooth'; };
    bass: { volume: number; attack: number; decay: number; sustain: number; release: number; oscillator: 'sine' | 'square' | 'triangle' | 'sawtooth'; };
    reverb: { decay: number; wet: number; };
}

export interface FinalComposition {
    theme: Theme;
    harmony: Harmony;
    rhythm: Rhythm;
    melody: Melody;
    mixParams: MixParams;
    tempo: number;
}
```

### `constants.ts`

```ts
import { AgentName } from './types';

export const GENRES: string[] = [
  'Trap',
  'Trap-Soul',
  'R&B',
  'Soul',
  '90s Rap',
  'Lo-fi',
  'Drill',
  'Ambient',
  'Techno'
];

export const KEYS: string[] = [
  'A minor', 'A major',
  'A# minor', 'A# major',
  'B minor', 'B major',
  'C minor', 'C major',
  'C# minor', 'C# major',
  'D minor', 'D major',
  'D# minor', 'D# major',
  'E minor', 'E major',
  'F minor', 'F major',
  'F# minor', 'F# major',
  'G minor', 'G major',
  'G# minor', 'G# major'
];

export const AGENT_PIPELINE: AgentName[] = [
  'Theme Director',
  'Harmony Designer',
  'Rhythm Architect',
  'Melody Composer',
  'Audio Synthesizer',
];

export const AGENT_ICONS: Record<AgentName, string> = {
    'Theme Director': 'fa-solid fa-lightbulb',
    'Harmony Designer': 'fa-solid fa-music',
    'Rhythm Architect': 'fa-solid fa-drum',
    'Melody Composer': 'fa-solid fa-signature',
    'Audio Synthesizer': 'fa-solid fa-sliders'
};

export const AI_MODELS = ['gemini-2.5-pro', 'gemini-2.5-flash'];
```

### `components/ControlPanel.tsx`

```tsx
import React from 'react';
import { GenerationSettings, Agent, SampleMap } from '../types';
import { GENRES, KEYS } from '../constants';
import { AgentModelSelector } from './AgentModelSelector';

interface ControlPanelProps {
  settings: GenerationSettings;
  setSettings: React.Dispatch<React.SetStateAction<GenerationSettings>>;
  onGenerate: () => void;
  isGenerating: boolean;
  agents: Agent[];
  setAgents: React.Dispatch<React.SetStateAction<Agent[]>>;
  samples: SampleMap;
  setSamples: React.Dispatch<React.SetStateAction<SampleMap>>;
}

const fileToDataUrl = (file: File): Promise<string> => {
    return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.onload = () => resolve(reader.result as string);
        reader.onerror = reject;
        reader.readAsDataURL(file);
    });
};

export const ControlPanel: React.FC<ControlPanelProps> = ({ 
    settings, setSettings, onGenerate, isGenerating,
    agents, setAgents, samples, setSamples
}) => {
  const handleInputChange = <K extends keyof GenerationSettings>(key: K, value: GenerationSettings[K]) => {
    setSettings(prev => ({ ...prev, [key]: value }));
  };
  
  const handleFileChange = async (instrument: keyof SampleMap, file: File | null) => {
    if (file) {
        const url = await fileToDataUrl(file);
        setSamples(prev => ({...prev, [instrument]: url }));
    } else {
        setSamples(prev => ({...prev, [instrument]: null }));
    }
  };

  const inputStyle = "w-full bg-gray-700/50 border border-gray-600 rounded-md p-2 focus:ring-2 focus:ring-purple-500 focus:border-purple-500 outline-none transition duration-200";
  const labelStyle = "block text-sm font-medium text-gray-300 mb-1";
  const fileInputStyle = "text-xs text-gray-400 file:mr-2 file:py-1 file:px-2 file:rounded-md file:border-0 file:text-xs file:font-semibold file:bg-purple-500/20 file:text-purple-300 hover:file:bg-purple-500/40";


  return (
    <div className="bg-gray-800/50 backdrop-blur-md border border-gray-700 rounded-lg p-4 shadow-lg h-full flex flex-col">
      <h2 className="text-xl font-bold mb-4 text-transparent bg-clip-text bg-gradient-to-r from-purple-400 to-pink-500">Generation Settings</h2>
      <div className="space-y-4 flex-grow overflow-y-auto pr-2">
        <div>
          <label htmlFor="prompt" className={labelStyle}>Prompt</label>
          <textarea
            id="prompt"
            rows={2}
            className={inputStyle}
            value={settings.prompt}
            onChange={(e) => handleInputChange('prompt', e.target.value)}
          />
        </div>
        
        <div className="grid grid-cols-1 sm:grid-cols-2 gap-4">
            <div>
              <label htmlFor="genre" className={labelStyle}>Genre</label>
              <select id="genre" className={inputStyle} value={settings.genre} onChange={(e) => handleInputChange('genre', e.target.value)}>
                {GENRES.map(g => <option key={g} value={g}>{g}</option>)}
              </select>
            </div>
            <div>
              <label htmlFor="key" className={labelStyle}>Key</label>
              <select id="key" className={inputStyle} value={settings.key} onChange={(e) => handleInputChange('key', e.target.value)}>
                {KEYS.map(k => <option key={k} value={k}>{k}</option>)}
              </select>
            </div>
        </div>
        
        <div>
          <label htmlFor="mood" className={labelStyle}>Mood / Vibe</label>
          <input type="text" id="mood" className={inputStyle} value={settings.mood} onChange={(e) => handleInputChange('mood', e.target.value)} />
        </div>

        <div>
            <label htmlFor="tempo" className={`${labelStyle} flex justify-between`}><span>Tempo</span><span>{settings.tempo} BPM</span></label>
            <input type="range" id="tempo" min="60" max="180" step="1" className="w-full h-2 bg-gray-700 rounded-lg appearance-none cursor-pointer accent-pink-500" value={settings.tempo} onChange={(e) => handleInputChange('tempo', parseInt(e.target.value, 10))} />
        </div>
        
        <hr className="border-gray-600"/>

        <div>
            <h3 className="text-lg font-semibold mb-2 text-gray-300">Custom Samples</h3>
            <div className="space-y-2">
                <div>
                    <label htmlFor="kick-sample" className={labelStyle}>Kick</label>
                    <input type="file" id="kick-sample" accept=".wav,.mp3,.ogg" className={`${inputStyle} ${fileInputStyle}`} onChange={(e) => handleFileChange('kick', e.target.files?.[0] || null)} />
                </div>
                 <div>
                    <label htmlFor="snare-sample" className={labelStyle}>Snare</label>
                    <input type="file" id="snare-sample" accept=".wav,.mp3,.ogg" className={`${inputStyle} ${fileInputStyle}`} onChange={(e) => handleFileChange('snare', e.target.files?.[0] || null)} />
                </div>
                 <div>
                    <label htmlFor="melody-sample" className={labelStyle}>Lead Melody</label>
                    <input type="file" id="melody-sample" accept=".wav,.mp3,.ogg" className={`${inputStyle} ${fileInputStyle}`} onChange={(e) => handleFileChange('melody', e.target.files?.[0] || null)} />
                </div>
                 <div>
                    <label htmlFor="pads-sample" className={labelStyle}>Pads / Atmosphere</label>
                    <input type="file" id="pads-sample" accept=".wav,.mp3,.ogg" className={`${inputStyle} ${fileInputStyle}`} onChange={(e) => handleFileChange('pads', e.target.files?.[0] || null)} />
                </div>
                 <div>
                    <label htmlFor="counter-melody-sample" className={labelStyle}>Counter-Melody / SoundFont</label>
                    <input type="file" id="counter-melody-sample" accept=".wav,.mp3,.ogg,.sf2" className={`${inputStyle} ${fileInputStyle}`} onChange={(e) => handleFileChange('counterMelody', e.target.files?.[0] || null)} />
                     <p className="text-xs text-gray-500 mt-1">Note: .sf2 files are not directly playable, please use .wav, .mp3, or .ogg.</p>
                </div>
            </div>
        </div>
        
        <hr className="border-gray-600"/>
        
        <AgentModelSelector agents={agents} setAgents={setAgents} />

      </div>
      <button
        onClick={onGenerate}
        disabled={isGenerating}
        className="mt-6 w-full bg-gradient-to-r from-purple-500 to-pink-500 text-white font-bold py-3 px-4 rounded-md hover:from-purple-600 hover:to-pink-600 transition duration-300 disabled:opacity-50 disabled:cursor-not-allowed flex items-center justify-center gap-2"
      >
        {isGenerating ? (<><i className="fas fa-spinner fa-spin"></i>Generating...</>) : (<><i className="fas fa-magic"></i>Generate Music</>)}
      </button>
    </div>
  );
};
```

### `components/OutputDisplay.tsx`

```tsx
import React, { useState } from 'react';
import { Agent, AgentName, Rhythm, Melody } from '../types';
import { AGENT_ICONS } from '../constants';

interface OutputDisplayProps {
  agents: Agent[];
}

const OutputSummary: React.FC<{agentName: AgentName, output: any}> = ({ agentName, output }) => {
    if (agentName === 'Rhythm Architect') {
        const rhythm = output as Rhythm;
        const counts = [
            rhythm.kick?.length && `${rhythm.kick.length} kick hits`,
            rhythm.snare?.length && `${rhythm.snare.length} snare hits`,
            rhythm.hihat?.length && `${rhythm.hihat.length} hi-hat hits`,
            rhythm.percussion?.length && `${rhythm.percussion.length} percussion hits`,
        ].filter(Boolean).join(', ');
        return <div className="text-sm text-gray-400 italic">Generated rhythm: {counts}.</div>;
    }
    if (agentName === 'Melody Composer') {
         const melody = output as Melody;
         const counts = [
            melody.lead?.length && `${melody.lead.length} lead notes`,
            melody.counter?.length && `${melody.counter.length} counter-melody notes`,
            melody.pads?.length && `${melody.pads.length} pad notes`,
         ].filter(Boolean).join(', ');
        return <div className="text-sm text-gray-400 italic">Generated melody: {counts}.</div>;
    }
    return (
        <pre className="text-xs bg-black/30 p-2 rounded-md overflow-x-auto text-pink-300">
            <code>{JSON.stringify(output, null, 2)}</code>
        </pre>
    )
}

export const OutputDisplay: React.FC<OutputDisplayProps> = ({ agents }) => {
  const [collapsed, setCollapsed] = useState<Record<string, boolean>>({});

  const toggleCollapse = (name: AgentName) => {
    setCollapsed(prev => ({ ...prev, [name]: !prev[name] }));
  };

  return (
    <div className="bg-gray-800/50 backdrop-blur-md border border-gray-700 rounded-lg p-4 shadow-lg h-full overflow-y-auto">
      <h2 className="text-xl font-bold mb-4 text-transparent bg-clip-text bg-gradient-to-r from-purple-400 to-pink-500">Agent Outputs</h2>
      <div className="space-y-4">
        {agents.map((agent, index) => (
          <div key={index} className="bg-gray-900/50 border border-gray-700 rounded-lg p-4 transition-all duration-300">
            <button onClick={() => toggleCollapse(agent.name)} className="w-full text-left">
                <h3 className="text-lg font-semibold text-gray-300 mb-2 flex items-center justify-between">
                    <span className="flex items-center gap-2">
                        <i className={`${AGENT_ICONS[agent.name]}`}></i>
                        {agent.name}
                    </span>
                    <i className={`fas fa-chevron-down transition-transform duration-200 ${!collapsed[agent.name] ? 'rotate-180' : ''}`}></i>
                </h3>
            </button>
            
            {!collapsed[agent.name] && (
                <div className="mt-2">
                    {agent.status === 'processing' && (
                    <div className="flex items-center gap-2 text-blue-400">
                        <i className="fas fa-spinner fa-spin"></i>
                        <span>Processing...</span>
                    </div>
                    )}
                    {agent.status === 'complete' && agent.output && (
                        <OutputSummary agentName={agent.name} output={agent.output} />
                    )}
                    {agent.status === 'error' && (
                        <div className="text-red-400">
                            <i className="fas fa-exclamation-triangle mr-2"></i>
                            An error occurred.
                        </div>
                    )}
                     {agent.status === 'idle' && (
                        <div className="text-sm text-gray-500 italic">
                            Waiting to start...
                        </div>
                    )}
                </div>
            )}
          </div>
        ))}
      </div>
    </div>
  );
};
```

### `components/ChainFlow.tsx`

```tsx
import React from 'react';
import { Agent, AgentName } from '../types';
import { AGENT_ICONS } from '../constants';

interface ChainFlowProps {
  agents: Agent[];
}

const getStatusStyles = (status: Agent['status']): { container: string, icon: string, text: string } => {
    switch(status) {
        case 'processing':
            return { container: 'border-blue-500 bg-blue-500/20', icon: 'text-blue-400 animate-pulse', text: 'text-blue-300' };
        case 'complete':
            return { container: 'border-green-500 bg-green-500/20', icon: 'text-green-400', text: 'text-green-300' };
        case 'error':
            return { container: 'border-red-500 bg-red-500/20', icon: 'text-red-400', text: 'text-red-300' };
        default:
            return { container: 'border-gray-600 bg-gray-700/30', icon: 'text-gray-400', text: 'text-gray-400' };
    }
}

export const ChainFlow: React.FC<ChainFlowProps> = ({ agents }) => {
  return (
    <div className="bg-gray-800/50 backdrop-blur-md border border-gray-700 rounded-lg p-4 shadow-lg">
      <div className="flex items-center justify-around flex-wrap gap-4">
        {agents.map((agent, index) => {
            const styles = getStatusStyles(agent.status);
            return (
                <React.Fragment key={agent.name}>
                    <div className={`flex flex-col items-center p-3 rounded-lg border-2 transition-all duration-300 ${styles.container}`}>
                        <i className={`${AGENT_ICONS[agent.name]} text-2xl mb-1 ${styles.icon}`}></i>
                        <span className={`text-xs font-semibold ${styles.text}`}>{agent.name}</span>
                    </div>
                    {index < agents.length - 1 && (
                        <div className="hidden sm:block text-gray-500 text-2xl">
                           <i className="fas fa-arrow-right"></i>
                        </div>
                    )}
                </React.Fragment>
            );
        })}
      </div>
    </div>
  );
};
```

### `components/AudioControls.tsx`

```tsx
import React, { useRef, useEffect } from 'react';
import { AudioService } from '../services/audioService';

interface AudioControlsProps {
  onPlay: () => void;
  onStop: () => void;
  onExport: () => void;
  isPlaying: boolean;
  isGenerating: boolean;
  isCompositionReady: boolean;
  audioService: AudioService | null;
}

export const AudioControls: React.FC<AudioControlsProps> = ({
  onPlay,
  onStop,
  onExport,
  isPlaying,
  isGenerating,
  isCompositionReady,
  audioService,
}) => {
  const visualizerRef = useRef<HTMLCanvasElement>(null);

  useEffect(() => {
    if (visualizerRef.current && audioService) {
        audioService.setVisualizerCanvas(visualizerRef.current);
    }
  }, [audioService]);

  const buttonStyle = "flex-1 bg-gray-700/50 border border-gray-600 rounded-md py-3 px-4 text-gray-300 hover:bg-gray-600/50 hover:text-white transition duration-200 disabled:opacity-50 disabled:cursor-not-allowed flex items-center justify-center gap-2";

  return (
    <div className="bg-gray-800/50 backdrop-blur-md border border-gray-700 rounded-lg p-4 shadow-lg flex flex-col">
      <h2 className="text-xl font-bold mb-4 text-transparent bg-clip-text bg-gradient-to-r from-purple-400 to-pink-500">Playback & Visualizer</h2>
      <div className="bg-black/30 p-2 rounded-md mb-4 h-24">
        <canvas ref={visualizerRef} className="w-full h-full"></canvas>
      </div>
      <div className="flex gap-2">
        <button onClick={isPlaying ? onStop : onPlay} disabled={isGenerating || !isCompositionReady} className={buttonStyle}>
          {isPlaying ? (
              <>
                <i className="fas fa-stop"></i> Stop
              </>
          ) : (
              <>
                <i className="fas fa-play"></i> Play
              </>
          )}
        </button>
        <button onClick={onExport} disabled={isGenerating || !isCompositionReady} className={buttonStyle}>
            <i className="fas fa-download"></i> Export
        </button>
      </div>
    </div>
  );
};
```

### `components/AgentModelSelector.tsx`

```tsx
import React from 'react';
import { Agent, AIModel } from '../types';
import { AI_MODELS, AGENT_ICONS } from '../constants';

interface AgentModelSelectorProps {
  agents: Agent[];
  setAgents: React.Dispatch<React.SetStateAction<Agent[]>>;
}

export const AgentModelSelector: React.FC<AgentModelSelectorProps> = ({ agents, setAgents }) => {
  
  const handleModelChange = (agentName: string, model: AIModel) => {
    setAgents(prev => 
      prev.map(agent => 
        agent.name === agentName ? { ...agent, model } : agent
      )
    );
  };

  const labelStyle = "block text-sm font-medium text-gray-300 mb-1 flex items-center gap-2";
  const selectStyle = "w-full bg-gray-700/50 border border-gray-600 rounded-md p-2 focus:ring-2 focus:ring-purple-500 focus:border-purple-500 outline-none transition duration-200 text-sm";


  return (
    <div>
      <h3 className="text-lg font-semibold mb-2 text-gray-300">Agent AI Models</h3>
      <div className="space-y-3">
        {agents.map(agent => (
          <div key={agent.name}>
            <label htmlFor={`${agent.name}-model`} className={labelStyle}>
              <i className={AGENT_ICONS[agent.name]}></i>
              {agent.name}
            </label>
            <select
              id={`${agent.name}-model`}
              value={agent.model}
              onChange={(e) => handleModelChange(agent.name, e.target.value as AIModel)}
              className={selectStyle}
            >
              {AI_MODELS.map(model => (
                <option key={model} value={model}>
                  {model}
                </option>
              ))}
            </select>
          </div>
        ))}
      </div>
    </div>
  );
};
```

### `components/SettingsMenu.tsx`

```tsx
import React, { useState, useRef, useEffect } from 'react';
import { ALL_FILES } from '../services/readmeService';

const generateReadmeContent = () => {
    let content = `# Orchestrator AI Music Studio v4 - Build Instructions\n\n`;
    content += `This document provides complete instructions to rebuild this application from scratch using the files provided.\n\n`;
    content += `## Project Overview\n\n`;
    content += `Orchestrator AI Music Studio is an AI-powered music generation application that uses a chained orchestration of Large Language Model (LLM) agents to create musical compositions from user prompts. Users can customize genres, keys, tempos, and even provide their own audio samples for instruments. The application visualizes the entire creation process and allows for in-browser playback and WAV export of the final track.\n\n`;
    content += `## File Structure\n\n`;
    content += "\`\`\`\n"
    content += `/
|-- index.html
|-- index.tsx
|-- metadata.json
|-- App.tsx
|-- types.ts
|-- constants.ts
|-- components/
|   |-- ControlPanel.tsx
|   |-- OutputDisplay.tsx
|   |-- ChainFlow.tsx
|   |-- AudioControls.tsx
|   |-- AgentModelSelector.tsx
|   |-- SettingsMenu.tsx
|-- services/
    |-- geminiService.ts
    |-- audioService.ts
    |-- readmeService.ts
`
    content += "\n\`\`\`\n\n";
    content += `## File Contents\n\n`;

    ALL_FILES.forEach(file => {
        const lang = file.path.split('.').pop();
        content += `### \`${file.path}\`\n\n`;
        content += `\`\`\`${lang}\n`;
        content += file.content.trim();
        content += `\n\`\`\`\n\n`;
    });

    return content;
}


export const SettingsMenu: React.FC = () => {
  const [isOpen, setIsOpen] = useState(false);
  const menuRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    const handleClickOutside = (event: MouseEvent) => {
      if (menuRef.current && !menuRef.current.contains(event.target as Node)) {
        setIsOpen(false);
      }
    };
    document.addEventListener('mousedown', handleClickOutside);
    return () => document.removeEventListener('mousedown', handleClickOutside);
  }, []);

  const handleDownloadReadme = () => {
    const content = generateReadmeContent();
    const blob = new Blob([content], { type: 'text/markdown' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = 'README_Orchestrator_AI_v4.md';
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    URL.revokeObjectURL(url);
    setIsOpen(false);
  };

  return (
    <div className="relative" ref={menuRef}>
      <button
        onClick={() => setIsOpen(!isOpen)}
        className="text-gray-400 hover:text-white transition-colors duration-200 text-2xl p-2"
        aria-label="Settings"
      >
        <i className="fas fa-cog"></i>
      </button>

      {isOpen && (
        <div className="absolute top-full right-0 mt-2 w-64 bg-gray-800 border border-gray-700 rounded-lg shadow-xl z-10">
          <div className="p-2">
            <button
              onClick={handleDownloadReadme}
              className="w-full text-left px-4 py-2 text-sm text-gray-300 hover:bg-gray-700 rounded-md flex items-center gap-2"
            >
              <i className="fas fa-file-download"></i>
              <span>Download Build Instructions (README)</span>
            </button>
          </div>
        </div>
      )}
    </div>
  );
};
```

### `services/geminiService.ts`

```ts
import { GoogleGenAI, Type } from "@google/genai";
import { GenerationSettings, AgentName, FinalComposition, Theme, Harmony, Rhythm, Melody, MixParams, Agent, AIModel } from '../types';

const ai = new GoogleGenAI({ apiKey: process.env.API_KEY as string });

const noteSchema = {
    type: Type.OBJECT,
    properties: {
        time: { type: Type.STRING, description: 'The time in Tone.js Transport format (e.g., "0:0:0").' },
        note: { type: Type.STRING, description: 'The musical note (e.g., "C4", or an array of notes for a chord like ["C4", "E4", "G4"]).' },
        duration: { type: Type.STRING, description: 'The duration of the note (e.g., "4n", "8t").' },
        velocity: { type: Type.NUMBER, description: 'The velocity of the note (0-1).' },
    },
    required: ['time', 'note', 'duration', 'velocity']
};

const callAgent = async <T,>(prompt: string, schema: any, model: AIModel, onComplete: (output: T) => void): Promise<T> => {
    try {
        const response = await ai.models.generateContent({
            model: model,
            contents: prompt,
            config: {
                responseMimeType: 'application/json',
                responseSchema: schema,
            },
        });

        const jsonText = response.text;
        const result = JSON.parse(jsonText) as T;
        onComplete(result);
        return result;
    } catch (error) {
        console.error(`Error calling Gemini API with model ${model}:`, error);
        throw new Error(`Failed to get a valid response from the AI agent using ${model}.`);
    }
};

const getAgentModel = (agents: Agent[], name: AgentName): AIModel => {
    return agents.find(a => a.name === name)?.model || 'gemini-2.5-flash';
}

export const orchestrateMusicGeneration = async (
    settings: GenerationSettings,
    agents: Agent[],
    onAgentStart: (name: AgentName) => void,
    onAgentComplete: (name: AgentName, output: any) => void
): Promise<FinalComposition> => {
    const fullContext: any = { settings };
    
    // Agent 1: Theme Director
    onAgentStart('Theme Director');
    const themeModel = getAgentModel(agents, 'Theme Director');
    const themePrompt = `You are a musical theme director. Based on the user's request, generate a high-level musical theme, lyrical concepts, and a song structure for a song of about 3-4 minutes.
    Settings: ${JSON.stringify(settings, null, 2)}`;
    const themeSchema = {
        type: Type.OBJECT, properties: {
            musicalTheme: { type: Type.STRING },
            lyricalConcepts: { type: Type.ARRAY, items: { type: Type.STRING } },
            songStructure: { type: Type.ARRAY, items: { type: Type.STRING, description: "e.g., Intro, Verse, Chorus, Bridge, Outro" } }
        }, required: ['musicalTheme', 'lyricalConcepts', 'songStructure']
    };
    fullContext.theme = await callAgent<Theme>(themePrompt, themeSchema, themeModel, (output) => onAgentComplete('Theme Director', output));

    // Agent 2: Harmony Designer
    onAgentStart('Harmony Designer');
    const harmonyModel = getAgentModel(agents, 'Harmony Designer');
    const harmonyPrompt = `You are a harmony designer. Based on the theme, create a compelling 16-bar chord progression and a matching bassline in the key of ${settings.key}.
    Context: ${JSON.stringify(fullContext, null, 2)}`;
    const harmonySchema = {
        type: Type.OBJECT, properties: {
            chordProgression: { type: Type.ARRAY, items: noteSchema },
            bassline: { type: Type.ARRAY, items: noteSchema }
        }, required: ['chordProgression', 'bassline']
    };
    fullContext.harmony = await callAgent<Harmony>(harmonyPrompt, harmonySchema, harmonyModel, (output) => onAgentComplete('Harmony Designer', output));

    // Agent 3: Rhythm Architect
    onAgentStart('Rhythm Architect');
    const rhythmModel = getAgentModel(agents, 'Rhythm Architect');
    const rhythmPrompt = `You are a rhythm architect. Design a 16-bar drum pattern for the genre '${settings.genre}' at ${settings.tempo} BPM. Create patterns for kick, snare, hi-hats, and one percussion element.
    Context: ${JSON.stringify(fullContext, null, 2)}`;
    const drumPatternSchema = { type: Type.ARRAY, items: { type: Type.OBJECT, properties: { time: { type: Type.STRING } }, required: ['time'] } };
    const rhythmSchema = {
        type: Type.OBJECT, properties: {
            kick: drumPatternSchema,
            snare: drumPatternSchema,
            hihat: drumPatternSchema,
            percussion: drumPatternSchema,
        }, required: ['kick', 'snare', 'hihat']
    };
    fullContext.rhythm = await callAgent<Rhythm>(rhythmPrompt, rhythmSchema, rhythmModel, (output) => onAgentComplete('Rhythm Architect', output));

    // Agent 4: Melody Composer
    onAgentStart('Melody Composer');
    const melodyModel = getAgentModel(agents, 'Melody Composer');
    const melodyPrompt = `You are a melody composer. Create a 16-bar lead melody, a counter-melody, and atmospheric pads for the key of ${settings.key}. Use the provided harmony and rhythm as inspiration.
    Context: ${JSON.stringify(fullContext, null, 2)}`;
    const melodySchema = {
        type: Type.OBJECT, properties: {
            lead: { type: Type.ARRAY, items: noteSchema },
            counter: { type: Type.ARRAY, items: noteSchema },
            pads: { type: Type.ARRAY, items: noteSchema }
        }, required: ['lead', 'pads']
    };
    fullContext.melody = await callAgent<Melody>(melodyPrompt, melodySchema, melodyModel, (output) => onAgentComplete('Melody Composer', output));

    // Agent 5: Audio Synthesizer
    onAgentStart('Audio Synthesizer');
    const synthModel = getAgentModel(agents, 'Audio Synthesizer');
    const synthPrompt = `You are an audio engineer. Define mixing parameters (volume, ADSR) for each instrument and a global reverb setting. The user may provide their own samples, so focus on universal parameters. Make it sound good for the genre.
    Context: ${JSON.stringify(fullContext, null, 2)}`;
    const synthParamSchema = {
        type: Type.OBJECT, properties: {
            volume: { type: Type.NUMBER }, attack: { type: Type.NUMBER }, decay: { type: Type.NUMBER },
            sustain: { type: Type.NUMBER }, release: { type: Type.NUMBER },
            oscillator: { type: Type.STRING, description: 'Default if no sample provided. Can be sine, square, triangle, or sawtooth' }
        }, required: ['volume', 'attack', 'decay', 'sustain', 'release', 'oscillator']
    };
    const drumParamSchema = { type: Type.OBJECT, properties: { volume: { type: Type.NUMBER }, attack: { type: Type.NUMBER }, decay: { type: Type.NUMBER },}, required: ['volume', 'attack', 'decay']};
    const snareParamSchema = { ...drumParamSchema, properties: { ...drumParamSchema.properties, noiseType: { type: Type.STRING, description: "Can be white, pink, or brown"} }, required: [...drumParamSchema.required, 'noiseType']};

    const synthSchema = {
        type: Type.OBJECT, properties: {
            kick: drumParamSchema, snare: snareParamSchema, hihat: drumParamSchema,
            melody: synthParamSchema, chords: synthParamSchema, bass: synthParamSchema,
            reverb: { type: Type.OBJECT, properties: { decay: { type: Type.NUMBER }, wet: { type: Type.NUMBER } }, required: ['decay', 'wet'] }
        }, required: ['kick', 'snare', 'hihat', 'melody', 'chords', 'bass', 'reverb']
    };
    fullContext.mixParams = await callAgent<MixParams>(synthPrompt, synthSchema, synthModel, (output) => onAgentComplete('Audio Synthesizer', output));
    
    return {
        theme: fullContext.theme,
        harmony: fullContext.harmony,
        rhythm: fullContext.rhythm,
        melody: fullContext.melody,
        mixParams: fullContext.mixParams,
        tempo: settings.tempo
    };
};
```

### `services/audioService.ts`

```ts
import * as Tone from 'tone';
import { FinalComposition, SampleMap } from '../types';

export class AudioService {
    private isInitialized = false;
    private synths: any = {};
    private parts: Tone.Part[] = [];
    private composition: FinalComposition | null = null;
    private samples: SampleMap | null = null;
    private analyser: Tone.Analyser | null = null;
    private visualizerCanvas: HTMLCanvasElement | null = null;
    private animationFrameId: number | null = null;

    async initialize() {
        if (!this.isInitialized) {
            await Tone.start();
            this.isInitialized = true;
            console.log('AudioContext started');
        }
    }

    setVisualizerCanvas(canvas: HTMLCanvasElement) {
        this.visualizerCanvas = canvas;
        if (!this.analyser) {
            this.analyser = new Tone.Analyser('waveform', 1024);
            Tone.getDestination().connect(this.analyser);
        }
        this.drawVisualizer();
    }
    
    private drawVisualizer = () => {
        if (this.visualizerCanvas && this.analyser && Tone.Transport.state === 'started') {
            const canvas = this.visualizerCanvas;
            const ctx = canvas.getContext('2d');
            const values = this.analyser.getValue();

            if (ctx) {
                ctx.clearRect(0, 0, canvas.width, canvas.height);
                ctx.lineWidth = 2;
                const grad = ctx.createLinearGradient(0, 0, canvas.width, canvas.height);
                grad.addColorStop(0, '#a855f7');
                grad.addColorStop(1, '#ec4899');
                ctx.strokeStyle = grad;
                
                ctx.beginPath();
                let x = 0;
                const sliceWidth = canvas.width * 1.0 / values.length;

                for (let i = 0; i < values.length; i++) {
                    const v = (values[i] as number / 2) + 0.5;
                    const y = v * canvas.height;

                    if (i === 0) {
                        ctx.moveTo(x, y);
                    } else {
                        ctx.lineTo(x, y);
                    }
                    x += sliceWidth;
                }
                ctx.stroke();
            }
        } else if (this.visualizerCanvas) {
             const ctx = this.visualizerCanvas.getContext('2d');
             ctx?.clearRect(0, 0, this.visualizerCanvas.width, this.visualizerCanvas.height);
        }
        this.animationFrameId = requestAnimationFrame(this.drawVisualizer);
    }

    private async _createSynths(composition: FinalComposition, samples: SampleMap): Promise<any> {
        const newSynths: any = {};
        const reverb = new Tone.Reverb(composition.mixParams.reverb).toDestination();

        // Sampler/Synth setup helper
        const createInstrument = async (name: keyof SampleMap, fallback: () => any) => {
            if (samples[name]) {
                return new Promise((resolve, reject) => {
                    const sampler = new Tone.Sampler({
                        urls: { C3: samples[name]! },
                        onload: () => resolve(sampler.connect(reverb)),
                        onerror: (e) => reject(new Error(`Failed to load sample for ${name}: ${e}`)),
                    });
                });
            }
            return fallback();
        };
        
        const polySynthFallback = (params: any) => new Tone.PolySynth(Tone.Synth, {
             oscillator: { type: params.oscillator as any },
             envelope: params,
             volume: params.volume
        }).connect(reverb);

        newSynths.kick = await createInstrument('kick', () => new Tone.MembraneSynth(composition.mixParams.kick).connect(reverb));
        newSynths.snare = await createInstrument('snare', () => new Tone.NoiseSynth(composition.mixParams.snare).connect(reverb));
        newSynths.melody = await createInstrument('melody', () => polySynthFallback(composition.mixParams.melody));
        newSynths.pads = await createInstrument('pads', () => polySynthFallback(composition.mixParams.chords)); // Use chord settings for pads fallback
        newSynths.counterMelody = await createInstrument('counterMelody', () => polySynthFallback(composition.mixParams.melody)); // Use melody settings for counter-melody fallback

        // Default synths for other parts
        newSynths.hihat = new Tone.MetalSynth(composition.mixParams.hihat).connect(reverb);
        newSynths.percussion = new Tone.MembraneSynth().connect(reverb);
        newSynths.chords = polySynthFallback(composition.mixParams.chords);
        newSynths.bass = new Tone.MonoSynth({ oscillator: { type: composition.mixParams.bass.oscillator as any }, envelope: composition.mixParams.bass, volume: composition.mixParams.bass.volume }).connect(reverb);
        
        return newSynths;
    }

    private _scheduleParts(composition: FinalComposition, synths: any, transport: typeof Tone.Transport): Tone.Part[] {
        const scheduledParts: Tone.Part[] = [];
        
        transport.bpm.value = composition.tempo;

        const triggerNote = (synth: any, note: any, time: Tone.Unit.Time) => {
             if (!synth) { return; }
            if (synth instanceof Tone.Sampler || synth instanceof Tone.PolySynth || synth instanceof Tone.MonoSynth) {
                 synth.triggerAttackRelease(note.note, note.duration, time, note.velocity);
            } else {
                 // For one-shot drum synths
                 const duration = '8n';
                 const pitch = (synth instanceof Tone.MembraneSynth) ? 'C1' : undefined;
                 synth.triggerAttackRelease(pitch, duration, time);
            }
        };

        scheduledParts.push(new Tone.Part((time, note) => {
            try { triggerNote(synths.kick, note, time); } catch (e) { console.error('Error in kick part:', e); }
        }, composition.rhythm.kick.map(k => ({...k, note: 'C1', duration: '8n', velocity: 1.0 }))).start(0));

        scheduledParts.push(new Tone.Part((time, note) => {
            try { triggerNote(synths.snare, note, time); } catch (e) { console.error('Error in snare part:', e); }
        }, composition.rhythm.snare.map(s => ({...s, note: 'C1', duration: '8n', velocity: 1.0 }))).start(0));
        
        scheduledParts.push(new Tone.Part((time) => {
            try { synths.hihat.triggerAttackRelease('16n', time, 0.6); } catch (e) { console.error('Error in hihat part:', e); }
        }, composition.rhythm.hihat).start(0));

        if (composition.rhythm.percussion && composition.rhythm.percussion.length > 0) {
            scheduledParts.push(new Tone.Part((time) => {
                try { synths.percussion.triggerAttackRelease('G2', '16n', time); } catch (e) { console.error('Error in percussion part:', e); }
            }, composition.rhythm.percussion).start(0));
        }

        scheduledParts.push(new Tone.Part((time, note) => {
           if (note.note) try { triggerNote(synths.chords, note, time); } catch (e) { console.error('Error in chords part:', e); }
        }, composition.harmony.chordProgression).start(0));

        scheduledParts.push(new Tone.Part((time, note) => {
           if (note.note) try { triggerNote(synths.bass, note, time); } catch (e) { console.error('Error in bass part:', e); }
        }, composition.harmony.bassline).start(0));
        
        scheduledParts.push(new Tone.Part((time, note) => {
           if (note.note) try { triggerNote(synths.melody, note, time); } catch (e) { console.error('Error in melody part:', e); }
        }, composition.melody.lead).start(0));

        if (composition.melody.counter && composition.melody.counter.length > 0) {
            scheduledParts.push(new Tone.Part((time, note) => {
               if (note.note) try { triggerNote(synths.counterMelody, note, time); } catch (e) { console.error('Error in counter-melody part:', e); }
            }, composition.melody.counter).start(0));
        }

        scheduledParts.push(new Tone.Part((time, note) => {
           if (note.note) try { triggerNote(synths.pads, note, time); } catch (e) { console.error('Error in pads part:', e); }
        }, composition.melody.pads).start(0));

        transport.loop = true;
        transport.loopEnd = '16m';
        
        return scheduledParts;
    }

    async scheduleMusic(composition: FinalComposition, samples: SampleMap) {
        this.stop();
        this.composition = composition;
        this.samples = samples;

        // Clean up old resources
        this.parts.forEach(p => p.dispose());
        Object.values(this.synths).forEach((synth: any) => synth.dispose());
        Tone.Transport.cancel();

        // Create new resources
        this.synths = await this._createSynths(composition, samples);
        this.parts = this._scheduleParts(composition, this.synths, Tone.Transport);
    }

    play() {
        if (Tone.Transport.state !== 'started') {
            Tone.Transport.start();
        }
    }

    stop() {
        Tone.Transport.stop();
        Tone.Transport.position = 0;
    }
    
    async exportWav() {
        if (!this.composition || !this.samples) {
            console.error("No composition is available to export.");
            return;
        }
        await this.initialize();
        
        const composition = this.composition;
        const samples = this.samples;
        const duration = Tone.Time('16m').toSeconds();

        console.log("Starting offline render...");
        const buffer = await Tone.Offline(async (offlineContext) => {
            const { transport } = offlineContext;
            // All synths and samplers must be created within the offline context
            const offlineSynths = await this._createSynths(composition, samples);
            this._scheduleParts(composition, offlineSynths, transport);
            transport.start();
            console.log("Offline transport started.");
        }, duration);
        console.log("Offline render complete.");

        const blob = new Blob([this.bufferToWave(buffer)], { type: 'audio/wav' });
        const url = URL.createObjectURL(blob);
        const a = document.createElement('a');
        a.style.display = 'none';
        a.href = url;
        a.download = 'orchestrator-ai-music.wav';
        document.body.appendChild(a);
        a.click();
        window.URL.revokeObjectURL(url);
        document.body.removeChild(a);
    }
    
    private bufferToWave(abuffer: Tone.ToneAudioBuffer): ArrayBuffer {
        const numOfChan = abuffer.numberOfChannels;
        const length = abuffer.length * numOfChan * 2 + 44;
        const buffer = new ArrayBuffer(length);
        const view = new DataView(buffer);
        const channels = [];
        let i, sample;
        let offset = 0;
        let pos = 0;

        // RIFF header
        this.setUint32(view, pos, 0x46464952); pos += 4; // "RIFF"
        this.setUint32(view, pos, length - 8); pos += 4; // file length - 8
        this.setUint32(view, pos, 0x45564157); pos += 4; // "WAVE"

        // "fmt " chunk
        this.setUint32(view, pos, 0x20746d66); pos += 4; // "fmt " chunk
        this.setUint32(view, pos, 16); pos += 4; // length of format data
        this.setUint16(view, pos, 1); pos += 2; // format type 1 (PCM)
        this.setUint16(view, pos, numOfChan); pos += 2;
        this.setUint32(view, pos, abuffer.sampleRate); pos += 4;
        this.setUint32(view, pos, abuffer.sampleRate * 2 * numOfChan); pos += 4; // byte rate
        this.setUint16(view, pos, numOfChan * 2); pos += 2; // block align
        this.setUint16(view, pos, 16); pos += 2; // bits per sample

        // "data" chunk
        this.setUint32(view, pos, 0x61746164); pos += 4; // "data" chunk
        this.setUint32(view, pos, length - pos - 4); pos += 4; // data length

        for (i = 0; i < abuffer.numberOfChannels; i++) {
            channels.push(abuffer.getChannelData(i));
        }

        while (pos < length) {
            for (i = 0; i < numOfChan; i++) {
                sample = Math.max(-1, Math.min(1, channels[i][offset]));
                sample = (0.5 + sample < 0 ? sample * 32768 : sample * 32767) | 0;
                view.setInt16(pos, sample, true);
                pos += 2;
            }
            offset++;
        }
        return buffer;
    }

    private setUint16(view: DataView, offset: number, val: number) { view.setUint16(offset, val, true); }
    private setUint32(view: DataView, offset: number, val: number) { view.setUint32(offset, val, true); }

    dispose() {
        this.stop();
        if (this.animationFrameId) {
            cancelAnimationFrame(this.animationFrameId);
        }
        this.parts.forEach(p => p.dispose());
        Object.values(this.synths).forEach((synth: any) => synth.dispose());
        this.analyser?.dispose();
    }
}
```

### `services/readmeService.ts`

```ts
This file (readmeService.ts) is part of the build and contains the content for all other files to generate the README.md.
```

