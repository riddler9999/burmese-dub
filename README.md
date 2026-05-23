# Burmese Dub — Web Interface

🎙️ Submit a video URL (YouTube · TikTok · Xiaohongshu) and get a Burmese-dubbed video back. Auto-generates SEO metadata too.

**🔗 Live:** https://riddler9999.github.io/burmese-dub/

## How it works

1. Paste a video URL
2. Pick target language (Burmese / English / Japanese / Korean)
3. Hit `Start Dubbing`
4. Backend (n8n + Whisper + Gemini + edge-tts + FFmpeg on Hostinger VPS):
   - Downloads video via yt-dlp
   - Transcribes with Whisper (preserves timestamps)
   - Translates segment-by-segment with Gemini (character-budget rule)
   - Generates voiceover with edge-tts, chunk-split + silence-padding
   - Applies global atempo correction to lock VO duration to video duration
   - Mixes VO into video as new audio track
5. Watch / Download final video right in the page

## Architecture

```
[GitHub Pages]  ──POST──▶  [n8n Webhook]
                              │
                              ▼
                  ┌───────────────────────┐
                  │ Hostinger VPS (KVM 2) │
                  │  - yt-dlp             │
                  │  - Whisper (CPU)      │
                  │  - Gemini API         │
                  │  - edge-tts           │
                  │  - FFmpeg             │
                  └───────────────────────┘
                              │
[GitHub Pages]  ◀──poll────  [n8n status webhook]
[GitHub Pages]  ◀──video───  [n8n video serve webhook]
```

## Backend webhooks

| Endpoint | Method | Use |
|---|---|---|
| `/webhook/burmese-dub` | POST | Submit URL, returns `job_id` |
| `/webhook/burmese-dub-status?job_id=...` | GET | Poll job status, returns SEO metadata when done |
| `/webhook/burmese-dub-video?job_id=...` | GET | Serve final MP4 binary |

## License

MIT
