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
import React from 'react';
import ChatBox from './components/ChatBox';
import './App.css';

function App() {
  return (
    <div className="app-bg">
      <header className="header">
        <img src="https://cdn-icons-png.flaticon.com/512/4712/4712035.png" alt="AI Icon" className="logo"/>
        <h2>AI Q&A Chat</h2>
      </header>
      <main className="main-container">
        <ChatBox />
      </main>
      <footer className="footer">
        &copy; {new Date().getFullYear()} SPARTA12323 &mdash; Powered by OpenAI
      </footer>
    </div>
  );
}

export default App;
import React, { useState } from 'react';
import './ChatBox.css';

const API_URL = 'http://localhost:5000/api/ask';

function ChatBox() {
  const [question, setQuestion] = useState('');
  const [messages, setMessages] = useState([]);
  const [loading, setLoading] = useState(false);

  const askAI = async () => {
    if (!question.trim()) return;
    const userMsg = { from: 'user', text: question };
    setMessages([...messages, userMsg]);
    setLoading(true);
    setQuestion('');
    try {
      const res = await fetch(API_URL, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ question: userMsg.text }),
      });
      const data = await res.json();
      setMessages(msgs => [
        ...msgs,
        { from: 'ai', text: data.answer || data.error || 'No response' }
      ]);
    } catch (e) {
      setMessages(msgs => [
        ...msgs,
        { from: 'ai', text: 'Error connecting to AI service.' }
      ]);
    }
    setLoading(false);
  };

  const handleKeyDown = e => {
    if (e.key === 'Enter' && !e.shiftKey) {
      e.preventDefault();
      askAI();
    }
  };

  return (
    <div className="chatbox">
      <div className="messages">
        {messages.map((m, i) => (
          <div
            key={i}
            className={`bubble ${m.from === 'user' ? 'user' : 'ai'}`}
          >
            {m.text}
          </div>
        ))}
        {loading && (
          <div className="bubble ai">
            <span className="loader"></span>
            <span style={{marginLeft: 8}}>AI is thinking...</span>
          </div>
        )}
      </div>
      <form className="input-area" onSubmit={e => { e.preventDefault(); askAI(); }}>
        <textarea
          rows={2}
          value={question}
          onChange={e => setQuestion(e.target.value)}
          onKeyDown={handleKeyDown}
          placeholder="Ask a question..."
          disabled={loading}
        />
        <button type="submit" disabled={loading || !question.trim()}>
          Send
        </button>
      </form>
    </div>
  );
}

export default ChatBox;
@import url('https://fonts.googleapis.com/css?family=Inter:400,700&display=swap');

.app-bg {
  min-height: 100vh;
  background: linear-gradient(135deg, #f6f7fb 0%, #e9effd 100%);
  display: flex;
  flex-direction: column;
  font-family: 'Inter', Arial, sans-serif;
}

.header {
  display: flex;
  align-items: center;
  gap: 14px;
  background: #3157d5;
  color: #fff;
  padding: 22px 0 18px 0;
  justify-content: center;
  box-shadow: 0 2px 8px 0 rgba(49,87,213,0.07);
  position: sticky;
  top: 0;
  z-index: 1;
}

.logo {
  width: 38px;
  height: 38px;
}

.main-container {
  flex: 1;
  display: flex;
  justify-content: center;
  align-items: flex-start;
  padding: 28px 8px 18px 8px;
}

.footer {
  text-align: center;
  padding: 12px 0 8px 0;
  color: #6c7a9c;
  background: none;
  font-size: 15px;
  letter-spacing: 0.02em;
}
.chatbox {
  background: #fff;
  border-radius: 14px;
  box-shadow: 0 6px 32px 0 rgba(49,87,213,0.09);
  width: 100%;
  max-width: 510px;
  padding: 0 0 10px 0;
  display: flex;
  flex-direction: column;
}

.messages {
  flex: 1;
  min-height: 320px;
  max-height: 400px;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  gap: 10px;
  padding: 22px 18px 12px 18px;
}

.bubble {
  display: inline-block;
  max-width: 80%;
  padding: 12px 16px;
  border-radius: 16px;
  font-size: 16px;
  line-height: 1.6;
  margin-bottom: 2px;
  animation: fadeIn 0.4s;
  word-break: break-word;
}

.user {
  align-self: flex-end;
  background: linear-gradient(135deg, #3157d5 85%, #5a8fff 100%);
  color: #fff;
  border-bottom-right-radius: 4px;
}

.ai {
  align-self: flex-start;
  background: #f2f4fa;
  color: #2e3756;
  border-bottom-left-radius: 4px;
}

.input-area {
  display: flex;
  gap: 8px;
  align-items: flex-end;
  border-top: 1px solid #f0f2fa;
  padding: 14px 18px 0 18px;
  background: none;
}

.input-area textarea {
  flex: 1;
  resize: none;
  border: 1.5px solid #d4def8;
  border-radius: 8px;
  font-size: 16px;
  padding: 10px;
  font-family: inherit;
  margin-bottom: 0;
  outline: none;
  transition: border-color 0.18s;
  min-height: 40px;
  background: #f7faff;
}

.input-area textarea:focus {
  border-color: #3157d5;
}

.input-area button {
  background: #3157d5;
  color: #fff;
  border: none;
  border-radius: 8px;
  padding: 10px 22px;
  font-size: 16px;
  font-weight: 700;
  cursor: pointer;
  transition: background 0.16s;
}

.input-area button:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(14px); }
  to { opacity: 1; transform: translateY(0); }
}

/* Loader Spinner */
.loader,
.loader:before,
.loader:after {
  border-radius: 50%;
}
.loader {
  color: #3157d5;
  font-size: 10px;
  text-indent: -9999em;
  margin: 0 6px 0 0;
  position: relative;
  width: 18px;
  height: 18px;
  box-shadow: inset 0 0 0 2px;
  transform: translateZ(0);
  animation: load6 1.1s infinite linear;
  display: inline-block;
}
@keyframes load6 {
  0% { transform: rotate(0deg);}
  100% { transform: rotate(360deg);}
}
.chatbox {
  background: #1b1b20;
  border-radius: 14px;
  box-shadow: 0 6px 32px 0 rgba(0,0,0,0.18);
  width: 100%;
  max-width: 510px;
  padding: 0 0 10px 0;
  display: flex;
  flex-direction: column;
}

.messages {
  flex: 1;
  min-height: 320px;
  max-height: 400px;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  gap: 10px;
  padding: 22px 18px 12px 18px;
}

.bubble {
  display: inline-block;
  max-width: 80%;
  padding: 12px 16px;
  border-radius: 16px;
  font-size: 16px;
  line-height: 1.6;
  margin-bottom: 2px;
  animation: fadeIn 0.4s;
  word-break: break-word;
}

.user {
  align-self: flex-end;
  background: linear-gradient(135deg, #23233b 85%, #2d2d4a 100%);
  color: #fff;
  border-bottom-right-radius: 4px;
}

.ai {
  align-self: flex-start;
  background: #21222d;
  color: #e4e6ee;
  border-bottom-left-radius: 4px;
}

.input-area {
  display: flex;
  gap: 8px;
  align-items: flex-end;
  border-top: 1px solid #23233b;
  padding: 14px 18px 0 18px;
  background: none;
}

.input-area textarea {
  flex: 1;
  resize: none;
  border: 1.5px solid #23233b;
  border-radius: 8px;
  font-size: 16px;
  padding: 10px;
  font-family: inherit;
  margin-bottom: 0;
  outline: none;
  transition: border-color 0.18s;
  min-height: 40px;
  background: #191920;
  color: #fff;
}

.input-area textarea:focus {
  border-color: #5a8fff;
}

.input-area button {
  background: #23233b;
  color: #fff;
  border: none;
  border-radius: 8px;
  padding: 10px 22px;
  font-size: 16px;
  font-weight: 700;
  cursor: pointer;
  transition: background 0.16s;
}

.input-area button:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(14px); }
  to { opacity: 1; transform: translateY(0); }
}

/* Loader Spinner */
.loader,
.loader:before,
.loader:after {
  border-radius: 50%;
}
.loader {
  color: #5a8fff;
  font-size: 10px;
  text-indent: -9999em;
  margin: 0 6px 0 0;
  position: relative;
  width: 18px;
  height: 18px;
  box-shadow: inset 0 0 0 2px;
  transform: translateZ(0);
  animation: load6 1.1s infinite linear;
  display: inline-block;
}
@keyframes load6 {
  0% { transform: rotate(0deg);}
  100% { transform: rotate(360deg);}
}
.chatbox {
  background: #1b1b20;
  border-radius: 14px;
  box-shadow: 0 6px 32px 0 rgba(0,0,0,0.18);
  width: 100%;
  max-width: 510px;
  padding: 0 0 10px 0;
  display: flex;
  flex-direction: column;
}

.messages {
  flex: 1;
  min-height: 320px;
  max-height: 400px;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  gap: 10px;
  padding: 22px 18px 12px 18px;
}

.bubble {
  display: inline-block;
  max-width: 80%;
  padding: 12px 16px;
  border-radius: 16px;
  font-size: 16px;
  line-height: 1.6;
  margin-bottom: 2px;
  animation: fadeIn 0.4s;
  word-break: break-word;
}

.user {
  align-self: flex-end;
  background: linear-gradient(135deg, #23233b 85%, #2d2d4a 100%);
  color: #fff;
  border-bottom-right-radius: 4px;
}

.ai {
  align-self: flex-start;
  background: #21222d;
  color: #e4e6ee;
  border-bottom-left-radius: 4px;
}

.input-area {
  display: flex;
  gap: 8px;
  align-items: flex-end;
  border-top: 1px solid #23233b;
  padding: 14px 18px 0 18px;
  background: none;
}

.input-area textarea {
  flex: 1;
  resize: none;
  border: 1.5px solid #23233b;
  border-radius: 8px;
  font-size: 16px;
  padding: 10px;
  font-family: inherit;
  margin-bottom: 0;
  outline: none;
  transition: border-color 0.18s;
  min-height: 40px;
  background: #191920;
  color: #fff;
}

.input-area textarea:focus {
  border-color: #5a8fff;
}

.input-area button {
  background: #23233b;
  color: #fff;
  border: none;
  border-radius: 8px;
  padding: 10px 22px;
  font-size: 16px;
  font-weight: 700;
  cursor: pointer;
  transition: background 0.16s;
}

.input-area button:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(14px); }
  to { opacity: 1; transform: translateY(0); }
}

/* Loader Spinner */
.loader,
.loader:before,
.loader:after {
  border-radius: 50%;
}
.loader {
  color: #5a8fff;
  font-size: 10px;
  text-indent: -9999em;
  margin: 0 6px 0 0;
  position: relative;
  width: 18px;
  height: 18px;
  box-shadow: inset 0 0 0 2px;
  transform: translateZ(0);
  animation: load6 1.1s infinite linear;
  display: inline-block;
}
@keyframes load6 {
  0% { transform: rotate(0deg);}
  100% { transform: rotate(360deg);}
}
@import url('https://fonts.googleapis.com/css?family=Inter:400,700&display=swap');

.app-bg {
  min-height: 100vh;
  background: linear-gradient(135deg, #18181c 0%, #23232b 100%);
  display: flex;
  flex-direction: column;
  font-family: 'Inter', Arial, sans-serif;
}

.header {
  display: flex;
  align-items: center;
  gap: 14px;
  
  background: #101015;
  color: #fff;
  padding: 22px 0 18px 0;
  justify-content: center;
  box-shadow: 0 2px 8px 0 rgba(0,0,0,0.18);
  position: sticky;
  top: 0;
  z-index: 1;
}

.logo {
  width: 38px;
  height: 38px;
}

.main-container {
  flex: 1;
  display: flex;
  justify-content: center;
  align-items: flex-start;
  padding: 28px 8px 18px 8px;
}

.footer {
  text-align: center;
  padding: 12px 0 8px 0;
  color: #bbb;
  background: none;
  font-size: 15px;
  letter-spacing: 0.02em;
}
.chatbox {
  background: #1b1b20;
  border-radius: 14px;
  box-shadow: 0 6px 32px 0 rgba(0,0,0,0.18);
  width: 100%;
  max-width: 510px;
  padding: 0 0 10px 0;
  display: flex;
  flex-direction: column;
}

.messages {
  flex: 1;
  min-height: 320px;
  max-height: 400px;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  gap: 10px;
  padding: 22px 18px 12px 18px;
}

.bubble {
  display: inline-block;
  max-width: 80%;
  padding: 12px 16px;
  border-radius: 16px;
  font-size: 16px;
  line-height: 1.6;
  margin-bottom: 2px;
  animation: fadeIn 0.4s;
  word-break: break-word;
}

.user {
  align-self: flex-end;
  background: linear-gradient(135deg, #23233b 85%, #2d2d4a 100%);
  color: #fff;
  border-bottom-right-radius: 4px;
}

.ai {
  align-self: flex-start;
  background: #21222d;
  color: #e4e6ee;
  border-bottom-left-radius: 4px;
}

.input-area {
  display: flex;
  gap: 8px;
  align-items: flex-end;
  border-top: 1px solid #23233b;
  padding: 14px 18px 0 18px;
  background: none;
}

.input-area textarea {
  flex: 1;
  resize: none;
  border: 1.5px solid #23233b;
  border-radius: 8px;
  font-size: 16px;
  padding: 10px;
  font-family: inherit;
  margin-bottom: 0;
  outline: none;
  transition: border-color 0.18s;
  min-height: 40px;
  background: #191920;
  color: #fff;
}

.input-area textarea:focus {
  border-color: #5a8fff;
}

.input-area button {
  background: #23233b;
  color: #fff;
  border: none;
  border-radius: 8px;
  padding: 10px 22px;
  font-size: 16px;
  font-weight: 700;
  cursor: pointer;
  transition: background 0.16s;
}

.input-area button:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(14px); }
  to { opacity: 1; transform: translateY(0); }
}

/* Loader Spinner */
.loader,
.loader:before,
.loader:after {
  border-radius: 50%;
}
.loader {
  color: #5a8fff;
  font-size: 10px;
  text-indent: -9999em;
  margin: 0 6px 0 0;
  position: relative;
  width: 18px;
  height: 18px;
  box-shadow: inset 0 0 0 2px;
  transform: translateZ(0);
  animation: load6 1.1s infinite linear;
  display: inline-block;
}
@keyframes load6 {
  0% { transform: rotate(0deg);}
  100% { transform: rotate(360deg);}
}
import React from 'react';
import ChatBox from './components/ChatBox';
import './App.css';

function App() {
  return (
    <div className="app-bg">
      <header className="header">
        <img src="https://cdn-icons-png.flaticon.com/512/4712/4712035.png" alt="AI Icon" className="logo"/>
        <h2>AI Q&A Chat</h2>
      </header>
      <main className="main-container">
        <ChatBox />
      </main>
      <footer className="footer">
        &copy; {new Date().getFullYear()} SPARTA12323 &mdash; Powered by OpenAI
      </footer>
    </div>
  );
}

export default App;
import React, { useState } from 'react';
import './ChatBox.css';

const API_URL = 'http://localhost:5000/api/ask';

function ChatBox() {
  const [question, setQuestion] = useState('');
  const [messages, setMessages] = useState([]);
  const [loading, setLoading] = useState(false);

  const askAI = async () => {
    if (!question.trim()) return;
    const userMsg = { from: 'user', text: question };
    setMessages([...messages, userMsg]);
    setLoading(true);
    setQuestion('');
    try {
      const res = await fetch(API_URL, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ question: userMsg.text }),
      });
      const data = await res.json();
      setMessages(msgs => [
        ...msgs,
        { from: 'ai', text: data.answer || data.error || 'No response' }
      ]);
    } catch (e) {
      setMessages(msgs => [
        ...msgs,
        { from: 'ai', text: 'Error connecting to AI service.' }
      ]);
    }
    setLoading(false);
  };

  const handleKeyDown = e => {
    if (e.key === 'Enter' && !e.shiftKey) {
      e.preventDefault();
      askAI();
    }
  };

  return (
    <div className="chatbox">
      <div className="messages">
        {messages.map((m, i) => (
          <div
            key={i}
            className={`bubble ${m.from === 'user' ? 'user' : 'ai'}`}
          >
            {m.text}
          </div>
        ))}
        {loading && (
          <div className="bubble ai">
            <span className="loader"></span>
            <span style={{marginLeft: 8}}>AI is thinking...</span>
          </div>
        )}
      </div>
      <form className="input-area" onSubmit={e => { e.preventDefault(); askAI(); }}>
        <textarea
          rows={2}
          value={question}
          onChange={e => setQuestion(e.target.value)}
          onKeyDown={handleKeyDown}
          placeholder="Ask a question..."
          disabled={loading}
        />
        <button type="submit" disabled={loading || !question.trim()}>
          Send
        </button>
      </form>
    </div>
  );
}

export default ChatBox;
