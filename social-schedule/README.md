# XPM Tutors – SAT USA posting schedule (Instagram + Facebook)

25 days, 22 Nov – 16 Dec 2026. Three posts a day, US Eastern time:

| Slot | Time (ET) | Time (PT) | Content |
| --- | --- | --- | --- |
| post | 7:45 AM | 4:45 AM | Image post from `sat-content-kit/posts_jpg/` (posts 01–25), aimed at parents |
| math | 3:45 PM | 12:45 PM | SAT Math tip video from `sat/` (Math 01–25), after school on the East Coast |
| english | 7:45 PM | 4:45 PM | SAT English tip video from `sat/` (English 01–25), evening ET, after school on the West Coast |

Each item goes to both:
- Instagram Business account **XPM Tutors** (id 17841480452221517). Videos go out as Reels.
- Facebook Page **XPM** (id 937535929440088).

## Files

- `schedule.csv`: one row per item: `id, day, slot, type, time_et, time_utc, title, media_url, caption_instagram, caption_facebook, media_url_alt`.
- `posted.csv`: log of what has been published: `id, platform, posted_at_utc, result`.
- Media are served from this public repo. Videos use jsDelivr (`media_url`, served as video/mp4); `media_url_alt` is the raw GitHub link for the same file.

## Runbook (followed by the scheduled task)

1. `git pull` this repo.
2. Find rows in `schedule.csv` where `time_utc` is at or before now (UTC) **and** no `ok` line exists in `posted.csv` for that `id` + platform. Ignore rows more than 6 hours overdue; log them as `skipped-late` so they never post at the wrong time.
3. For each due row, post through Zapier:
   - **Instagram, video:** Instagram for Business › Publish Video (`video` = media_url, `caption` = caption_instagram, `instagramPageId` = 17841480452221517).
   - **Instagram, photo:** Instagram for Business › Publish Photo(s) (`media` = [media_url], `caption` = caption_instagram, same account).
   - **Facebook, video:** Facebook Pages › Create Page Video (`page` = 937535929440088, `source` = media_url, `title` = title, `description` = caption_facebook).
   - **Facebook, photo:** Facebook Pages › Create Page Photo (`page` = 937535929440088, `source` = [media_url], `message` = caption_facebook).
4. Append one line per platform to `posted.csv` (`ok` or `error: <message>`), commit and push. Never post the same id + platform twice. Retry a failed call once at most; for a failed media fetch, the retry uses `media_url_alt`.

To pause posting, pause the scheduled task, or add a line `id,all,manual,hold` to `posted.csv` for each item to hold.
