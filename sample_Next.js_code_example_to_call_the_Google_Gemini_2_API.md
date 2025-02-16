Okay, let's craft a sample Next.js code example to call the Google Gemini 2 API with user inputs for `system_prompt`, `api_key`, `model`, and `temperature`.

This example will include:

1.  **A Next.js Frontend (React Component):**  To take user inputs for the API parameters and the main prompt, and display the Gemini API response.
2.  **A Next.js API Route (Backend):** To securely handle the API call to Google Gemini, keeping the API key server-side.

**Project Setup (if you haven't already):**

```bash
npx create-next-app gemini-app
cd gemini-app
npm install @google/generative-ai
```

**Code Structure:**

We will have the following files:

*   `pages/index.js`:  The main page with the React component for user interaction.
*   `pages/api/gemini.js`: The Next.js API route to handle the Gemini API call.

**1. `pages/index.js` (Frontend Component)**

```javascript
import React, { useState } from 'react';

export default function Home() {
    const [apiKey, setApiKey] = useState('');
    const [systemPrompt, setSystemPrompt] = useState('');
    const [model, setModel] = useState('gemini-pro'); // Default model
    const [temperature, setTemperature] = useState(0.7); // Default temperature
    const [prompt, setPrompt] = useState('');
    const [response, setResponse] = useState('');
    const [isLoading, setIsLoading] = useState(false);
    const [error, setError] = useState('');

    const handleGeminiCall = async () => {
        setIsLoading(true);
        setError('');
        setResponse(''); // Clear previous response

        try {
            const res = await fetch('/api/gemini', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify({
                    apiKey,
                    systemPrompt,
                    model,
                    temperature: parseFloat(temperature), // Ensure temperature is a number
                    prompt,
                }),
            });

            if (!res.ok) {
                const errorData = await res.json();
                throw new Error(errorData.error || `HTTP error! status: ${res.status}`);
            }

            const data = await res.json();
            setResponse(data.response);

        } catch (e) {
            setError(e.message);
            console.error("Gemini API Error:", e);
        } finally {
            setIsLoading(false);
        }
    };

    return (
        <div className="container mx-auto p-4">
            <h1 className="text-2xl font-bold mb-4">Google Gemini 2 API Demo</h1>

            {/* API Key Input */}
            <div className="mb-4">
                <label htmlFor="apiKey" className="block text-gray-700 text-sm font-bold mb-2">API Key:</label>
                <input
                    type="password" // Use password type for API key input
                    id="apiKey"
                    className="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
                    placeholder="Enter your Google Gemini API Key"
                    value={apiKey}
                    onChange={(e) => setApiKey(e.target.value)}
                />
                <p className="text-xs text-gray-500 mt-1">Get your API key from Google Cloud AI Studio.</p>
            </div>

            {/* System Prompt Input */}
            <div className="mb-4">
                <label htmlFor="systemPrompt" className="block text-gray-700 text-sm font-bold mb-2">System Prompt (Optional):</label>
                <textarea
                    id="systemPrompt"
                    className="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
                    placeholder="e.g., You are a helpful assistant."
                    value={systemPrompt}
                    onChange={(e) => setSystemPrompt(e.target.value)}
                />
                <p className="text-xs text-gray-500 mt-1">Sets the context for the model's behavior.</p>
            </div>

            {/* Model Select */}
            <div className="mb-4">
                <label htmlFor="model" className="block text-gray-700 text-sm font-bold mb-2">Model:</label>
                <select
                    id="model"
                    className="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
                    value={model}
                    onChange={(e) => setModel(e.target.value)}
                >
                    <option value="gemini-pro">gemini-pro</option>
                    {/* Add other models as needed, e.g., gemini-pro-vision */}
                </select>
                <p className="text-xs text-gray-500 mt-1">Choose the Gemini model to use.</p>
            </div>

            {/* Temperature Slider */}
            <div className="mb-4">
                <label htmlFor="temperature" className="block text-gray-700 text-sm font-bold mb-2">Temperature:</label>
                <input
                    type="range"
                    id="temperature"
                    min="0"
                    max="1"
                    step="0.1"
                    className="w-full"
                    value={temperature}
                    onChange={(e) => setTemperature(e.target.value)}
                />
                <input
                    type="number"
                    id="temperature-value"
                    min="0"
                    max="1"
                    step="0.1"
                    className="shadow appearance-none border rounded w-16 py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline ml-2"
                    value={temperature}
                    onChange={(e) => setTemperature(e.target.value)} // Keep numeric input in sync with slider
                />
                <p className="text-xs text-gray-500 mt-1">Controls randomness: 0 for deterministic, 1 for max randomness.</p>
            </div>


            {/* Prompt Input */}
            <div className="mb-4">
                <label htmlFor="prompt" className="block text-gray-700 text-sm font-bold mb-2">Prompt:</label>
                <textarea
                    id="prompt"
                    className="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
                    placeholder="Enter your prompt here..."
                    rows="4"
                    value={prompt}
                    onChange={(e) => setPrompt(e.target.value)}
                />
            </div>

            {/* Call Gemini Button */}
            <div className="mb-4">
                <button
                    className="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline"
                    type="button"
                    onClick={handleGeminiCall}
                    disabled={isLoading || !apiKey}
                >
                    {isLoading ? 'Loading...' : 'Generate Response'}
                </button>
                {!apiKey && <p className="text-red-500 text-sm mt-1">API Key is required.</p>}
            </div>

            {/* Response Display */}
            {error && <div className="text-red-500 mb-4">Error: {error}</div>}
            {response && (
                <div className="border p-4 rounded">
                    <h2 className="font-bold mb-2">Gemini Response:</h2>
                    <pre className="whitespace-pre-wrap">{response}</pre>
                </div>
            )}
        </div>
    );
}
```

**Explanation of `pages/index.js`:**

*   **State Variables:**  Uses `useState` hooks to manage user inputs (`apiKey`, `systemPrompt`, `model`, `temperature`, `prompt`), the API response (`response`), loading state (`isLoading`), and error messages (`error`).
*   **Input Fields:**  Provides input fields for each parameter:
    *   `apiKey`:  A password input type for API key security (on the client side - remember the *real* security is on the backend).
    *   `systemPrompt`: A textarea for the system prompt.
    *   `model`: A dropdown select for choosing the Gemini model.
    *   `temperature`: A range slider and a number input to control the temperature.
    *   `prompt`: A textarea for the main user prompt.
*   **`handleGeminiCall` Function:**
    *   This `async` function is triggered when the "Generate Response" button is clicked.
    *   Sets `isLoading` to `true` and clears any previous errors and responses.
    *   Uses `fetch` to make a `POST` request to the `/api/gemini` API route (which we'll create next).
    *   Sends the user inputs in the request body as JSON.
    *   Handles successful responses and errors.
    *   Updates the `response` state with the Gemini API's response.
    *   Sets `isLoading` back to `false` in the `finally` block.
*   **UI Elements:**
    *   Labels and input fields are used for each parameter.
    *   A button to trigger the API call, disabled when loading or if `apiKey` is missing.
    *   Error message display.
    *   Response display in a `pre` tag for preserving formatting.

**2. `pages/api/gemini.js` (Backend API Route)**

```javascript
import { GoogleGenerativeAI } from "@google/generative-ai";

export default async function handler(req, res) {
    if (req.method !== 'POST') {
        return res.status(405).json({ error: 'Method Not Allowed' });
    }

    const { apiKey, systemPrompt, model, temperature, prompt } = req.body;

    if (!apiKey) {
        return res.status(400).json({ error: 'API Key is required' });
    }
    if (!prompt) {
        return res.status(400).json({ error: 'Prompt is required' });
    }

    try {
        const genAI = new GoogleGenerativeAI(apiKey);
        const generativeModel = genAI.getModel(model);

        const generationConfig = {
            temperature: temperature,
            topP: 0.95,
            topK: 40,
        };

        const safetySettings = [
            {
                category: 'HARM_CATEGORY_HARASSMENT',
                threshold: 'BLOCK_MEDIUM_AND_ABOVE',
            },
            {
                category: 'HARM_CATEGORY_HATE_SPEECH',
                threshold: 'BLOCK_MEDIUM_AND_ABOVE',
            },
            {
                category: 'HARM_CATEGORY_SEXUALLY_EXPLICIT',
                threshold: 'BLOCK_MEDIUM_AND_ABOVE',
            },
            {
                category: 'HARM_CATEGORY_DANGEROUS_CONTENT',
                threshold: 'BLOCK_MEDIUM_AND_ABOVE',
            },
        ];

        const parts = [];
        if (systemPrompt) {
            parts.push({ text: systemPrompt + "\n\n" + prompt }); // Combine system and user prompt
        } else {
            parts.push({ text: prompt });
        }


        const result = await generativeModel.generateContent({
            generationConfig,
            safetySettings,
            contents: [{ role: 'user', parts }],
        });

        const geminiResponse = result.response.text();
        return res.status(200).json({ response: geminiResponse });

    } catch (error) {
        console.error("Error calling Gemini API:", error);
        return res.status(500).json({ error: 'Failed to generate response', details: error.message });
    }
}
```

**Explanation of `pages/api/gemini.js`:**

*   **API Route Handler:**  This is a Next.js API route function (`handler`).
*   **Method Check:**  It checks if the request method is `POST`. If not, it returns a 405 "Method Not Allowed" error.
*   **Input Extraction:**  It extracts `apiKey`, `systemPrompt`, `model`, `temperature`, and `prompt` from the request body (`req.body`).
*   **Input Validation:**  It checks if `apiKey` and `prompt` are provided. If not, it returns a 400 "Bad Request" error.
*   **Gemini API Call:**
    *   **Initialize Gemini API:**  `const genAI = new GoogleGenerativeAI(apiKey);`  This initializes the Google Generative AI SDK with the API key received from the frontend. **Important:** The API key is used *only* on the server-side here, which is crucial for security.
    *   **Get Model:** `const generativeModel = genAI.getModel(model);` Gets the specified Gemini model.
    *   **Configure Generation:**  `generationConfig` and `safetySettings` are set. You can customize these further as needed.
    *   **Prepare Content:**  The `parts` array constructs the content to send to the Gemini API. It combines the `systemPrompt` (if provided) and the user `prompt` into a single text part.
    *   **`generateContent` Call:**  `generativeModel.generateContent(...)` makes the actual API call to Google Gemini.
    *   **Extract Response:**  `const geminiResponse = result.response.text();` Extracts the text response from the Gemini API result.
*   **Successful Response:**  Returns a 200 status code with the `geminiResponse` in the JSON body.
*   **Error Handling:**  Uses a `try...catch` block to handle errors during the API call. If an error occurs:
    *   Logs the error to the server console.
    *   Returns a 500 status code with an error message in the JSON body.

**How to Run the Code:**

1.  **Install Dependencies:**  Make sure you've run `npm install @google/generative-ai` in your project directory.
2.  **Start the Development Server:**  Run `npm run dev` in your terminal.
3.  **Open in Browser:**  Go to `http://localhost:3000` in your web browser.
4.  **Enter API Key:**  Paste your Google Gemini API key into the "API Key" field.
5.  **Enter Parameters and Prompt:**  Fill in the System Prompt (optional), Model, Temperature, and your Prompt.
6.  **Click "Generate Response":**  Click the button to send the request to the Gemini API.
7.  **See the Response:**  The Gemini API's response will be displayed below.

**Important Security Notes:**

*   **API Key Security:**  **Never** hardcode your API key into the frontend code or commit it to your repository. This example takes the API key as user input and sends it to the backend. The API key is used *only* in the `pages/api/gemini.js` (backend) file, which is the secure and correct way to handle API keys in a frontend application.
*   **Rate Limits and Costs:** Be aware of Google Gemini API's rate limits and usage costs. Monitor your API usage in Google Cloud AI Studio.
*   **Error Handling:** The code includes basic error handling, but you might want to enhance it for production applications (e.g., more specific error messages, logging, retries).
*   **Input Sanitization:** For production, you should sanitize user inputs on both the frontend and backend to prevent potential security vulnerabilities.

This sample provides a functional starting point for building a Next.js application that interacts with the Google Gemini API using user-configurable parameters. You can expand upon this to create more complex and feature-rich applications.
