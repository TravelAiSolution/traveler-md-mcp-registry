# Traveler.md MCP Server

Traveler.md gives an AI assistant a traveler's own travel memory. It holds a durable profile of how the traveler likes to travel and one plan per trip. The traveler owns the data, authorizes access once, and can revoke it at any time.

This repository describes the hosted MCP (Model Context Protocol) server at `https://mcp.traveler.md/mcp`. The server source is not published here. The full documentation is at <https://docs.traveler.md/mcp>.

## What the server does

The server stores two kinds of document and returns them on request.

- **traveler.md** is the profile. It holds stable preferences: seat choice, hotel style, budget, dining style, travel companions. One per traveler.
- **trip.md** is one trip. It holds destinations, dates, itinerary, bookings and notes. Many per traveler.

An assistant reads these before it plans or recommends, so the traveler does not repeat themselves. An assistant writes back what it learns, so the next conversation starts with the same knowledge.

Scope: travel memory only. The server has no inventory, no prices, no availability and no recommendation engine. Your assistant does the recommending. The profile changes what the recommendation is based on.

## Tools

| Tool             | Purpose                                                                  |
| ---------------- | ------------------------------------------------------------------------ |
| `read_profile`   | Return the profile, optionally filtered to named sections                |
| `create_profile` | Create the profile on first use                                          |
| `update_profile` | Record a lasting preference in one or more profile sections              |
| `list_trips`     | Find trips by title, content, status or start date                       |
| `read_trip`      | Return one trip with its sections and dated events                       |
| `create_trip`    | Open a trip with a title and a status, and optionally its booked events  |
| `update_trip`    | Change a trip's status, dates, sections or dated events                  |
| `archive_trip`   | Move a trip to the archive in the traveler's account. Nothing is deleted |

Every document is a set of named sections. Each section is a list of short sentences. The profile has 20 sections, for example `flight_preferences`, `accommodation`, `food_dining`, `loyalty_programs` and `home_and_entry`. The `home_and_entry` section holds the home city, the countries of citizenship and the names of trusted traveler programs. A trip keeps its dated bookings as `events`, separate from its prose sections. The server also publishes the section reference for both documents as MCP resources.

Every update requires the `expected_version_hash` from the most recent read or write of that record. Two assistants that write the same record cannot overwrite each other silently.

## Configuration

The server uses streamable HTTP and OAuth 2.1. There are no API keys and no environment variables. On first use the client opens a browser window. The traveler signs in to Traveler.md and approves the connection.

### VS Code and GitHub Copilot

One click:

[![Add to VS Code](https://img.shields.io/badge/Add%20to-VS%20Code-0098FF?style=for-the-badge)](https://vscode.dev/redirect/mcp/install?name=travelermd&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22https%3A%2F%2Fmcp.traveler.md%2Fmcp%22%7D)

Or add this to `.vscode/mcp.json` in a workspace, or to your user `mcp.json`:

```json
{
  "servers": {
    "travelermd": {
      "type": "http",
      "url": "https://mcp.traveler.md/mcp"
    }
  }
}
```

### Other clients

```bash
claude mcp add --transport http travelermd https://mcp.traveler.md/mcp   # Claude Code
codex mcp add travelermd --url https://mcp.traveler.md/mcp               # Codex
```

Any client that speaks remote MCP takes the URL directly:

```json
{
  "mcpServers": {
    "travelermd": {
      "url": "https://mcp.traveler.md/mcp"
    }
  }
}
```

Per-client guides are at <https://docs.traveler.md/guides>. There are guides for ChatGPT, Claude, Claude Code, Codex, Cursor, Gemini, Grok (including Grok Bot and Grok Build), Hermes, Instinct, Mistral Vibe, Muse, Notion, OpenClaw, Perplexity and VS Code. A general guide covers other clients.

## Usage

Once connected, ask about travel in plain language. The assistant calls the tools.

| You say                                      | The assistant calls                |
| -------------------------------------------- | ---------------------------------- |
| "Where should we stay in Lisbon?"            | `read_profile`, then recommends    |
| "What is my next trip?"                      | `list_trips` sorted by start date  |
| "What is the plan for Kyoto?"                | `list_trips`, then `read_trip`     |
| "I always want an aisle seat"                | `update_profile`                   |
| "We booked the ryokan"                       | `update_trip`                      |
| "I would love to see Patagonia one day"      | `create_trip` with status Dreaming |

Assistants often answer a travel question from the conversation alone and leave the profile unread. Add a standing instruction to your assistant's instruction file so it reads the profile first:

```
Before you recommend, plan, shortlist or book anything travel-related, call read_profile on the travelermd server. When a specific trip is in play, find it with list_trips and open it with read_trip.
```

## Skills and agent plugin

Agent Plugin version 1.4.0 packages this server together with skills for onboarding, trip interviews, trip organization and post-trip reviews. It is at <https://github.com/TravelAiSolution/traveler-md-agent-plugin>. The skills teach an assistant the section names, the sentence caps and the read-before-write loop.

## Data and privacy

The traveler owns every record. Access is granted per client through OAuth and revoked from the Traveler.md account page. The privacy policy is at <https://traveler.md/privacy>.

### What the server will not store

Traveler.md keeps four kinds of data out of the record: passport and ID numbers, known traveler numbers, payment card details and health details. The server rejects a write that contains a payment card number, a passport number, a known traveler number, a social security number or a door or access code. The error names the section and the type of data. The tool descriptions also tell the assistant not to record health, accessibility or biometric details, passwords or auth codes. A sentence can name a passport country, a card product such as Amex Platinum, or a trusted traveler program such as Global Entry.

## Support

Open an issue in this repository for questions about the server or its tools. For account or security matters, write to support@traveler.md.

## License

The README and the files in this repository are licensed under the MIT License. The hosted server is a Traveler.md service. Traveler.md and TravelAI are trademarks of TravelAI; this license grants no rights to use them.
