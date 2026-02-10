# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Ice Breaker is a Flask web application that generates personalized conversation starters by analyzing LinkedIn and Twitter profiles using LangChain and OpenAI. It is a learning project for the [LangChain Udemy course](https://www.udemy.com/course/langchain/).

## Development Commands

```bash
# Install dependencies
pipenv install

# Run the Flask app (serves on http://localhost:5000)
pipenv run python app.py

# Format code
pipenv run black .

# Sort imports
pipenv run isort .

# Lint
pipenv run pylint [module_name]

# Tests (no tests exist yet)
pipenv run pytest .
```

Python version: 3.10 (specified in Pipfile).

## Architecture

The app follows a pipeline pattern orchestrated by `ice_breaker.py`:

1. **Agents** (`agents/`) — LangChain ReAct agents (GPT-4o-mini, temperature=0) use Tavily search to find a person's LinkedIn URL and Twitter username. They pull the ReAct prompt from LangChain Hub (`hwchase17/react`).

2. **Data extraction** (`third_parties/`) — `linkedin.py` calls the Scrapin.io API to scrape LinkedIn profiles. `twitter.py` wraps Tweepy but defaults to `scrape_user_tweets_mock()` which returns mock data from a GitHub Gist.

3. **Chains** (`chains/custom_chains.py`) — Three LangChain `RunnableSequence` chains (prompt | llm | parser) process the scraped data:
   - Summary chain (GPT-3.5-turbo, temp=0) → `Summary` (short summary + 2 facts)
   - Interests chain (GPT-3.5-turbo, temp=0) → `TopicOfInterest` (3 topics)
   - Ice breaker chain (GPT-3.5-turbo, temp=1) → `IceBreaker` (2 ice breakers)

4. **Output parsing** (`output_parsers.py`) — Pydantic models with `PydanticOutputParser` for structured LLM responses.

5. **Web layer** (`app.py`) — Flask server with a single page (`templates/index.html`) using vanilla JS and MVP.css. The `/process` POST endpoint accepts a name and returns JSON.

## Environment Variables

Required in `.env`:
- `OPENAI_API_KEY` — OpenAI API (used by both agents and chains)
- `SCRAPIN_API_KEY` — Scrapin.io for LinkedIn scraping
- `TAVILY_API_KEY` — Tavily for agent web search

Optional:
- `TWITTER_API_KEY`, `TWITTER_API_SECRET`, `TWITTER_ACCESS_TOKEN`, `TWITTER_ACCESS_SECRET` — Twitter API (mock fallback exists)
- `LANGCHAIN_TRACING_V2=true`, `LANGCHAIN_API_KEY`, `LANGCHAIN_PROJECT=ice_breaker` — LangSmith tracing (will error if `TRACING_V2=true` without a valid API key)

## Key Implementation Details

- `ice_breaker.py` currently hardcodes `scrape_user_tweets_mock` instead of the real `scrape_user_tweets`
- Agents use `AgentExecutor` with `verbose=True` — debug output goes to stdout
- Flask runs in debug mode on `0.0.0.0:5000`
- The `tools/tools.py` module wraps `TavilySearchResults` as a LangChain `Tool`
