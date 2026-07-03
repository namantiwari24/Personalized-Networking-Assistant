# Personalized-Networking-Assistant
An AI-powered networking assistant built using FastAPI, Streamlit, Hugging Face Transformers, and the Wikipedia API.

## Features

- AI-based event theme extraction
- Personalized conversation starter generation
- Wikipedia fact checking
- Conversation history
- Feedback logging
- REST API with FastAPI
- Interactive Streamlit interface

## Tech Stack

- Python
- FastAPI
- Streamlit
- Hugging Face Transformers
- GPT-2
- Wikipedia API
- Pytest

## Project Structure

app/
frontend/
tests/

## Run

Backend

uvicorn app.main:app --reload

Frontend

streamlit run frontend/app.py
Personalized-Networking-Assistant/
│
├── app/
│   ├── routes/
│   └── services/
│
├── frontend/
│
├── tests/
│
├── requirements.txt
├── README.md
├── history.json
├── feedback.json
└── .gitignore__pycache__/
*.pyc
.venv/
venv/
.env
history.json
feedback.jsonfastapi
uvicorn
streamlit
transformers
torch
requests
wikipedia-api
pytest
httpx
pydantic
[]
[]
Initial project structure
from fastapi import FastAPI
from app.routes.conversation import router

app = FastAPI(
    title="Personalized Networking Assistant",
    version="1.0.0"
)

app.include_router(router)

@app.get("/")
def home():
    return {
        "message": "Personalized Networking Assistant API is Running!"
    }from pydantic import BaseModel
from typing import List

class EventRequest(BaseModel):
    description: str

class ConversationRequest(BaseModel):
    description: str
    interests: List[str]

class FactCheckRequest(BaseModel):
    query: str

class ThemeResponse(BaseModel):
    themes: List[str]

class ConversationResponse(BaseModel):
    themes: List[str]
    suggestions: List[str]

class FactCheckResponse(BaseModel):
    result: str
	from fastapi import APIRouter
from app.models import (
    EventRequest,
    ConversationRequest,
    FactCheckRequest
)

router = APIRouter()

@router.post("/analyze-event")
def analyze_event(request: EventRequest):
    return {
        "themes": ["AI", "Networking", "Technology"]
    }

@router.post("/generate-conversation")
def generate_conversation(request: ConversationRequest):
    return {
        "themes": ["AI", "Networking"],
        "suggestions": [
            "What inspired you to attend this event?",
            "Which AI technologies interest you the most?",
            "What projects are you currently working on?"
        ]
    }

@router.post("/fact-check")
def fact_check(request: FactCheckRequest):
    return {
        "result": "Wikipedia fact-check will be implemented in the next step."
    }
	Added FastAPI backend with API routes
	app/services/
	from transformers import pipeline

classifier = pipeline(
    "zero-shot-classification",
    model="facebook/bart-large-mnli"
)

LABELS = [
    "Artificial Intelligence",
    "Technology",
    "Healthcare",
    "Education",
    "Finance",
    "Blockchain",
    "Cybersecurity",
    "Networking"
]

def extract_themes(description):
    result = classifier(description, LABELS)
    return result["labels"][:3]
	from transformers import pipeline

generator = pipeline(
    "text-generation",
    model="gpt2"
)

def generate_topics(themes, interests):
    prompt = f"""
Event Themes: {', '.join(themes)}
User Interests: {', '.join(interests)}

Generate 3 short networking conversation starters:
"""

    output = generator(
        prompt,
        max_length=80,
        num_return_sequences=1
    )[0]["generated_text"]

    return output.split("\n")[:3]
	import requests

def fact_check(query):
    url = f"https://en.wikipedia.org/api/rest_v1/page/summary/{query}"

    try:
        response = requests.get(url)

        if response.status_code == 200:
            return response.json().get("extract", "No information found.")

        return "No information found."

    except Exception:
        return "Unable to connect to Wikipedia."
		Added AI services (BART, GPT-2, Wikipedia)
		import streamlit as st
import requests

BASE_URL = "http://127.0.0.1:8000"

st.set_page_config(page_title="Personalized Networking Assistant")

st.title("🤝 Personalized Networking Assistant")

event = st.text_area("Event Description")

interests = st.text_input(
    "Your Interests (comma separated)"
)

if st.button("Generate Conversation"):

    response = requests.post(
        f"{BASE_URL}/generate-conversation",
        json={
            "description": event,
            "interests": [
                i.strip() for i in interests.split(",")
            ]
        }
    )

    if response.status_code == 200:

        data = response.json()

        st.subheader("Themes")

        for theme in data["themes"]:
            st.success(theme)

        st.subheader("Conversation Starters")

        for item in data["suggestions"]:
            st.write("•", item)

st.divider()

st.header("Wikipedia
uvicorn app.main:app --reload
streamlit run frontend/app.py
Added Streamlit frontend
import json
from pathlib import Path
from datetime import datetime

FILE = Path("history.json")

def log_history(event, themes, suggestions):
    record = {
        "timestamp": datetime.now().isoformat(),
        "event": event,
        "themes": themes,
        "suggestions": suggestions
    }

    if FILE.exists():
        data = json.loads(FILE.read_text())
    else:
        data = []

    data.append(record)
    FILE.write_text(json.dumps(data, indent=4))


def load_history():
    if FILE.exists():
        return json.loads(FILE.read_text())
    return []
	import json
from pathlib import Path
from datetime import datetime

FILE = Path("feedback.json")

def save_feedback(text, action):
    record = {
        "timestamp": datetime.now().isoformat(),
        "suggestion": text,
        "action": action
    }

    if FILE.exists():
        data = json.loads(FILE.read_text())
    else:
        data = []

    data.append(record)
    FILE.write_text(json.dumps(data, indent=4))
	from app.services.history_logger import log_history

log_history(
    request.description,
    themes,
    suggestions
)
Added History and Feedback Logger
from fastapi import APIRouter
from app.models import EventRequest, ConversationRequest, FactCheckRequest
from app.services.event_analyzer import extract_themes
from app.services.topic_generator import generate_topics
from app.services.fact_checker import fact_check
from app.services.history_logger import log_history

router = APIRouter()

@router.post("/analyze-event")
def analyze_event(request: EventRequest):
    themes = extract_themes(request.description)
    return {"themes": themes}

@router.post("/generate-conversation")
def generate_conversation(request: ConversationRequest):
    themes = extract_themes(request.description)
    suggestions = generate_topics(themes, request.interests)

    log_history(
        request.description,
        themes,
        suggestions
    )

    return {
        "themes": themes,
        "suggestions": suggestions
    }

@router.post("/fact-check")
def check_fact(request: FactCheckRequest):
    result = fact_check(request.query)
    return {"result": result}
	from fastapi.testclient import TestClient
from app.main import app

client = TestClient(app)

def test_home():
    response = client.get("/")
    assert response.status_code == 200

def test_analyze():
    response = client.post(
        "/analyze-event",
        json={"description": "AI conference in Delhi"}
    )
    assert response.status_code == 200
	pytest
	gPersonalized-Networking-Assistant/
│
├── app/
│   ├── main.py
│   ├── models.py
│   ├── routes/
│   └── services/
│
├── frontend/
│
├── tests/
│
├── history.json
├── feedback.json
├── requirements.txt
├── README.md
└── .gitignoregit add .
git commit -m "Completed Personalized Networking Assistant"
git push origin main
uvicorn app.main:app --reload
streamlit run frontend/app.py
git add .
git commit -m "Final SkillWallet Project"
git push origin main
