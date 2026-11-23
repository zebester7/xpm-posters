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

## Instagram stories (added 23 Nov 2026)

Five story cards a day from `sat-stories/` (rows with `type` = `story`, ids `S01-tip` … `S24-answer`):

| Time (ET) | Stories |
| --- | --- |
| 7:45 AM | SAT tip of the day, then a "Did you know?" fact |
| 3:45 PM | Quick SAT quiz question, then a "Why XPM Tutors" card |
| 7:45 PM | The quiz answer |

Stories go to Instagram only. Facebook gets them through Instagram's "Share your story to Facebook" setting, so log the Facebook line as `ok-via-instagram` and never post a story to Facebook directly (it would duplicate).

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
   - **Instagram, story:** Instagram for Business › Make API Mutating Request: `POST https://graph.facebook.com/v21.0/17841480452221517/media` with JSON body `{"media_type":"STORIES","image_url":"<media_url>"}`; then `GET https://graph.facebook.com/v21.0/<container id>?fields=status_code` until `FINISHED`; then `POST https://graph.facebook.com/v21.0/17841480452221517/media_publish?creation_id=<container id>`. Post a day's stories in schedule order (tip before fact, quiz before why).
   - **Instagram video fallback:** if Publish Video errors with "still processing", do not retry it blindly. First check `GET https://graph.facebook.com/v21.0/17841480452221517/media?fields=caption,timestamp&limit=5` for a post with the same caption; if none, use the same three-step method with `{"media_type":"REELS","share_to_feed":true,"video_url":"<media_url>","caption":"<caption_instagram>"}`.
4. Append one line per platform to `posted.csv` (`ok` or `error: <message>`), commit and push. Never post the same id + platform twice. Retry a failed call once at most; for a failed media fetch, the retry uses `media_url_alt`.

To pause posting, pause the scheduled task, or add a line `id,all,manual,hold` to `posted.csv` for each item to hold.
