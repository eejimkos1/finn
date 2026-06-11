# Finn — English-to-Finnish Speech Translator

## Summary

A single-file mobile web app that listens to English speech, translates it to Finnish, and speaks the Finnish translation aloud. Deployed to GitHub Pages.

## User Flow

1. User opens the page on their phone
2. Sees one big microphone button centered on screen
3. Taps the button → browser requests mic permission (first time only)
4. User speaks in English
5. Speech recognition captures the text
6. App calls MyMemory API to translate English → Finnish
7. App uses browser SpeechSynthesis to speak the Finnish translation aloud
8. Button returns to idle state, ready for next tap

## Architecture

Single `index.html` file. No build tools, no dependencies, no framework. Static hosting on GitHub Pages (`main` branch).

## Tech Stack

| Concern | Solution |
|---------|----------|
| Speech-to-text | `webkitSpeechRecognition` / `SpeechRecognition` API, lang: `en-US` |
| Translation | MyMemory API: `https://api.mymemory.translated.net/get?q=TEXT&langpair=en|fi` |
| Text-to-speech | `SpeechSynthesisUtterance`, lang: `fi-FI` |
| Hosting | GitHub Pages (static, `main` branch) |

## UI Design

- Full-screen mobile viewport (`<meta name="viewport">`)
- One large circular button centered vertically and horizontally
- Microphone icon inside the button (inline SVG, no external assets)
- Button states communicated via color:
  - **Idle:** neutral color (e.g., dark/blue)
  - **Listening:** pulsing red
  - **Translating/Speaking:** brief green flash, then back to idle
- No text displayed on screen — audio-only output
- Background: simple solid color

## Translation API

- Endpoint: `https://api.mymemory.translated.net/get?q={text}&langpair=en|fi`
- Free tier: 5000 words/day, no API key required
- Returns JSON with `responseData.translatedText`
- No authentication needed — works from browser fetch

## Error Handling

- **Mic permission denied:** button flashes orange briefly
- **Translation API fails:** silent fail, button returns to idle (user can retry)
- **Browser doesn't support Speech API:** button is disabled on page load
- **No Finnish voice available:** falls back to default voice with `fi-FI` lang tag (browser will use closest match)

## Browser Compatibility

- Chrome Android: full support (Speech Recognition + Synthesis)
- Safari iOS: Speech Recognition requires user gesture (tap satisfies this), Synthesis supported
- Firefox: SpeechRecognition not supported — button will be disabled
- Target: Chrome on Android and Safari on iOS (covers vast majority of mobile users)

## Deployment

- Single `index.html` in repo root
- Enable GitHub Pages on `main` branch, root (`/`) as source
- No build step required

## Constraints

- No API keys needed
- No backend
- No npm/build tooling
- Must work on mobile browsers (Chrome Android, Safari iOS)
- 5000 words/day translation limit (MyMemory free tier)
