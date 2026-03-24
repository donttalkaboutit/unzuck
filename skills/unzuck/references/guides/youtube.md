# YouTube Scanning Guide

**URL**: `https://www.youtube.com`

## Scanning

1. Navigate to YouTube home (or `/feed/subscriptions` for subscription-only content)
2. Take a screenshot to confirm logged in and see layout
3. **Extract the topic filter chips** from the top bar (e.g., "Music", "AI", "Gaming") — these are valuable interest signals for bootstrapping
4. Use `read_page` with `filter: interactive` to extract video title links with `href` attributes
5. Scroll down 2-3 times and extract again to get more content
6. Skip Shorts (URLs containing `/shorts/`) and "YouTube Playables" sections

## Content extraction

Video title, channel name, view count, upload date, duration, URL.

## Thumbnail URL

Construct from the video ID (permanent, never expire):
- `https://i.ytimg.com/vi/{VIDEO_ID}/hqdefault.jpg` (480x360)
- `https://i.ytimg.com/vi/{VIDEO_ID}/mqdefault.jpg` (320x180)

Always capture the `videoId` separately — it's needed for the inline iframe player in the dashboard.

## Getting the direct URL

Video title links have `href="/watch?v=XXXXX"`. Prepend `https://www.youtube.com`. Always present in `read_page` output. Never construct search URLs.

## Enriching descriptions

Title alone is NOT enough. For the top 3-5 videos:
- Navigate to the video page (`/watch?v=XXXXX`)
- Use `get_page_text` to read the FULL description (often contains outlines, timestamps, key points)
- Note channel name, view count, duration
- Navigate back to the feed after each

## What to watch for

YouTube mixes Shorts, Playables, and regular videos. Regular videos have durations like "8:32" or "1:23:44". Shorts are sub-60 seconds. Focus on regular videos.