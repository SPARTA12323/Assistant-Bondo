ai-qa-website/
├── backend/
│   ├── package.json
│   ├── index.js
├── frontend/
│   ├── package.json
│   ├── src/
│   │   ├── App.js
│   │   ├── index.js
│   │   └── components/
│   │       └── ChatBox.js
├── README.md
const express = require('express');
const cors = require('cors');
const axios = require('axios');

const app = express();
const port = 5000;

app.use(cors());
app.use(express.json());

app.post('/api/ask', async (req, res) => {
  const { question } = req.body;
  try {
    // Replace YOUR_OPENAI_API_KEY with your actual OpenAI API key
    const response = await axios.post(
      'https://api.openai.com/v1/chat/completions',
      {
        model: "gpt-3.5-turbo",
        messages: [{ role: 'user', content: question }],
        max_tokens: 150,
        temperature: 0.7,
      },
      {
        headers: {
          'Authorization': `Bearer YOUR_OPENAI_API_KEY`,
          'Content-Type': 'application/json',
        }
      }
    );
    res.json({ answer: response.data.choices[0].message.content });
  } catch (err) {
    res.status(500).json({ error: 'AI service error', details: err.message });
  }
});

app.listen(port, () => {
  console.log(`Server running on http://localhost:${port}`);
});
import React, { useState } from 'react';
import ChatBox from './components/ChatBox';

function App() {
  return (
    <div style={{ maxWidth: 600, margin: "30px auto", padding: 20, border: "1px solid #ddd", borderRadius: 8 }}>
      <h2>AI Q&A Website</h2>
      <ChatBox />
    </div>
  );
}

export default App;
import React, { useState } from 'react';

const API_URL = 'http://localhost:5000/api/ask';

function ChatBox() {
  const [question, setQuestion] = useState('');
  const [answer, setAnswer] = useState('');
  const [loading, setLoading] = useState(false);

  const askAI = async () => {
    setLoading(true);
    setAnswer('');
    try {
      const res = await fetch(API_URL, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ question }),
      });
      const data = await res.json();
      setAnswer(data.answer || data.error || 'No response');
    } catch (e) {
      setAnswer('Error connecting to AI service.');
    }
    setLoading(false);
  };

  return (
    <div>
      <textarea
        rows={3}
        value={question}
        onChange={e => setQuestion(e.target.value)}
        placeholder="Ask a question..."
        style={{ width: "100%", marginBottom: 10 }}
      />
      <button onClick={askAI} disabled={loading || !question.trim()}>
        {loading ? "Asking..." : "Ask AI"}
      </button>
      {answer && (
        <div style={{ marginTop: 20, padding: 10, background: "#f6f6f6", borderRadius: 4 }}>
          <b>AI:</b> {answer}
        </div>
      )}
    </div>
  );
}

export default ChatBox;
# AI Question Answering Website

A simple website where users can ask questions and get AI-powered answers.

## Features

- Ask any question, get an AI answer instantly.
- Powered by OpenAI API.
- Modern React frontend and Express backend.

## Getting Started

### Prerequisites

- Node.js (v18+)
- OpenAI API Key

### Setup

#### 1. Backend

```bash
cd backend
npm install
# Replace YOUR_OPENAI_API_KEY in index.js with your actual OpenAI key
node index.js
```

#### 2. Frontend

```bash
cd frontend
npm install
npm start
```

The frontend runs on [http://localhost:3000](http://localhost:3000) and the backend on [http://localhost:5000](http://localhost:5000).

## Customization

- You can easily style the app or expand with user authentication, chat history, etc.
