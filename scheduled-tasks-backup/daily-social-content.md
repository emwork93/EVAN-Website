---
name: daily-social-content
description: Daily: pull best-performing footage from Instagram/TikTok/YouTube (or fresh local footage) on rotation, edit with TikTok-style on-screen text, write one shared caption, check for upcoming gigs, and recommend best posting time for manual Metricool upload
---

You are running the daily social-content prep routine for a musician's project at "/Users/evanmacgregor/CLAUDE WEB BUILD" (a git repo). This runs once per day, unattended, with no memory of prior runs except state files you maintain yourself. Do the following, in order:

## 1. Build today's footage candidate pool
The user wants footage sourced from TWO places, mixed together:

**A. Fresh local raw footage** in "website info/" (.mov/.mp4, some iPhone HEIC/MOV with rotation metadata — ffmpeg 7.1 auto-applies displaymatrix rotation on decode, no manual transpose needed) that hasn't been used yet.

**B. The user's own past posts on Instagram, TikTok, and YouTube**, which can be reused/remixed. Pull these via the Metricool MCP tools (prefix `mcp__4b6ac9d3-99e0-4995-8ff0-974bcfa6dd82__`), brandId 6583715:
- Instagram: `getAnalyticsDataByMetrics` with metrics ["IGRE06","IGRE03","IGRE08","IGRE23"] (url, content/caption, engagement, views) over connector "reels", lookback ~180 days.
- TikTok: same tool with metrics ["TKPO03","TKPO05","TKPO07","TKPO9999"] (shareurl, description, views, engagement) over connector "posts".
- YouTube: same tool with metrics ["YTVV05","YTVV17","YTVV06"] (watchUrl, title, views) over connector "all videos". Do NOT use the deprecated YTVI* fields.

Rank each network's results by views/engagement descending — these are your "best performers."

## 2. Maintain a rotation/cooldown so you don't repeat clips too often
Maintain "exports/.clip_history.json" (create if missing): a list of {id (local filename or post url), source, lastUsedDate, timesUsed}. This is the user's explicit rule: reuse based on performance is fine, but "mix it up daily so it doesn't do the same videos frequently."

- Exclude anything used in the last 14 days.
- If unused fresh local footage exists, prefer it (guarantees variety automatically).
- Otherwise, from the eligible (not-recently-used) ranked social posts, pick randomly among the top 5 performers per network rather than always the single #1 — this keeps daily picks varied even when reusing top content.
- After picking, append/update the entry in the history file (today's date, increment timesUsed).

## 3. Get the actual video file for the chosen clip
- If it's local raw footage, use the file directly from "website info/".
- If it's a past Instagram/TikTok/YouTube post, you need to download the actual video from its URL. Check for `yt-dlp` first: try `which yt-dlp`, then `python3 -m yt_dlp --version`. If neither works, install it with `pip3 install --user yt-dlp`. Download the chosen post's URL to a staging path like "exports/.staging/<id>.mp4". (This is the user's own public content from their own accounts — it's fine to fetch it this way.)

## 4. Check for an upcoming gig announcement
Read "gigs.json" in the project root (array of {date, time, venue, venueLink, city, address, ticketLink, notes}). Compare each gig's date to today's real date (get it from the system, e.g. `date` in bash — do not assume).

- If a gig is within the next 7 days AND not yet marked announced in "exports/.announced_gigs.json" (create/maintain this manifest), that gig takes priority: build a gig-announcement post instead of/alongside the generic content post, with the ticket link included directly in the caption text (not just "link in bio"). For the gig post, prefer footage from the live/venue-style source if one is available in the candidate pool.
- If the gig is within 3 days, explicitly flag in your summary that this should be posted TODAY regardless of the "best time" analytics window — timeliness beats optimal timing for near-term event announcements.
- Mark the gig as announced in the manifest once you've prepared its post so it isn't repeated daily.

## 5. Edit the video
Use ffmpeg. Locate a working ffmpeg binary dynamically each run (do not hardcode a path from a prior session — it may not exist anymore): try `which ffmpeg`, and if that fails, try `python3 -c "import imageio_ffmpeg; print(imageio_ffmpeg.get_ffmpeg_exe())"` (installed via `pip install --user imageio-ffmpeg` in a prior session; reinstall if missing).

Trim to a clean, engaging ~15-25s segment (skim frames first to confirm there's no dead air / awkward cut). Scale to 1080x1920 (9:16).

**On-screen text style — hard rule the user set explicitly:** burned-in captions must look like popular native TikTok-style text — bold, rounded, casual sans-serif (look for a rounded/bold system font other than plain Arial Bold; check /System/Library/Fonts and /System/Library/Fonts/Supplemental for something like a rounded bold face), short punchy phrases, positioned like native in-app captions. Do NOT reuse the old look (Arial Bold + flat black boxcolor box) — the user explicitly said that looked "weird," not authentic. If you can't find a better font, use drawtext with NO flat box background (or a soft rounded semi-transparent pill via boxborderw + small alpha) rather than the old boxy look, and say so in your summary.

Output the finished file to "exports/" with a dated filename, e.g. "exports/post_2026-07-20.mp4" or "exports/gig_2026-07-20.mp4".

## 6. Write ONE caption for all platforms
The user wants the exact same video AND the exact same caption posted to Instagram, TikTok, and YouTube — do not write platform-specific variants. One caption, reused everywhere.

- For a gig-announcement post: include venue, date, time, and the raw ticket URL directly in the caption text.
- For a generic content post (including reused/remixed footage): casual, short, authentic caption + a handful of relevant hashtags. If it's a reused clip, feel free to reference that lightly ("bringing this one back") but don't overdo it.

## 7. Get current best-time-to-post data
Call `getBestTimeToPostByNetwork` for network "instagram" and "tiktok" (brandId 6583715, timezone America/New_York, a ~7-day forward window from today). Summarize the single best upcoming day+hour combo for each network.

## 8. Do NOT attempt to auto-publish or auto-schedule via Metricool
Known constraint: Metricool's `createScheduledPost` requires media to be a public URL or a Google-Drive-linked file, and this account cannot link Google Drive without a paid plan upgrade the user has declined. Do not call `createScheduledPost` — it will fail or silently misbehave. The user uploads manually into the Metricool app themselves.

## 9. Also enforce: Instagram = Trial Reels only
Never frame or describe the Instagram post as a regular Reel. Always call it / format it as an Instagram **Trial Reel**. Always pair it with posting the identical video+caption to TikTok (and YouTube where relevant) too.

## Final output
End with a concise summary for the user containing: which clip was used and where it came from (local/reused-from-network + how it performed originally, or "no eligible footage today" if truly nothing qualifies), the finished file path, the one shared caption text in full, whether a gig announcement was included (and its urgency), and the recommended best day/hour to post on Instagram and TikTok. Keep it short and copy-paste ready — the user reads this once a day and manually uploads.