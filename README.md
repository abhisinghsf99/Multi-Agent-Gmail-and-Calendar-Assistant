# Multi-Agent Gmail & Calendar Assistant

Multi-agent system built in n8n. Chat in natural language to manage your inbox and calendar. A Groq-powered orchestrator agent interprets intent and delegates to two specialized sub-agents — one for Gmail, one for Google Calendar — each with its own toolset. Conversational memory keeps context across turns.

![Multi-Agent Gmail & Calendar Assistant](./assets/workflow.png)

## How It Works

The system runs as a three-tier agent hierarchy:

**Orchestrator (AI Agent Manager)** — Receives chat messages, identifies intent, and routes to the right sub-agent. It resolves relative dates ("tomorrow", "next Friday") into absolute timestamps before delegating, chains sub-agents for cross-domain requests (e.g. "check Friday's calendar and email the details to John"), and asks a clarifying question instead of guessing when a request is ambiguous. Session-based memory (10-message window) is attached here.

**Gmail AI Agent** — Handles anything involving email: retrieve messages, send, reply, and delete.

**Calendar AI Agent** — Handles scheduling: retrieve events, create, update, and delete on the primary Google Calendar.

Each sub-agent is exposed to the orchestrator as a tool, so the orchestrator never touches Gmail or Calendar directly — it delegates, then summarizes the result back to the user. Destructive actions (sending or deleting email, deleting events) are confirmed with the user first.

## Stack

* **Orchestration:** n8n
* **LLM:** Groq (`openai/gpt-oss-120b`) — one instance per agent
* **Memory:** Simple Memory (session-based, 10-message window)
* **Email:** Gmail node (getAll, send, reply, delete)
* **Calendar:** Google Calendar node (getAll, create, update, delete)
* **Trigger:** Chat Trigger

## Setup

### Prerequisites

* n8n instance (cloud or self-hosted)
* Groq API key
* Google account with Gmail and Google Calendar
* Google Cloud OAuth2 credentials (Gmail API + Google Calendar API enabled)

### Installation

1. Import `Multi-Agent Gmail & Calendar Assistant.json` into your n8n instance (three-dot menu → Import from File)
2. Configure credentials:
   * Add your Groq API key to **Groq Chat Model**, **Groq Chat Model1**, and **Groq Chat Model2**
   * Connect Gmail OAuth2 to the four Gmail tool nodes
   * Connect Google Calendar OAuth2 to the four Calendar tool nodes
3. Update the system message in **AI Agent Manager** if your timezone isn't `America/Los_Angeles` (also set it under Workflow Settings → Timezone)
4. Replace the `Your-Name` placeholder in the **Send an email in Gmail** node so outgoing emails are signed correctly
5. Activate the workflow

## Usage

Open the chat trigger URL and talk to it in plain English:

* `What's on my calendar tomorrow?`
* `Show me my last 5 emails`
* `Reply to Sarah's email and tell her I'll be there at 3`
* `Block 2–3 PM Thursday for a design review`
* `Check my Friday schedule and email it to john@example.com`

The orchestrator decides which sub-agent to call, chains them when a request spans both domains, and confirms before anything destructive.

## Notes

* Only the orchestrator has memory — sub-agents are stateless and receive a fully-resolved prompt on each call.
* All calendar operations target the `primary` calendar.
* Retrieval is capped at 10 messages / 10 events per call.

## License

MIT
