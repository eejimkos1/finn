# Finn Speech Translator Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a single-file mobile web app that recognizes English speech, translates it to Finnish via MyMemory API, and speaks the Finnish aloud.

**Architecture:** One `index.html` with inline CSS and JS. No dependencies. Uses Web Speech API for recognition, fetch for translation, SpeechSynthesis for output. Deployed as a static file on GitHub Pages.

**Tech Stack:** HTML5, CSS3, vanilla JS, Web Speech API, MyMemory Translation API, GitHub Pages

---

## File Structure

- Create: `index.html` — the entire app (HTML structure, inline CSS, inline JS)

That's it. One file.

---

### Task 1: HTML Shell with Mobile Viewport

**Files:**
- Create: `index.html`

- [ ] **Step 1: Create the base HTML file**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>Finn</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            height: 100dvh;
            display: flex;
            align-items: center;
            justify-content: center;
            background: #1a1a2e;
            overflow: hidden;
            -webkit-tap-highlight-color: transparent;
        }

        #mic-btn {
            width: 140px;
            height: 140px;
            border-radius: 50%;
            border: none;
            background: #0f3460;
            cursor: pointer;
            display: flex;
            align-items: center;
            justify-content: center;
            transition: background 0.3s ease;
        }

        #mic-btn:disabled {
            background: #333;
            opacity: 0.5;
            cursor: not-allowed;
        }

        #mic-btn.listening {
            animation: pulse 1s infinite;
            background: #c0392b;
        }

        #mic-btn.success {
            background: #27ae60;
        }

        #mic-btn.error {
            background: #e67e22;
        }

        @keyframes pulse {
            0% { transform: scale(1); }
            50% { transform: scale(1.08); }
            100% { transform: scale(1); }
        }

        #mic-btn svg {
            width: 60px;
            height: 60px;
            fill: #ffffff;
        }
    </style>
</head>
<body>
    <button id="mic-btn" aria-label="Tap to speak">
        <svg viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
            <path d="M12 14c1.66 0 3-1.34 3-3V5c0-1.66-1.34-3-3-3S9 3.34 9 5v6c0 1.66 1.34 3 3 3z"/>
            <path d="M17 11c0 2.76-2.24 5-5 5s-5-2.24-5-5H5c0 3.53 2.61 6.43 6 6.92V21h2v-3.08c3.39-.49 6-3.39 6-6.92h-2z"/>
        </svg>
    </button>
    <script>
        // JS will go here in Task 2
    </script>
</body>
</html>
```

- [ ] **Step 2: Open in browser to verify layout**

Open `index.html` in a browser. You should see a dark background with a blue circular button centered on screen containing a white microphone icon.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add HTML shell with mobile viewport and mic button"
```

---

### Task 2: Speech Recognition

**Files:**
- Modify: `index.html` (replace the `<script>` section)

- [ ] **Step 1: Add speech recognition logic**

Replace the `<script>` block in `index.html` with:

```javascript
const btn = document.getElementById('mic-btn');

const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;

if (!SpeechRecognition) {
    btn.disabled = true;
} else {
    const recognition = new SpeechRecognition();
    recognition.lang = 'en-US';
    recognition.interimResults = false;
    recognition.maxAlternatives = 1;

    let listening = false;

    btn.addEventListener('click', () => {
        if (listening) return;
        listening = true;
        btn.classList.add('listening');
        recognition.start();
    });

    recognition.addEventListener('result', (event) => {
        const transcript = event.results[0][0].transcript;
        listening = false;
        btn.classList.remove('listening');
        translateAndSpeak(transcript);
    });

    recognition.addEventListener('error', () => {
        listening = false;
        btn.classList.remove('listening');
        btn.classList.add('error');
        setTimeout(() => btn.classList.remove('error'), 600);
    });

    recognition.addEventListener('end', () => {
        if (listening) {
            listening = false;
            btn.classList.remove('listening');
        }
    });
}

async function translateAndSpeak(text) {
    // Will be implemented in Task 3
}
```

- [ ] **Step 2: Test in Chrome**

Open `index.html` in Chrome. Tap the button — it should turn red and pulse. Speak a word. The button should stop pulsing after you stop speaking. (No translation yet — that's next.)

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add speech recognition with button state feedback"
```

---

### Task 3: Translation via MyMemory API

**Files:**
- Modify: `index.html` (replace the `translateAndSpeak` function)

- [ ] **Step 1: Implement translation fetch**

Replace the `translateAndSpeak` function with:

```javascript
async function translateAndSpeak(text) {
    try {
        const url = `https://api.mymemory.translated.net/get?q=${encodeURIComponent(text)}&langpair=en|fi`;
        const response = await fetch(url);
        if (!response.ok) throw new Error('API error');
        const data = await response.json();
        const translated = data.responseData.translatedText;
        speak(translated);
    } catch (e) {
        btn.classList.add('error');
        setTimeout(() => btn.classList.remove('error'), 600);
    }
}

function speak(text) {
    // Will be implemented in Task 4
}
```

- [ ] **Step 2: Test translation**

Open `index.html` in Chrome. Open DevTools console. Tap the button, say "hello". In the console, temporarily add a `console.log(translated)` after line `const translated = ...` to verify you get "Hei" or similar Finnish text back.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add MyMemory translation API call (English to Finnish)"
```

---

### Task 4: Finnish Text-to-Speech Output

**Files:**
- Modify: `index.html` (replace the `speak` function)

- [ ] **Step 1: Implement TTS**

Replace the `speak` function with:

```javascript
function speak(text) {
    const utterance = new SpeechSynthesisUtterance(text);
    utterance.lang = 'fi-FI';

    const voices = speechSynthesis.getVoices();
    const finnishVoice = voices.find(v => v.lang.startsWith('fi'));
    if (finnishVoice) {
        utterance.voice = finnishVoice;
    }

    btn.classList.add('success');
    utterance.addEventListener('end', () => {
        btn.classList.remove('success');
    });
    utterance.addEventListener('error', () => {
        btn.classList.remove('success');
    });

    speechSynthesis.speak(utterance);
}
```

- [ ] **Step 2: Test the full flow**

Open `index.html` on your phone (or Chrome with device emulation). Tap the button, say "hello" in English. You should hear the Finnish translation spoken aloud. The button should go blue → red (listening) → green (speaking) → blue (idle).

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add Finnish text-to-speech output, completing full flow"
```

---

### Task 5: Voice Loading Fix for Mobile

**Files:**
- Modify: `index.html` (add voice-loading handler before the `speak` function)

- [ ] **Step 1: Add voice preloading**

On some mobile browsers, `getVoices()` returns empty until a `voiceschanged` event fires. Add this code before the `speak` function definition:

```javascript
if (speechSynthesis.onvoiceschanged !== undefined) {
    speechSynthesis.onvoiceschanged = () => {};
}
```

And update the `speak` function to handle async voice loading:

```javascript
function speak(text) {
    const utterance = new SpeechSynthesisUtterance(text);
    utterance.lang = 'fi-FI';

    const setVoice = () => {
        const voices = speechSynthesis.getVoices();
        const finnishVoice = voices.find(v => v.lang.startsWith('fi'));
        if (finnishVoice) {
            utterance.voice = finnishVoice;
        }
    };

    setVoice();
    if (speechSynthesis.getVoices().length === 0) {
        speechSynthesis.addEventListener('voiceschanged', setVoice, { once: true });
    }

    btn.classList.add('success');
    utterance.addEventListener('end', () => {
        btn.classList.remove('success');
    });
    utterance.addEventListener('error', () => {
        btn.classList.remove('success');
    });

    speechSynthesis.speak(utterance);
}
```

- [ ] **Step 2: Test on mobile**

Test on an actual phone or Chrome DevTools mobile emulation. The full flow (tap → speak English → hear Finnish) should work.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "fix: handle async voice loading on mobile browsers"
```

---

### Task 6: Final Polish and Deploy Prep

**Files:**
- Modify: `index.html` (minor meta tags)

- [ ] **Step 1: Add PWA-friendly meta tags for mobile home screen**

Add these inside the `<head>` tag, after the `<title>`:

```html
<meta name="theme-color" content="#1a1a2e">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
```

- [ ] **Step 2: Test final version**

Full end-to-end test:
1. Open in Chrome mobile (or on actual phone)
2. Tap mic button → turns red, pulses
3. Say "good morning" in English
4. Button turns green, Finnish translation ("hyvää huomenta") is spoken aloud
5. Button returns to blue

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add mobile meta tags for home screen experience"
```

- [ ] **Step 4: Push to GitHub and enable Pages**

```bash
git push origin main
```

Then on GitHub: Settings → Pages → Source: Deploy from branch → Branch: `main`, folder: `/ (root)` → Save.

The app will be live at `https://<username>.github.io/<repo-name>/`.
