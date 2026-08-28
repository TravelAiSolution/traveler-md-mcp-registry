# Traveler.md MCP Server

Traveler.md gives an AI assistant a traveler's own travel memory. It holds a durable profile of how the traveler likes to travel and one plan per trip. The traveler owns the data, authorizes access once, and can revoke it at any time.

This repository describes the hosted MCP (Model Context Protocol) server at `https://mcp.traveler.md/mcp`. The server source is not published here. The full documentation is at <https://docs.traveler.md/mcp>.

## What the server does

The server stores two kinds of document and returns them on request.

- **traveler.md** is the profile. It holds stable preferences: seat choice, hotel style, budget, dietary preferences, travel companions. One per traveler.
- **trip.md** is one trip. It holds destinations, dates, itinerary, bookings and notes. Many per traveler.

An assistant reads these before it plans or recommends, so the traveler does not repeat themselves. An assistant writes back what it learns, so the next conversation starts with the same knowledge.

Scope: travel memory only. The server has no inventory, no prices, no availability and no recommendation engine. Your assistant does the recommending. The profile changes what the recommendation is based on.

## Tools

| Tool             | Purpose                                                     |
| ---------------- | ----------------------------------------------------------- |
| `read_profile`   | Return the profile, optionally filtered to named sections   |
| `create_profile` | Create the profile on first use                             |
| `update_profile` | Change one or more profile sections                         |
| `list_trips`     | Page or search trips by title, content, status or date      |
| `read_trip`      | Return one trip                                             |
| `create_trip`    | Create a trip with a title and a status                     |
| `update_trip`    | Change a trip's status, dates or sections                   |
| `archive_trip`   | File a trip away                                            |

Every document is a set of named sections. Each section is a list of short sentences. Updates require the `version_hash` from the most recent read, so two assistants writing the same record cannot overwrite each other silently.

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

Per-client guides for Claude, Cursor, ChatGPT, Gemini and others are at <https://docs.traveler.md/mcp>.

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

An Agent Plugin that packages this server together with skills for onboarding, trip interviews, trip organization and post-trip reviews is at <https://github.com/TravelAiSolution/traveler-md-agent-plugin>. The skills teach an assistant the section names, the sentence caps and the read-before-write loop.

## Data and privacy

The traveler owns every record. Access is granted per client through OAuth and revoked from the Traveler.md account page. The server rejects medical detail in every section, including allergies and dietary restrictions written as health information. The privacy policy is at <https://traveler.md/privacy>.

## Support

Open an issue in this repository for questions about the server or its tools. For account or security matters, write to support@traveler.md.

## License

The README and the files in this repository are licensed under the MIT License. The hosted server is a Traveler.md service. Traveler.md and TravelAI are trademarks of TravelAI; this license grants no rights to use them.
