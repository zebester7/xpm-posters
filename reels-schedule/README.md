# XPM Tutors – Subject reels posting schedule (Instagram + Facebook)

60 educational reels (Chemistry, Physics, Maths; IGCSE, O Level, A Level), 20 days from 23 Nov to 12 Dec 2026.
Three reels a day, one per subject, Pakistan time (Asia/Karachi):
**Chemistry 4:37 PM**, **Physics 6:37 PM**, **Maths 8:37 PM** (UAE: 3:37 / 5:37 / 7:37 PM; UK: 11:37 AM / 1:37 / 3:37 PM).
Day N posts CN, PN and MN.

Each reel goes to both:
- Instagram Business account **XPM Tutors** (id 17841480452221517), as a Reel shared to the feed.
- Facebook Page **XPM** (id 937535929440088), as a Page video.

## Files

- `../reels/<ID>.mp4`: the videos (1080×1920, 65–88 s).
- `schedule.csv`: one row per reel: `id, date_pkt, time_pkt, time_utc, type, title, caption_facebook, caption_instagram, media_url, media_url_alt`.
  Captions are final: no emojis, with WhatsApp +92 300 9508592 and https://xpmtutors.com. The Instagram caption starts with the title line.
- `posted.csv`: log of what has been published: `id, platform, time_utc, result`.
- `media_url` is the GitHub Pages link (served as video/mp4); `media_url_alt` is the raw GitHub link to the same file.

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
