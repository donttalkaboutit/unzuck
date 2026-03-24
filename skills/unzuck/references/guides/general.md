# General Scanning Rules

- Wait for pages to fully load before extracting content
- Use `get_page_text` for text-heavy pages, `read_page` for structured link/element extraction
- Respect rate limits — don't scroll excessively or make rapid requests
- Every item needs a direct permalink URL. If you can't get one, exclude the item.
- **Extract full content descriptions** — not just the first line. Expand "See more" / "...more" links, read full visible text, and capture enough context to write a meaningful 3-5 sentence summary
- **For reading article content**: Try `WebFetch` first (fastest). If it fails, fall back to browser navigation + `get_page_text`.

## Description Quality

Every item description must be specific and informative. Vague descriptions are useless.

Each description should contain:
- **What**: What is the content about? Be specific — names, products, technologies, numbers
- **Why it matters**: Why would someone care?
- **Key details**: Specific claims, data points, or insights

❌ BAD: "A post about AI and coding"
✅ GOOD: "Catherine Wu from Anthropic describes how PM roles are evolving at AI companies: shorter sprint cycles because model capabilities improve weekly, PMs building demos instead of writing docs, and the PM/designer/engineer roles converging into a single 'builder' archetype."

## Expanding Truncated Content

Most platforms truncate post text with "...more", "See more", or "Show more" buttons. Always expand before extracting:

1. **Via JavaScript** (preferred): Use `javascript_tool` to find and click expand buttons
2. **Via coordinate clicking**: If JS click doesn't work, take a screenshot, locate the "more" link visually, and click with `computer` tool
3. **Via get_page_text**: After expanding, use `get_page_text` to capture the full visible text

## Return Format

Return a JSON array with this structure per item:

```json
{
  "platform": "...",
  "title": "Full title of the content",
  "author": "Author name",
  "description": "3-5 sentences minimum with specific data points, names, claims, and context",
  "permalink": "https://...",
  "screenshotImageId": "ss_xxx or null",
  "mediaType": "image|video|embed|none",
  "postDate": "2h ago",
  "readTime": "12 min",
  "engagement": { "views": "120K", "likes": "4.2K", "comments": "312" },
  "tags": ["AI", "coding"],
  "videoId": null,
  "postId": null,
  "tweetId": null
}
```

## Error Handling

- **Login required**: Skip platform
- **Rate limited**: Wait 30 seconds, try once more, then skip
- **Page not loading**: Refresh once, then skip
- **Layout changed**: Fall back to `get_page_text` for raw text
- **CAPTCHA**: Report to user
- **JavaScript tool blocked**: Simplify script to avoid returning URL parameters, or use `read_page` as fallback