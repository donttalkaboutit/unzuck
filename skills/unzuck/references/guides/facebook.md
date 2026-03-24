# Facebook Scanning Guide

**URL**: `https://www.facebook.com`

Requires login. Facebook is **opt-in** (default maxItems: 0) because of low signal-to-noise ratio. Only scan if the user has set maxItems > 0 in their profile or explicitly requests it.

## CRITICAL — Slow Loading

Facebook's feed takes 4-6 seconds to load. You'll see skeleton placeholders (gray pulsing rectangles). This is NORMAL — the user IS logged in. **Wait at least 5 seconds** before extracting. Do NOT mistake this for "not logged in."

## Mandatory Content Filtering

Skip ALL of the following:
- Profile picture/cover photo updates
- Birthday notifications and wishes
- Friend activity ("is now friends with...")
- Life events from non-public-figures
- Check-ins with no meaningful text
- Posts with no text body (pure photos with caption < 30 chars)
- Game/app activity
- Sponsored content and ads

**ONLY keep** items with: shared articles/links with external URLs, page posts with 50+ chars, posts from public figures with meaningful commentary, or substantive group discussions.

## Scanning

1. Navigate to the home feed
2. **Wait 5 seconds** for content to load
3. Take a screenshot to confirm feed is visible
4. Scroll 1-2 times, use `get_page_text` after each scroll
5. Use `read_page` to find links and buttons for permalink extraction
6. Focus on shared articles/links and page posts with substantial text

## Expanding truncated posts

Click "See more" using coordinate-based clicking (JS `.click()` may not work on Facebook). After expanding, use `get_page_text`.

## Getting the direct URL — timestamp-click method

Facebook obfuscates timestamps in the DOM but renders them normally on screen.

1. Visually locate the timestamp text (e.g., "March 19 at 10:47 AM", "3d", "2h") below the author name
2. Click on it using coordinate-based clicking (`left_click` with `coordinate`) — NOT JS `.click()`
3. Read the resulting tab URL — format: `https://www.facebook.com/{username}/posts/pfbid{hash}`
4. Navigate back to the feed to continue

**Alternative**: Links containing `/posts/`, `/videos/`, or `/photo/?fbid=` in `read_page` output are also valid permalinks.

For shared articles: also extract the external URL from the link preview card.

## Realistic expectations

3-5 items with valid permalinks. Budget ~10 seconds per post for permalink extraction.