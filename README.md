# ROMA API Integration Guide

This guide will teach you how to integrate ROMA into your applications using simple API calls.

## Prerequisites

You need to have ROMA running on your local machine or a VPS server preferably. 

If you don't have ROMA running yet, make sure it's running and accessible at your chosen URL (typically <http://localhost:5000> for your local machine)

And <http://YOUR.VPS.IP:5000> for your VPS server, only if set it up UFW 

If you don't know how to use unconfirmed firewall (UFW) to get a public URL for your VPS server, read this guide: <a href="https://github.com/Legendandy/ROMA-VPS-Guide-How-to-Tunnel-ROMA-Localhost-Frontend-to-a-Public-URL/blob/main/README.md">Turn Your Localhost URL into a Public Url</a>  

If you already have your backend url, then let's begin

## Step 1: Quick Test with cURL

Before building any application, let's test if ROMA is working properly using a simple cURL command.

Open your terminal and run this command:

```
curl -X POST http://localhost:5000/api/simple/execute \
  -H "Content-Type: application/json" \
  -d '{
    "goal": "Explain photosynthesis process in plants",
    "options": {
      "max_iterations": 1,
      "timeout": 7200
    },
    "enable_hitl": false
  }'
```
If ROMA is working correctly, you should see a JSON response containing research results about photosynthesis. The response will be quite long and contain detailed information.

When ROMA processes your request, it returns a complex JSON object. The most important part is the final_output field, which contains the actual research results you want to use.

## Step 2: Setting Up Your Project

Create a new folder for a mock project and organize it like this:

```
your-project/
├── index.html
├── script.js
├── styles.css
└── README.md
```

## Step 3: Create the HTML Interface

First, let's create a simple web page where users can input their research queries. Create a file called index.html in your project folder:

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ROMA Research Interface</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <div class="container">
        <h1>ROMA Research Interface</h1>
        <div class="input-section">
            <textarea id="queryInput" placeholder="Enter your research topic here..."></textarea>
            <button id="researchBtn">Start Research</button>
        </div>
        <div class="results-section">
            <div id="loadingIndicator" class="hidden">
                <p>Researching your topic... This may take a few minutes.</p>
            </div>
            <div id="resultsContainer" class="hidden">
                <h2>Research Results</h2>
                <div id="resultsContent"></div>
                <button id="copyBtn">Copy Results</button>
            </div>
        </div>
    </div>
    <script src="script.js"></script>
</body>
</html>
```
## Step 4: Add Basic Styling

Create a file called styles.css to make your interface look presentable:

```
body {
    font-family: Arial, sans-serif;
    margin: 0;
    padding: 20px;
    background-color: #f5f5f5;
}

.container {
    max-width: 800px;
    margin: 0 auto;
    background-color: white;
    padding: 30px;
    border-radius: 10px;
    box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

h1 {
    color: #333;
    text-align: center;
    margin-bottom: 30px;
}

.input-section {
    margin-bottom: 30px;
}

#queryInput {
    width: 100%;
    height: 100px;
    padding: 15px;
    border: 1px solid #ddd;
    border-radius: 5px;
    font-size: 16px;
    resize: vertical;
    box-sizing: border-box;
}

#researchBtn {
    width: 100%;
    padding: 15px;
    background-color: #007bff;
    color: white;
    border: none;
    border-radius: 5px;
    font-size: 16px;
    cursor: pointer;
    margin-top: 10px;
}

#researchBtn:hover {
    background-color: #0056b3;
}

#researchBtn:disabled {
    background-color: #6c757d;
    cursor: not-allowed;
}

.hidden {
    display: none;
}

#loadingIndicator {
    text-align: center;
    padding: 20px;
    color: #007bff;
}

#resultsContainer {
    margin-top: 20px;
    padding: 20px;
    border: 1px solid #ddd;
    border-radius: 5px;
    background-color: #f9f9f9;
}

#resultsContent {
    white-space: pre-wrap;
    line-height: 1.6;
    margin-bottom: 20px;
    max-height: 500px;
    overflow-y: auto;
    padding: 15px;
    background-color: white;
    border-radius: 5px;
}

#copyBtn {
    padding: 10px 20px;
    background-color: #28a745;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
}

#copyBtn:hover {
    background-color: #218838;
}
```

## Step 5: Create the script.js

Now, let's create the main JavaScript file that will handle the API communication. Create a file called script.js:

```
const ROMA_API_URL = 'http://localhost:5000';

document.addEventListener('DOMContentLoaded', function() {
    const queryInput = document.getElementById('queryInput');
    const researchBtn = document.getElementById('researchBtn');
    const loadingIndicator = document.getElementById('loadingIndicator');
    const resultsContainer = document.getElementById('resultsContainer');
    const resultsContent = document.getElementById('resultsContent');
    const copyBtn = document.getElementById('copyBtn');

    researchBtn.addEventListener('click', handleResearch);
    copyBtn.addEventListener('click', copyResults);
});
```

## Step 6: Building the Research Function

Now, Let's break down the research functionality into smaller, understandable pieces. Add this to your script.js file:

```
async function handleResearch() {
    const query = document.getElementById('queryInput').value.trim();
    
    if (!query) {
        alert('Please enter a research topic!');
        return;
    }

    if (query.length < 3) {
        alert('Please provide a more detailed query (at least 3 characters)');
        return;
    }

    showLoadingState();
    
    try {
        const result = await performResearch(query);
        const extractedContent = extractFinalOutput(result);
        const formattedContent = formatStudyNotes(extractedContent);
        displayResults(formattedContent);
    } catch (error) {
        showError(error.message);
    }
}
```
## Step 7: Create the API Communication Function

This function handles the actual communication with ROMA. Add this to your script.js:

```
async function performResearch(query) {
    const requestBody = {
        goal: query,
        options: {
            max_iterations: 1,
            timeout: 7200
        },
        enable_hitl: false
    };

    const response = await fetch(`${ROMA_API_URL}/api/simple/execute`, {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
        },
        body: JSON.stringify(requestBody)
    });

    if (!response.ok) {
        throw new Error(`API request failed with status ${response.status}`);
    }

    return await response.json();
}
```

## Step 8: Extract the Research Results

ROMA's response contains a lot of information, but we mainly need the final_output. This function extracts it:

```
function extractFinalOutput(result) {
    if (result.final_output) {
        return result.final_output;
    }
    
    if (result.result) {
        if (typeof result.result === 'string') {
            try {
                const parsedResult = JSON.parse(result.result);
                return parsedResult.final_output || result.result;
            } catch {
                return result.result;
            }
        } else if (result.result.final_output) {
            return result.result.final_output;
        } else {
            return JSON.stringify(result.result, null, 2);
        }
    }
    
    if (result.content) {
        return result.content;
    }
    
    if (result.response) {
        return result.response;
    }
    
    return JSON.stringify(result, null, 2);
}
```

## Step 9: Format the Content for Display

This function cleans up and formats the research results for better readability:

```
function formatStudyNotes(content) {
    if (!content || typeof content !== 'string') {
        return 'No content available.';
    }

    let formatted = content
        .replace(/\\n/g, '\n')
        .replace(/\\"/g, '"')
        .replace(/\\\\/g, '\\')
        .replace(/\\t/g, '\t');

    formatted = formatted
        .replace(/\n{3,}/g, '\n\n')
        .replace(/[ \t]+/g, ' ')
        .trim();

    formatted = formatted
        .split('\n')
        .map(line => line.trim())
        .filter(line => line.length > 0)
        .join('\n\n');

    return formatted;
}
```

## Step 10: Create Helper Functions

Add these utility functions to manage the user interface:

```
function showLoadingState() {
    document.getElementById('researchBtn').disabled = true;
    document.getElementById('researchBtn').textContent = 'Researching...';
    document.getElementById('loadingIndicator').classList.remove('hidden');
    document.getElementById('resultsContainer').classList.add('hidden');
}

function displayResults(content) {
    document.getElementById('researchBtn').disabled = false;
    document.getElementById('researchBtn').textContent = 'Start Research';
    document.getElementById('loadingIndicator').classList.add('hidden');
    document.getElementById('resultsContainer').classList.remove('hidden');
    document.getElementById('resultsContent').textContent = content;
}

function showError(message) {
    document.getElementById('researchBtn').disabled = false;
    document.getElementById('researchBtn').textContent = 'Start Research';
    document.getElementById('loadingIndicator').classList.add('hidden');
    alert(`Research failed: ${message}`);
}

function copyResults() {
    const content = document.getElementById('resultsContent').textContent;
    navigator.clipboard.writeText(content).then(() => {
        const originalText = document.getElementById('copyBtn').textContent;
        document.getElementById('copyBtn').textContent = 'Copied!';
        setTimeout(() => {
            document.getElementById('copyBtn').textContent = originalText;
        }, 2000);
    });
}
```
## Step 11: Understanding the API Endpoint

When you send a request to ROMA's /api/simple/execute endpoint, you need to include these key fields:

- goal: This is your research topic or question
- options: Configuration settings for the research process
- enable_hitl: Whether to enable human-in-the-loop interaction (usually set to false for automated requests)

### The Response Structure

ROMA will return a complex JSON object with several possible fields:

- final_output: The main research results (this is what you usually want)
- execution_id: A unique identifier for the research request
- status: Whether the research was completed successfully
- timestamp: When the research was completed

# Advanced Integration Examples (Like my <a href="https://studybuddy.college">StudyBuddy</a> Website)

## Step 1: When using in Node.js Applications

If you want to use ROMA in a Node.js backend application, here's how you can create an API route:

```
const express = require('express');
const fetch = require('node-fetch');

const app = express();
app.use(express.json());

app.post('/research', async (req, res) => {
    const { query } = req.body;
    
    const romaResponse = await fetch('http://localhost:5000/api/simple/execute', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
        },
        body: JSON.stringify({
            goal: query,
            options: {
                max_iterations: 1,
                timeout: 7200
            },
            enable_hitl: false
        })
    });

    const result = await romaResponse.json();
    const finalOutput = extractFinalOutput(result);
    
    res.json({
        success: true,
        content: finalOutput
    });
});

function extractFinalOutput(result) {
    if (result.final_output) {
        return result.final_output;
    }
    return JSON.stringify(result, null, 2);
}

app.listen(3000, () => {
    console.log('Server running on port 3000');
});
```

## Step 2: When Using with React Applications

For React applications, you can create a custom hook to manage ROMA interactions:

```
import { useState } from 'react';

export const useRomaResearch = () => {
    const [isLoading, setIsLoading] = useState(false);
    const [results, setResults] = useState('');

    const performResearch = async (query) => {
        setIsLoading(true);
        
        try {
            const response = await fetch('http://localhost:5000/api/simple/execute', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify({
                    goal: query,
                    options: {
                        max_iterations: 1,
                        timeout: 7200
                    },
                    enable_hitl: false
                })
            });

            const result = await response.json();
            const finalOutput = extractFinalOutput(result);
            setResults(finalOutput);
        } finally {
            setIsLoading(false);
        }
    };

    return { performResearch, isLoading, results };
};

function extractFinalOutput(result) {
    if (result.final_output) {
        return result.final_output;
    }
    return JSON.stringify(result, null, 2);
}
```

# IMPORTANT CONFIGURATION NOTES: 

## API Endpoint URL

Make sure to update the ROMA_API_URL variable in your JavaScript code to match where your ROMA instance is running. If ROMA is running on a different port or server, change this accordingly:

```
const ROMA_API_URL = 'http://your-server:5000';
```


## IF YOU RUN INTO ANY PROBLEM PLEASE, DM ME ON X: @_hadeelen. THANKS. 
