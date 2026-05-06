# question.md

## Final sharpened question

In my Week 10 conversion engine, I describe the system as an "agent that uses tools" (e.g., Cal.com booking, HubSpot CRM updates), but `agent/integrations/openrouter_llm.py` sends vanilla chat completions with no `tools` parameter — the "tool choice" is deterministic scaffolding in `lead_orchestrator.py` (keyword matching + regex). My τ²-Bench Week 11 evaluation shows `finish_reason: "tool_calls"` in traces, which made me realize I have never actually seen how provider-native tool calling works.

What is the exact mechanism by which an LLM emits a tool call when the API request includes a `tools=[...]` schema? Specifically: does the model generate a special token (or token sequence) that the provider intercepts, or does it generate JSON text that the provider parses post-hoc? And how does this mechanism differ — in parsing reliability, latency, and the model's ability to refuse or abstain — from my current approach of prompting for JSON and scraping it with `_safe_parse_json`?

## Connection to existing artifact

- **Artifact:** `week_10_data/agent/integrations/openrouter_llm.py` (lines 45–70, `chat_completion` method)
- **Current language:** The payload contains only `"model"`, `"messages"`, `"temperature"`. No `tools`, no `tool_choice`, no `response_format`.
- **What I cannot defend:** If asked "how does the model choose which tool to call?" I would have to admit it doesn't — Python substring matching in `lead_orchestrator.py` (line 371, `_booking_intent`) decides, then calls the LLM for arguments.
- **What changes if gap closes:** I will rewrite the agent architecture section of my Week 10 blog post to distinguish "orchestrated workflow with LLM classifiers" from "agent with model-visible tool schema," and I will add a concrete comparison showing the exact OpenRouter request/response payload for a real `tools=[...]` call versus my current payload.

## Why this gap matters

- **To my work:** I am misrepresenting my system's architecture in portfolio language. Closing this gap lets me describe it accurately — or justify why deterministic scaffolding was the right choice.
- **To other FDEs:** Every production agent deployment faces the "when do I use real tool calling versus orchestration?" decision. Most early-stage systems look like mine but claim to be "agentic." Understanding the actual mechanism prevents architectural over-engineering or under-engineering.

