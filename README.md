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
fastapi
uvicorn
streamlit
transformers
torch
requests
wikipedia-api
pytest
httpx
pydanticpip install -r requirements.txtfrom transformers import pipeline

# Zero-shot classifier
classifier = pipeline(
    "zero-shot-classification",
    model="facebook/bart-large-mnli"
)

DEFAULT_LABELS = [
    "Artificial Intelligence",
    "Machine Learning",
    "Networking",
    "Technology",
    "Healthcare",
    "Education",
    "Finance",
    "Blockchain",
    "Cybersecurity",
    "Cloud Computing",
    "Data Science",
    "Sustainability"
]

def extract_themes(description: str, labels=None):
    """
    Extract top 3 themes from an event description.
    """
    if labels is None:
        labels = DEFAULT_LABELS

    result = classifier(description, labels)

    return result["labels"][:3]from app.services.event_analyzer import extract_themes

text = """
Join us for an AI and Machine Learning conference
focused on Data Science, Cloud Computing,
and the future of Artificial Intelligence.
"""

themes = extract_themes(text)

print(themes)[
'Artificial Intelligence',
'Machine Learning',
'Data Science'
]git add .
git commit -m "Added BART Theme Extraction Model"
git push
from transformers import pipeline, set_seed

# Load GPT-2 model
generator = pipeline(
    "text-generation",
    model="gpt2"
)

# Fixed seed for reproducible results
set_seed(42)

def generate_topics(themes, interests):
    prompt = f"""
You are a professional networking coach.

Event Themes:
{', '.join(themes)}

User Interests:
{', '.join(interests)}

Generate exactly 3 short, professional networking conversation starters.
Each starter should be on a new line.
"""

    result = generator(
        prompt,
        max_length=120,
        num_return_sequences=1,
        do_sample=True,
        temperature=0.7
    )

    text = result[0]["generated_text"]

    # Remove promptfrom app.services.topic_generator import generate_topics

themes = [
    "Artificial Intelligence",
    "Networking",
    "Technology"
]

interests = [
    "Python",
    "Machine Learning",
    "Data Science"
]

result = generate_topics(themes, interests)

for i, topic in enumerate(result, 1):
    print(f"{i}. {topic}")
    1. What inspired you to attend this AI networking event?
2. Which machine learning trends do you think will have the biggest impact this year?
3. Have you worked on any interesting Python or data science projects recently?git add .
git commit -m "Added GPT-2 Conversation Generator"
git push
import requests

WIKI_API = "https://en.wikipedia.org/api/rest_v1/page/summary/"

def fact_check(query: str):
    try:
        url = WIKI_API + query.replace(" ", "_")

        response = requests.get(url, timeout=5)

        if response.status_code == 200:
            data = response.json()
            return {
                "status": "Verified",
                "summary": data.get("extract", "No summary available."),
                "source": data.get("content_urls", {}).get("desktop", {}).get("page", "")
            }

        return {
            "status": "Not Found",
            "summary": "No matching article
	    from app.services.fact_checker import fact_check

result = fact_check("Artificial Intelligence")

print(result)
{
 'status': 'Verified',
 'summary': 'Artificial intelligence (AI) is intelligence demonstrated by machines...',
 'source': 'https://en.wikipedia.org/wiki/Artificial_intelligence'
}from app.services.event_analyzer import extract_themes
from app.services.topic_generator import generate_topics
from app.services.fact_checker import fact_check
@router.post("/analyze-event")
def analyze_event(request: EventRequest):
    themes = extract_themes(request.description)
    return {"themes": themes}


@router.post("/generate-conversation")
def generate_conversation(request: ConversationRequest):
    themes = extract_themes(request.description)
    suggestions = generate_topics(themes, request.interests)

    return {
        "themes": themes,
        "suggestions": suggestions
    }


@router.post("/fact-check")
def check_fact(request: FactCheckRequest):
    return fact_check(request.query)
    git add .
git commit -m "Added Wikipedia Fact Checker"
git push origin main
import streamlit as st
import requests

BASE_URL = "http://127.0.0.1:8000"

...

if "suggestions" in st.session_state:
    st.subheader("Conversation Starters")

    for i, suggestion in enumerate(st.session_state["suggestions"]):
        st.write(f"**{i+1}. {suggestion}**")

        col1, col2 = st.columns(
	class FeedbackRequest(BaseModel):
    suggestion: str
    action: str
    from app.models import FeedbackRequest
from app.services.feedback_logger import save_feedback

@router.post("/feedback")
def feedback(request: FeedbackRequest):

    save_feedback(
        request.suggestion,
        request.action
    )

    return {
        "message": "Feedback saved successfully"
    }
    import json

st.header("Previous Conversations")

try:
    with open("history.json") as file:

        history = json.load(file)

        for item in history[-5:]:

            st.write(item["timestamp"])

            st.write(item["event"])

            st.write(item["themes"])

            st.write(item["suggestions"])

            st.divider()

except:
    st.write("No history available.")
    git add .
git commit -m "Added Feedback and History"
git push origin main
# Personalized Networking Assistant

## Overview
An AI-powered networking assistant that helps users generate personalized conversation starters using Hugging Face Transformers and verifies facts using Wikipedia.

## Features
- Theme Extraction using BART (Zero-shot Classification)
- Conversation Generation using GPT-2
- Wikipedia Fact Checker
- FastAPI Backend
- Streamlit Frontend
- Conversation History
- Feedback System

## Tech Stack
- Python
- FastAPI
- Streamlit
- Transformers
- Torch
- Wikipedia API
- Pytest

## Installation

pip install -r requirements.txt

## Run Backend

uvicorn app.main:app --reload

## Run Frontend

streamlit run frontend/app.py

## API Documentation

http://127.0.0.1:8000/docs

## Author

Naman Tiwari
from fastapi.testclient import TestClient
from app.main import app

client = TestClient(app)

def test_home():
    response = client.get("/")
    assert response.status_code == 200

def test_generate():
    response = client.post(
        "/generate-conversation",
        json={
            "description": "AI Conference",
            "interests": ["Python", "Machine Learning"]
        }
    )

    assert response.status_code == 200
    pytest
    Personalized-Networking-Assistant/
│
├── app/
│   ├── main.py
│   ├── models.py
│   ├── routes/
│   │   └── conversation.py
│   └── services/
│       ├── event_analyzer.py
│       ├── topic_generator.py
│       ├── fact_checker.py
│       ├── history_logger.py
│       └── feedback_logger.py
│
├── frontend/
│   └── app.py
│
├── tests/
│   └── test_routes.py
│
├── history.json
├── feedback.json
├── requirements.txt
├── README.md
└── .gitignore
git add .
git commit -m "Final SkillWallet Submission"
git push origin main
from pydantic import BaseModel, Field
from typing import List


class EventRequest(BaseModel):
    description: str = Field(..., min_length=10)


class ThemeResponse(BaseModel):
    themes: List[str]


class ConversationRequest(BaseModel):
    description: str = Field(..., min_length=10)
    interests: List[str]


class ConversationResponse(BaseModel):
    themes: List[str]
    suggestions: List[str]


class FactCheckRequest(BaseModel):
    query: str


class FactCheckResponse(BaseModel):
    status: str
    summary: str
    source: str


class FeedbackRequest(BaseModel):
    suggestion: str
    action: str
from fastapi import FastAPI
from app.routes.conversation import router

app = FastAPI(
    title="Personalized Networking Assistant",
    version="1.0.0",
    description="AI Powered Networking Assistant using FastAPI and Streamlit"
)

app.include_router(router)


@app.get("/")
def home():
    return {
        "message": "Welcome to Personalized Networking Assistant",
        "status": "Running"
    }uvicorn app.main:app --reloadhttp://127.0.0.1:8000/docsgit add .
git commit -m "Completed models and FastAPI entry point"
git push
import streamlit as st
import requests

BASE_URL = "http://127.0.0.1:8000"

st.set_page_config(
    page_title="Personalized Networking Assistant",
    page_icon="🤝",
    layout="wide"
)

st.title("🤝 Personalized Networking Assistant")
st.write("Generate smart networking conversation starters using AI.")

# Event Description
event_description = st.text_area(
    "Enter Event Description",
    height=150,
    placeholder="Example: AI Conference focused on Machine Learning and Data Science..."
)

# User Interests
user_interests = st.text_input(
    "Your Interests (comma separated)",
    placeholder="Python, AI, Machine Learning"
)

if st.button("🚀 Generate Conversation"):

    if event_description.strip() == "":
        st.error("Please enter an event description.")
    else:

        interests = [
            i.strip()
            for i in user_interests.split(",")
            if i.strip()
        ]

        response = requests.post(
            f"{BASE_URL}/generate-conversation",
            json={
                "description": event_description,
                "interests": interests
            }
        )

        if response.status_code == 200:

            data = response.json()

            st.success("Conversation generated successfully!")

            st.subheader("🎯 Event Themes")

            for theme in data["themes"]:
                st.write("✅", theme)

            st.subheader("💬 Conversation Starters")

            for suggestion in data["suggestions"]:
                st.info(suggestion)

        else:
            st.error("Unable to connect to the backend.")
	    st.divider()

st.header("📚 Wikipedia Fact Checker")

query = st.text_input("Enter Topic")

if st.button("🔍 Check Fact"):

    response = requests.post(
        f"{BASE_URL}/fact-check",
        json={
            "query": query
        }
    )

    if response.status_code == 200:

        result = response.json()

        st.success(result["status"])
        st.write(result["summary"])

        if result["source"]:
            st.write("Source:", result["source"])uvicorn app.main:app --reload
	    streamlit run frontend/app.py
	    http://localhost:8501
	    from fastapi import APIRouter
from app.models import (
    EventRequest,
    ConversationRequest,
    FactCheckRequest,
    FeedbackRequest
)

from app.services.event_analyzer import extract_themes
from app.services.topic_generator import generate_topics
from app.services.fact_checker import fact_check
from app.services.history_logger import log_history
from app.services.feedback_logger import save_feedback

router = APIRouter(prefix="/api", tags=["Networking Assistant"])


@router.post("/analyze-event")
def analyze_event(request: EventRequest):
    themes = extract_themes(request.description)
    return {
        "success": True,
        "themes": themes
    }


@router.post("/generate-conversation")
def generate_conversation(request: ConversationRequest):

    themes = extract_themes(request.description)

    suggestions = generate_topics(
        themes,
        request.interests
    )

    log_history(
        request.description,
        themes,
        suggestions
    )

    return {
        "success": True,
        "themes": themes,
        "suggestions": suggestions
    }


@router.post("/fact-check")
def check_fact(request: FactCheckRequest):

    result = fact_check(request.query)

    return result


@router.post("/feedback")
def feedback(request: FeedbackRequest):


   uvicorn app.main:app --reload
   http://127.0.0.1:8000/docs
   
