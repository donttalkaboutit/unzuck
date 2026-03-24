# YouTube Transcript Extraction Guide

This guide covers extracting the full spoken content from YouTube videos for deep-dive analysis. Transcripts are the single most valuable data source for YouTube deep dives — they contain everything the creator said, not just what's in the title and description.

## Method 1: Browser transcript panel (most reliable)

1. Navigate to the video page (`/watch?v=XXXXX`)
2. Wait for the page to fully load (2-3 seconds)
3. Click the "...more" button below the video description to expand it
4. Look for the **"Show transcript"** button in the expanded description area
5. Click it — a transcript panel opens on the right side
6. Use `get_page_text` to capture the full transcript text
7. The transcript includes timestamps like `0:00`, `1:23`, `12:45` next to each segment

### Extracting via JavaScript (faster)

```javascript
// Click "Show transcript" and extract
const moreBtn = document.querySelector('tp-yt-paper-button#expand');
if (moreBtn) moreBtn.click();
setTimeout(() => {
  const transcriptBtn = Array.from(document.querySelectorAll('button'))
    .find(b => b.textContent.includes('Show transcript'));
  if (transcriptBtn) transcriptBtn.click();
  setTimeout(() => {
    const segments = document.querySelectorAll('ytd-transcript-segment-renderer');
    const transcript = [];
    segments.forEach(seg => {
      const time = seg.querySelector('.segment-timestamp')?.textContent?.trim();
      const text = seg.querySelector('.segment-text')?.textContent?.trim();
      if (time && text) transcript.push({ time, text });
    });
    console.log(JSON.stringify(transcript));
  }, 1500);
}, 500);
```

### Fallback: coordinate-based clicking

If JavaScript selectors don't work (YouTube changes DOM frequently):
1. Take a screenshot of the video page
2. Visually locate "...more" below the description, click it
3. Take another screenshot
4. Visually locate "Show transcript", click it
5. Wait 1-2 seconds for the panel to load
6. Use `get_page_text` to capture everything

## Method 2: WebFetch on transcript services

Some services provide YouTube transcripts via URL. Try these in order:

1. **YouTube's own timedtext API** (may require cookies):
   ```
   https://www.youtube.com/api/timedtext?v={VIDEO_ID}&lang=en&fmt=srv3
   ```

2. **Third-party services** (use WebFetch):
   - Search for "{video title} transcript" to find blog posts or services that have extracted it

## Extracting chapters

YouTube chapters appear as segments in the progress bar and in the description. They provide a structured outline of the video.

### From the description
Chapters are listed as timestamps at the start of lines:
```
0:00 Introduction
2:15 Why caches exist
5:30 L1 vs L2 vs L3
12:00 Cache coherency
18:45 Conclusion
```

Parse these from the video description (already available via `get_page_text` on the video page).

### From the chapter markers
```javascript
const chapters = document.querySelectorAll('ytd-macro-markers-list-item-renderer');
const result = [];
chapters.forEach(ch => {
  const title = ch.querySelector('#details h4')?.textContent?.trim();
  const time = ch.querySelector('#time')?.textContent?.trim();
  if (title && time) result.push({ time, title });
});
JSON.stringify(result);
```

## Extracting top comments

Top comments often contain expert corrections, summaries, and additional context.

### Via browser
1. Scroll down past the video to load the comments section
2. Use `get_page_text` to capture the visible comments
3. The first 5-10 comments are usually the most valuable

### Via JavaScript
```javascript
const comments = document.querySelectorAll('ytd-comment-thread-renderer');
const result = [];
comments.forEach((c, i) => {
  if (i >= 5) return;
  const author = c.querySelector('#author-text')?.textContent?.trim();
  const text = c.querySelector('#content-text')?.textContent?.trim();
  const likes = c.querySelector('#vote-count-middle')?.textContent?.trim();
  if (text) result.push({ author, text, likes });
});
JSON.stringify(result);
```

## Extracting the pinned comment

The pinned comment (if present) is the first comment with a "Pinned" badge. It often contains the creator's own summary, corrections, or links.

```javascript
const pinned = document.querySelector('ytd-comment-thread-renderer #pinned-comment-badge');
if (pinned) {
  const thread = pinned.closest('ytd-comment-thread-renderer');
  const text = thread?.querySelector('#content-text')?.textContent?.trim();
  const author = thread?.querySelector('#author-text')?.textContent?.trim();
  JSON.stringify({ author, text, isPinned: true });
}
```

## Building keyPoints from transcript

Once you have the transcript, extract key points with timestamps:

1. **Identify topic shifts** — look for chapter boundaries or significant topic changes
2. **Extract specific claims** — numbers, statistics, conclusions
3. **Note key quotes** — memorable or surprising statements
4. **Map to timestamps** — each key point should link to where it was said

### Output format

```json
{
  "keyPoints": [
    { "time": "0:00", "seconds": 0, "point": "Introduction: why this topic matters" },
    { "time": "2:15", "seconds": 135, "point": "L1 cache sits on-core with ~1ns latency, only 32KB" },
    { "time": "5:30", "seconds": 330, "point": "Each cache level filters ~95% of accesses" },
    { "time": "12:00", "seconds": 720, "point": "Cache coherency protocols add 30-50% overhead" }
  ],
  "chapters": [
    { "time": "0:00", "title": "Introduction" },
    { "time": "2:15", "title": "Cache hierarchy explained" }
  ],
  "topComments": [
    { "author": "ExpertUser", "text": "Great video but note that...", "likes": "1.2K" }
  ]
}
```

## When to use each method

| Scenario | Method |
|----------|--------|
| Deep dive on a video (have browser tab) | Browser transcript panel |
| Deep dive without browser access | WebFetch on transcript service |
| Quick scan enrichment | Description + chapters only |
| High-engagement video (1K+ comments) | Also extract top comments |
