# XPM Tutors – Daily video series (Instagram + Facebook)

200 short videos, ONE a day, alternating two series (requested by Zubair on 23 Nov 2026):
- odd days: **SAT Desmos tips** #1–100 (`videos/S001.mp4` …)
- even days: **A Level Economics tough concepts** #1–100 (`videos/E001.mp4` …)

Day 1 was posted on 23 Nov 2026 at the time of the request. From day 2 (24 Nov 2026) every video goes out at
**9:27 PM Pakistan time** (16:27 UTC; 11:27 AM US Eastern; 4:27 PM UK). The last one (day 200) is on 10 Jun 2027.

Each video goes to both:
- Instagram Business account **XPM Tutors** (id 17841480452221517), as a Reel shared to the feed.
- Facebook Page **XPM** (id 937535929440088), as a Page video.

## Files

- `videos/<S|E>NNN.mp4`: the videos (1080×1920, 28–57 s).
- `schedule.csv`: one row per day: `id, day, series, date_pkt, time_pkt, time_utc, type, title, caption_facebook, caption_instagram, media_url, media_url_alt`.
  Captions are final: no emojis, with WhatsApp +92 300 9508592 and the website. The Instagram caption starts with the title line.
- `posted.csv`: log: `id, platform, time_utc, result`.
- `media_url` is the GitHub Pages link (video/mp4); `media_url_alt` is the raw GitHub link to the same file.

## Runbook (followed by the scheduled task)

1. `git pull` this repo.
2. Due rows: `time_utc` at or before now (UTC), and no line with result `ok`, `hold` or `skipped-late` in `posted.csv` for that id + platform.
   A row more than 6 hours past its `time_utc` is not posted: log it as `skipped-late`.
3. For each due row:
   - **Instagram (container flow; never use Zapier's "Publish Video", it times out before processing ends):**
     a. Duplicate check: Instagram for Business › New Media Posted (instagramPageId 17841480452221517). If a recent post's caption starts with this row's `title`, it is already posted: log `ok` with that id and skip.
     b. Create one container: Make API Mutating Request, POST `https://graph.facebook.com/v21.0/17841480452221517/media`, fail_on_errors "false", querystring `media_type=REELS`, `share_to_feed=true`, `video_url=<media_url>`, `caption=<caption_instagram>`.
     c. Every 30 s (`sleep 30`), Make API GET Request `https://graph.facebook.com/v21.0/<container id>` with `fields=status_code,status`, up to 10 minutes. ERROR/EXPIRED or still IN_PROGRESS after 10 minutes → do not publish; log the error. If the error looks like a media fetch failure, you may create ONE new container with `media_url_alt`.
     d. When FINISHED, publish once: POST `https://graph.facebook.com/v21.0/17841480452221517/media_publish` with `creation_id=<container id>`. If that errors, re-run the duplicate check before trying `media_publish` once more with the same container.
   - **Facebook:** Facebook Pages › Create Page Video (`page` = 937535929440088, `source` = media_url, `title` = title, `description` = caption_facebook). Retry once on failure, using `media_url_alt` if the video could not be fetched.
4. Append one line per platform to `posted.csv` (`ok` or `error: <short message>`), commit as "XPM Tutors <xpm-tutors@users.noreply.github.com>" and push. Never post the same id to the same platform twice.

To pause, pause the scheduled task, or add `id,instagram,manual,hold` / `id,facebook,manual,hold` lines to `posted.csv`.
