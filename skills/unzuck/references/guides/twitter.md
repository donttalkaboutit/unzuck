# Twitter/X Scanning Guide

**URL**: `https://x.com/home`

Requires login. If not logged in, skip.

## Scanning

1. Navigate to the home timeline
2. Check which tab is active: "For you" (algorithmic) vs "Following" (chronological). "Following" gives more consistent, less noisy content.
3. Scroll through the feed 3-4 times
4. Use `read_page` on the timeline to extract tweet data
5. Also extract the "Today's News" and "What's happening" sections from the right sidebar — these are trending topics

## Content extraction

Tweet text, author name, handle, likes, retweets, replies, views, timestamp, embedded links.

## Getting the direct URL

Each tweet has a permalink: `x.com/{handle}/status/{id}`. To find it:
1. **Best method**: Use `read_page` and look for `href` attributes containing `/status/` — timestamp links (e.g., "3h", "7h") point to the permalink
2. **Fallback**: Click into a tweet and read the tab URL
3. The analytics link (href ending in `/analytics`) also contains the status ID — strip `/analytics`

**Never** use `x.com`, `x.com/home`, or `x.com/{handle}` as a tweet URL.

## Tweet ID for embeds

Extract from the permalink URL (`/status/{TWEET_ID}`). Used for embed iframes: `https://platform.twitter.com/embed/Tweet.html?id={TWEET_ID}&theme=dark`

## Enriching descriptions

Tweets are often short but context-rich:
- Capture the FULL tweet text (expand "Show more" for long tweets)
- For threads, read ALL tweets in the thread
- Note quote tweets — the quoted content provides additional context
- If the tweet links to an article, note the article title/domain
- For tweets linking to articles, try reading the linked article via `WebFetch` or `get_page_text` — the tweet itself may be just a teaser

## Thumbnails

Take a screenshot of the top 3-5 tweets for dashboard visuals. Do NOT rely on `pbs.twimg.com` media URLs.