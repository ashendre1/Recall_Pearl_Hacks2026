Recall

Turn what you read into what you remember

Recall is an intelligent learning platform that helps you remember what you've read by generating personalized quizzes from web-scraped content. It uses semantic search, AI-powered question generation, and audio evaluation to create an engaging learning experience.

## 🎯 Features

- **Web Scraping**: Chrome extension to scrape and store any webpage content
- **Semantic Search**: Find relevant content using vector embeddings, not just keyword matching
- **AI Quiz Generation**: Automatically generate quiz questions using Google Gemini
- **Text Quizzes**: Multiple-choice and true/false questions with instant feedback
- **Audio Quizzes**: Record spoken answers with fluency and articulateness evaluation
- **Personalized Feedback**: AI-generated feedback tailored to your answers

## 🏗️ Architecture

The project consists of four main components:

1. **Chrome Extension** (`extension/`) - Scrapes web pages and sends data to backend
2. **Node.js Backend** (`backend/`) - Receives scraped content, generates embeddings, stores in MongoDB
3. **FastAPI Backend** (`backend-fastapi/`) - Quiz generation and evaluation service
4. **Next.js Frontend** (`frontend/`) - Web interface for taking quizzes

## 📋 Prerequisites

- Node.js 18+ and npm
- Python 3.9+
- MongoDB Atlas account (or local MongoDB)
- API Keys:
  - Hugging Face (for embeddings)
  - Google Gemini (for quiz generation)
  - Eleven Labs (for audio transcription, optional)

## 🚀 Quick Start

### 1. Clone the Repository

```bash
git clone <repository-url>
cd PH-Recall
```

### 2. Set Up Chrome Extension

1. Open Chrome and navigate to `chrome://extensions/`
2. Enable "Developer mode"
3. Click "Load unpacked" and select the `extension/` folder
4. The extension icon should appear in your toolbar

### 3. Set Up Node.js Backend (Scraping Service)

```bash
cd backend
npm install
```

Create a `.env` file:

```env
MONGO_URI=mongodb+srv://username:password@cluster.mongodb.net/?retryWrites=true&w=majority
MONGO_DB=recall
MONGO_COLLECTION=paragraphs
HF_API_KEY=your_huggingface_api_key_here
PORT=3001
```

Start the server:

```bash
npm start
# or for development with auto-reload
npm run dev
```

The server runs on `http://localhost:3001`

### 4. Set Up FastAPI Backend (Quiz Service)

```bash
cd backend-fastapi
pip install -r requirements.txt
```

Create a `.env` file (copy from `env.example`):

```env
MONGO_URI=mongodb+srv://username:password@cluster.mongodb.net/?retryWrites=true&w=majority
MONGO_DB=recall
MONGO_COLLECTION=paragraphs
HF_API_KEY=your_huggingface_api_key_here
GEMINI_API_KEY=your_gemini_api_key_here
ELEVENLABS_API_KEY=your_elevenlabs_api_key_here
PORT=8000
```

Start the server:

```bash
python main.py
# or with uvicorn directly
uvicorn main:app --reload --port 8000
```

The API runs on `http://localhost:8000`

### 5. Set Up Frontend

```bash
cd frontend
npm install
```

Create a `.env.local` file (if needed for API endpoints):

```env
NEXT_PUBLIC_API_URL=http://localhost:8000
```

Start the development server:

```bash
npm run dev
```

The frontend runs on `http://localhost:3000`

## 📖 How It Works

### 1. Scraping Content

1. Navigate to any webpage you want to learn from
2. Click the Recall extension icon
3. Click "Scrape" to extract content
4. The extension sends paragraphs to the Node.js backend
5. Each paragraph is converted to a 384-dimensional embedding vector
6. Paragraphs and embeddings are stored in MongoDB

### 2. Generating Quizzes

1. Open the frontend at `http://localhost:3000`
2. Enter a topic you want to be quizzed on
3. Choose Text Mode or Audio Mode
4. The FastAPI backend:
   - Converts your topic to an embedding
   - Finds the top 3 most relevant paragraphs using cosine similarity
   - Sends paragraphs to Gemini AI to generate questions
   - Returns questions to the frontend

### 3. Taking Quizzes

**Text Mode:**
- Answer multiple-choice and true/false questions
- Get instant scoring and personalized feedback from Gemini

**Audio Mode:**
- Listen to the question (text-to-speech)
- Record your spoken answer
- Get feedback on:
  - **Speech fluency** (from Eleven Labs) - analyzes hesitancy and pauses
  - **Articulateness** (from Gemini) - evaluates clarity and expression

## 🔧 Configuration

### MongoDB Collections

The system uses two main collections:

- `paragraphs` (or `MONGO_COLLECTION`) - Stores scraped paragraphs with embeddings
- `quiz_questions` - Stores generated quiz questions
- `scraped_pages` - Stores original scraped page data (optional)

### Environment Variables

**Node.js Backend** (`backend/.env`):
- `MONGO_URI` - MongoDB connection string
- `MONGO_DB` - Database name
- `MONGO_COLLECTION` - Collection name for paragraphs
- `HF_API_KEY` - Hugging Face API key
- `PORT` - Server port (default: 3001)

**FastAPI Backend** (`backend-fastapi/.env`):
- `MONGO_URI` - MongoDB connection string
- `MONGO_DB` - Database name
- `MONGO_COLLECTION` - Collection name for paragraphs
- `HF_API_KEY` - Hugging Face API key
- `GEMINI_API_KEY` - Google Gemini API key
- `ELEVENLABS_API_KEY` - Eleven Labs API key (optional, for audio quizzes)
- `PORT` - Server port (default: 8000)

## 📡 API Endpoints

### FastAPI Backend

#### `POST /quiz/generate`

Generate quiz questions based on a topic.

**Request:**
```json
{
  "topic": "machine learning",
  "quizType": "text",
  "numQuestions": 5
}
```

**Response:**
```json
{
  "questions": [
    {
      "question": "What is machine learning?",
      "questionType": "mcq",
      "options": ["Option A", "Option B", "Option C", "Option D"],
      "correctAnswer": 0,
      "paragraphIds": ["id1", "id2", "id3"]
    }
  ],
  "topic": "machine learning",
  "paragraphsUsed": [...]
}
```

#### `POST /quiz/score`

Score text quiz answers.

**Request:**
```json
{
  "answers": [
    {
      "questionId": "q1",
      "question": "What is machine learning?",
      "options": ["A", "B", "C", "D"],
      "correctAnswer": 0,
      "selectedAnswer": 0
    }
  ]
}
```

#### `POST /quiz/score-audio`

Score audio quiz answer (multipart/form-data).

**Form Data:**
- `audio`: Audio file (webm, mp3, wav)
- `question`: Question text
- `topic`: Topic string
- `paragraphIds`: JSON string array

**Response:**
```json
{
  "score": 0.0,
  "transcribedText": "Your spoken answer...",
  "isHesitant": false,
  "elevenLabsFeedback": "Your speech flow is good...",
  "geminiFeedback": "Your answer is articulate...",
  "audioFeedbackUrl": null
}
```

#### `GET /health`

Health check endpoint.

### Node.js Backend

#### `POST /api/scrape`

Receive scraped content from extension.

**Request:**
```json
{
  "url": "https://example.com",
  "title": "Page Title",
  "headings": ["Heading 1", "Heading 2"],
  "paragraphs": ["Paragraph 1", "Paragraph 2"]
}
```

#### `GET /api/health`

Health check endpoint.

## 🛠️ Tech Stack

- **Frontend**: Next.js 16, React 19, TypeScript, Tailwind CSS
- **Backend (Quiz)**: FastAPI, Python, Uvicorn
- **Backend (Scraping)**: Node.js, Express
- **Database**: MongoDB Atlas
- **AI/ML**:
  - Google Gemini (question generation, feedback)
  - Hugging Face (embeddings)
  - Eleven Labs (speech-to-text)
- **Extension**: Chrome Extension Manifest V3

## 📁 Project Structure

```
PH-Recall/
├── extension/           # Chrome extension for web scraping
│   ├── manifest.json
│   ├── popup.html/js
│   ├── content.js
│   └── options.html/js
├── backend/            # Node.js backend (scraping service)
│   ├── server.js
│   ├── services/
│   │   ├── mongo.js
│   │   └── embedding.js
│   └── package.json
├── backend-fastapi/    # FastAPI backend (quiz service)
│   ├── main.py
│   ├── services/
│   │   ├── mongo.py
│   │   ├── embedding.py
│   │   ├── similarity.py
│   │   ├── gemini.py
│   │   └── elevenlabs.py
│   ├── requirements.txt
│   └── env.example
└── frontend/           # Next.js frontend
    ├── app/
    │   ├── page.tsx
    │   ├── text-quiz/
    │   ├── audio-quiz/
    │   └── ...
    ├── components/
    ├── lib/
    └── package.json
```

## 🧪 Development

### Running in Development Mode

**Node.js Backend:**
```bash
cd backend
npm run dev  # Auto-reloads on file changes
```

**FastAPI Backend:**
```bash
cd backend-fastapi
uvicorn main:app --reload --port 8000
```

**Frontend:**
```bash
cd frontend
npm run dev  # Next.js dev server
```

## 🐛 Troubleshooting

### Extension not working
- Make sure the Node.js backend is running on port 3001
- Check browser console for errors
- Verify extension permissions in `chrome://extensions/`

### Quiz generation fails
- Verify MongoDB connection and that paragraphs exist with embeddings
- Check that `GEMINI_API_KEY` is set correctly
- Ensure `HF_API_KEY` is valid

### Audio quiz not working
- Check browser microphone permissions
- Verify `ELEVENLABS_API_KEY` is set (optional but recommended)
- Audio features work best in Chrome, Edge, or Safari

### Embedding generation fails
- Verify `HF_API_KEY` is valid
- Check Hugging Face API status
- Ensure text is not too long (truncated at 8000 chars)



## 📧 Contact

mkatha14@gmail.com
ashendr1@charlotte.edu
mrunmayi1202@gmail.com
