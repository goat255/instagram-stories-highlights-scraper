# AGENTS.md: Instagram Stories & Highlights Scraper

Guide for AI agents (Claude, Codex, any MCP or HTTP client) that need Instagram stories or highlights.

## When to use it

Use this actor when a task needs the current stories or the highlight reels of one or more public Instagram profiles: archiving stories, monitoring brands, researching content, vetting creators. It does not post, follow or log in, and it cannot read private accounts.

## Call it

**MCP (recommended):** add the Apify MCP server with this actor enabled:

```json
{
  "mcpServers": {
    "apify": {
      "url": "https://mcp.apify.com/?actors=goat255/instagram-stories-highlights-scraper",
      "headers": { "Authorization": "Bearer YOUR_APIFY_TOKEN" }
    }
  }
}
```

**HTTP:** one synchronous call returns the rows:

```
POST https://api.apify.com/v2/acts/goat255~instagram-stories-highlights-scraper/run-sync-get-dataset-items?token=YOUR_APIFY_TOKEN
Content-Type: application/json

{"usernames": ["example_user"]}
```

## Input

| Field | Type | Default | Meaning |
|---|---|---|---|
| `usernames` | array of strings | required | Usernames or profile URLs |
| `includeStories` | boolean | `true` | Return the current 24-hour stories |
| `includeHighlights` | boolean | `true` | Return the highlight reels |
| `expandHighlightItems` | boolean | `true` | Return every item inside each highlight |
| `maxHighlightsPerUser` | integer | `20` | Cap on highlights per profile |
| `compactOutput` | boolean | `false` | Drop empty story and highlight fields |

## Output: one row per username

| Field | Meaning |
|---|---|
| `status` | `ok`, `no_active_stories`, `private`, `not_found` (permanent) or `fetch_failed` (retryable) |
| `isAccessible` | `true` when public media could be read |
| `errorMessage` | Human-readable reason when `isAccessible` is `false`, else `null` |
| `storyCount`, `stories` | The live story items with media URLs, `takenAt` and `expiringAt` (Unix seconds) |
| `highlightCount`, `highlights` | Highlight reels, each with `title`, `mediaCount`, `coverImageUrl` and `items` |
| `highlightsFetchFailed` | `true` when highlights could not be read this run; retry later instead of treating the list as empty |

## Tips for agents

- Batch many usernames into one run; it is cheaper and faster than one run per name.
- Story media URLs expire. Download the media soon after the run if you need to keep it.
- Retry rows with `status` `fetch_failed` a little later; they are not charged beyond the start fee.
- A row with `highlightsFetchFailed: true` still carries its stories; send the handle again later for its highlights.
- The last dataset item is a run summary tagged `recordType: "runSummary"`; filter it out of per-profile rows.

Store page: https://apify.com/goat255/instagram-stories-highlights-scraper
