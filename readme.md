# SavageScript

SavageScript is a small AI roast-and-compliment app built around Google Gemini. You enter a short self-description, the backend generates a roast plus a sincere compliment, and the web UI stores recent results in a leaderboard.

The repository contains two runnable paths:

- A Flask + React web app with SQLite persistence.
- A terminal CLI version in [src/savage_script.py](src/savage_script.py).

## Overview

The web app exposes a simple text prompt experience. The backend sends the user input to Gemini with a fixed system prompt that asks for a single JSON object containing `roast` and `compliment`. The Flask API validates the response, falls back to a safe default if parsing fails, and stores successful generations in SQLite.

The React frontend provides the input form, the roast and compliment cards, dark/light theme toggling, toast notifications, a small audio cue on submit, and a recent-roasts leaderboard.

## Features

- Roast + compliment generation from one short user description.
- Flask API with `/api/health`, `/api/roast`, and `/api/leaderboard` endpoints.
- React UI with animated sections, theme toggle, and toast feedback.
- SQLite storage for recent roast history.
- Dockerized deployment path for the full web app.
- Separate CLI implementation for terminal use.

## Tech Stack

- Python 3.11
- Flask
- Flask-CORS
- python-dotenv
- google-generativeai
- SQLite
- React 18
- Framer Motion
- React Hot Toast
- Tailwind CSS
- Docker and Docker Compose

## Architecture

The app is split into a Python backend and a React frontend:

- [app.py](app.py) boots the Flask server, serves the built React app from `frontend/build`, connects to Gemini, and writes roast history to `savage_script.db`.
- [frontend/src/App.js](frontend/src/App.js) manages the main page state, fetches the leaderboard, submits roast requests, and handles theme persistence.
- [frontend/src/components/InputSection.js](frontend/src/components/InputSection.js) renders the confession textarea and submit button.
- [frontend/src/components/ResultsSection.js](frontend/src/components/ResultsSection.js) renders the roast and compliment output.
- [frontend/src/components/LeaderboardSection.js](frontend/src/components/LeaderboardSection.js) shows recent saved roasts.
- [frontend/src/components/ThemeToggle.js](frontend/src/components/ThemeToggle.js) switches between dark and light mode.

Request flow:

1. The user enters a description in the React UI.
2. The frontend POSTs the text to `/api/roast`.
3. Flask sends a structured prompt to Gemini.
4. The response is parsed as JSON and saved in SQLite.
5. The frontend displays the roast, compliment, and refreshed leaderboard.

The CLI path in [src/savage_script.py](src/savage_script.py) uses the same Gemini API key and prompt idea, but runs entirely in the terminal.


## Installation

### Prerequisites

- Python 3.11 or newer
- Node.js 18 or newer
- A valid Gemini API key

### Backend dependencies

```bash
python -m pip install -r requirements.txt
```

### Frontend dependencies

```bash
cd frontend
npm install
```

## Environment Variables

Create a `.env` file in the project root with:

```env
GEMINI_API_KEY=your_api_key_here
```

The app reads only `GEMINI_API_KEY`. The Docker image also expects the same variable to be available at runtime.

## Running Locally

### Web app in development

Run the backend in one terminal:

```bash
python app.py
```

Run the frontend in another terminal:

```bash
cd frontend
npm start
```

The React app proxies API requests to `http://localhost:5000`, while the Flask server runs on port `5000`.

### Web app with Docker

```bash
docker-compose up --build
```

Then open `http://localhost:5000`.

### CLI mode

```bash
python src/savage_script.py
```

## Notes

- Successful roast generations are stored in `savage_script.db`.
- The leaderboard shows recent saved roasts only.
- If Gemini returns malformed output, the API falls back to a safe default roast and compliment.

Dynamic prompting allows the model to generate responses based on real-time input, adjusting its behavior as the conversation or task progresses. The model dynamically updates its approach based on the provided context, making it more responsive and tailored to the user's needs. Unlike other methods, the prompt changes continuously depending on the situation.

## How it’s used in SavageScript

In dynamic prompting, the model generates a roast and a compliment based on the user's description, adjusting both the tone and content depending on the given context. The model uses the input information and modifies its approach as necessary, without rigid examples or formats.

## SavageScript (Dynamic Prompt)

**Task:** Generate a witty, personalized roast and a sincere, uplifting compliment based on the user's input.

**Constraints:**

- The model must adjust its approach dynamically based on the input provided.
- The roast and compliment should be directly tailored to the user’s description.
- **Format:** Return a single JSON object with exactly two keys: `"roast"` and `"compliment"`. Do not include any extra text before or after the JSON.

**Example:**

```json
{
  "roast": "You're like a coffee cup—always warm, but no one really notices until you're gone.",
  "compliment": "Your love for books shows how deep and thoughtful you are, even in silence."
}
```
**Input:**

user_description="A coffee-loving introvert who’s always buried in books and avoids large gatherings."


**Note:** The model should modify its response dynamically based on the provided input, ensuring that the roast and compliment are unique to each user description. The model must follow the required JSON format.

# Chain of Thought Prompting

## What is Chain of Thought Prompting?

Chain of thought prompting encourages the model to work through a problem or task step by step, providing intermediate reasoning and thought processes before arriving at the final output. This method helps the model to break down complex tasks and ensures more logical and structured responses.

## How it’s used in SavageScript

In chain of thought prompting, the model generates a roast and a compliment by thinking through the necessary steps and providing an intermediate reasoning process. The model generates the final output only after reasoning through each part of the task to ensure accuracy and relevance to the user’s input.

## SavageScript (Chain of Thought Prompt)

**Task:** Generate a witty, personalized roast and a sincere, uplifting compliment based on the user's input. Think through the reasoning step by step and explain your thought process before generating the final response.

**Constraints:**

- The model must work through the task step by step, providing reasoning before the final response.
- Each part of the output must be reasoned through and explained logically.
- **Format:** Return a single JSON object with exactly two keys: `"roast"` and `"compliment"`. Do not include any extra text before or after the JSON.

**Example:**

user_description="A charismatic leader who loves giving motivational speeches and rallies the team during tough times."


**Chain of Thought Reasoning:**

- First, I want to acknowledge that this person is a leader who motivates and energizes others.
- The roast should play on the idea that sometimes motivational speeches can be a bit over the top or cliché.
- The compliment should highlight their ability to inspire and support others, which is a key trait of a great leader.

**Generated Output:**

```json
{
  "roast": "You give motivational speeches like a broken record—great at first, but after a while, you start repeating yourself.",
  "compliment": "Your leadership is truly inspiring; you lift everyone around you and help them see the best in themselves."
}
```
