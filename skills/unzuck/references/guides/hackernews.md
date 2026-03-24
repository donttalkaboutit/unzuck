# Hacker News Scanning Guide

**URL**: `https://news.ycombinator.com`

No login needed. Structured layout, every story has a direct URL, rich metadata (points, comments, author, time). The most reliable platform for content extraction.

## Scanning

1. Navigate to the front page
2. Use `read_page` with `depth: 5` to extract all 30 stories with their `href` attributes, points, author, comment count, and time — one call gets everything
3. The front page has 30 items — one page is usually enough

## Getting the direct URL

Story title links are already the external article URLs (the `href` on the story title `<a>` element). For Ask HN / Show HN / HN discussions, the URL is `https://news.ycombinator.com/item?id=XXXXX`.

## Enriching descriptions

For items with 100+ points, the title alone is insufficient. Read the actual article:

1. Try `WebFetch` on the external article URL first (fastest)
2. If WebFetch fails: navigate to the URL in your tab, wait 2-3 seconds, use `get_page_text`
3. Write a 3-5 sentence description based on what you actually read — include specific claims, data points, names
4. Navigate back to HN (`navigate` with `url: "back"`) to continue

## Thumbnails

HN has no built-in thumbnails. For the top ~5 items by points, you can take a screenshot of the article page. For remaining items, use a platform placeholder.