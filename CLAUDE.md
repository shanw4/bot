# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Experimental boilerplate for building bots compatible with Open WebUI "Channels" (v0.5.0+). The APIs are unstable and not production-ready.

## Running

```bash
# Run a specific example
python -m examples.<example_name>
# e.g.:
python -m examples.channel_ai_bot
python -m examples.duckduckgo-agent
```

## Configuration

Copy `.env` (not committed) and set:
- `WEBUI_URL` — Open WebUI instance URL (default: `http://localhost:8080`)
- `TOKEN` — authentication token for the bot user

## Architecture

**Core files:**
- `env.py` — loads `.env` vars via python-dotenv
- `utils.py` — `send_message(channel_id, message)` (HTTP POST) and `send_typing(sio, channel_id)` (Socket.IO emit)
- `main.py` — minimal bot skeleton showing the connection/auth/event loop pattern

**Communication flow:**
1. Connect async Socket.IO client to `WEBUI_URL/ws/socket.io`
2. Authenticate with a `user-join` event carrying the TOKEN
3. Listen for channel events (`channel-events` in older API; `events:channel` in v0.5.10+)
4. Filter out own messages by checking `user_id`
5. Respond via HTTP to `/api/v1/channels/{id}/messages/post`; send typing indicators via Socket.IO

**Examples** (`examples/`) show progressively more complex patterns:
- `ai.py` — simple echo/pong
- `channel_ai_bot.py` — filters messages prefixed with `"AI:"`, calls OpenAI chat completions (most up-to-date against v0.5.10)
- `smolagents.py` / `duckduckgo-agent.py` — integrate `smolagents` library with `CodeAgent`/`ToolCallingAgent`; run blocking calls in `asyncio.get_event_loop().run_in_executor`

**Stack:** Python 3, `python-socketio[asyncio_client]`, `aiohttp`, `python-dotenv`; optionally `smolagents`, `litellm`
