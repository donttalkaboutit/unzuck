# Reddit Scanning Guide

**URL**: `https://www.reddit.com`

No login required for browsing, but logged-in users see their personalized home feed with subscribed subreddits. If not logged in, scan `/r/popular` or `/r/all` instead.

## Scanning

1. Navigate to the home feed (logged in) or `https://www.reddit.com/r/popular/` (logged out)
2. Take a screenshot to confirm the page loaded and check login state
3. Scroll 2-3 times to load more content
4. Use `get_page_text` to extract post titles, subreddits, authors, scores, comment counts, and timestamps
5. Use `read_page` to extract permalink `href` attributes

## Content extraction

Post title, subreddit, author, upvotes, comment count, timestamp, flair/tags, linked URL (if link post).

## Post types

- **Link posts**: Title links to an external URL. The permalink is for the Reddit discussion.
- **Self/text posts**: Content is on Reddit itself. The permalink is the content URL.
- **Image/video posts**: Media hosted on Reddit or imgur. Note the media type.
- **Crosspost**: Shared from another subreddit. Use the original post's content.

## Getting the direct URL

Each post has a permalink: `reddit.com/r/{subreddit}/comments/{id}/{slug}/`. To find it:
1. Use `read_page` and look for `href` attributes containing `/comments/` — these are post permalinks
2. For link posts, also capture the external URL (the domain shown next to the title)
3. Always prepend `https://www.` if the href is relative

**Never** use `reddit.com/r/{subreddit}/` or `reddit.com/` as a post URL.

## Enriching descriptions

Reddit posts often have rich discussion:
1. For link posts with 100+ upvotes: try `WebFetch` on the external URL to read the actual article
2. For self/text posts: the post body often contains the full content — expand truncated text
3. For popular posts (500+ upvotes): read the top 3-5 comments — Reddit comments frequently contain expert analysis, corrections, and additional context that enriches the summary
4. Click "View more" or scroll to load full post text if truncated

## Expanding truncated posts

Reddit truncates long self posts. Use JavaScript:
```javascript
document.querySelectorAll('[id*="read-more"]').forEach(b => b.click());
```
Wait 1 second, then re-extract with `get_page_text`.

## Subreddit signals for bootstrapping

The subreddits appearing in the user's feed reveal interests:
- Tech subreddits: r/programming, r/machinelearning, r/webdev, r/rust, r/golang
- Business: r/startups, r/entrepreneur, r/SaaS
- Location: r/london, r/unitedkingdom
- Niche interests: r/chess, r/formula1, r/electronicmusic

Extract the subreddit names for the bootstrap interest profile.

## Realistic expectations

10-15 items with valid permalinks per scroll session. Reddit's DOM is more complex than HN but content is reliably extractable.
