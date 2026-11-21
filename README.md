# Simple Subtitle Helper & Sync Tool

A single-file, browser-based tool for reading, aligning and practising with several subtitle files side by side, with phrase-by-phrase audio playback. No installation, no server, no internet connection: open `index.html` and work.

---

## 🌟 What is this for?
- **Language practice:** Show the same episode in several languages, blur the ones you're learning, listen to a phrase, then click to check yourself.
- **Fixing sync:** If a translation doesn't line up with the original, drag the phrase to the right row.
- **Making multi-language subs:** Combine several languages into one `.srt` file.
- **Flashcards:** Export the rows to Anki or Quizlet.

---

## 🚀 Features
* **As many columns as you have subtitles:** Open a folder and you get one column per `.srt` file next to a **Timecodes** column. Column titles come from the language suffix (`Show.E01.es.srt` → `es`).
* **Align by time:** All columns are lined up on one timeline: the column with audio clips, else the one with an audio file, else the one with the most lines. Each phrase from another language goes into the row it overlaps most in time. If two phrases land in the same row they are joined, and no text is lost.
* **Phrase-by-phrase audio:** In a column that has its own audio file, hover over a phrase and press ▶ to hear exactly that phrase. Playback stops by itself at the end of the phrase.
* **Drag & drop:** Move any phrase up or down with the handle (⠿) without affecting other columns. A phrase keeps its audio when you move it.
* **Replace one column:** Drop a single `.srt` onto a column to replace only that column. The other columns and your edits stay.
* **Smart blur:**
    * **Single click:** removes the blur and opens the cell for editing.
    * **Double click:** blurs the cell and locks it so you don't edit it by accident.
    * **Eye icon (header):** blurs or unblurs the whole column.
* **Resizable columns, sticky header, text formatting** (select text for **Bold** / *Italic* / <u>Underline</u>).
* **Fully offline:** The only library (SortableJS 1.15.0, MIT) is embedded in the page.

---

## 📁 Preparing a folder

Put the subtitles and the audio for one episode in one folder:

```
es/
├── Show.E01.en.srt       ← no audio file with this name: no ▶ in this column
├── Show.E01.es.srt
├── Show.E01.es.ogg       ← audio for Show.E01.es.srt
└── Show.E01.ru.srt       ← no ▶
```

**An audio file must have exactly the same name as its subtitle file**, with only the extension different: `Show.E01.es.srt` ↔ `Show.E01.es.ogg`. A subtitle file without a matching audio file gets no ▶ buttons. The same applies to a single `.srt` dropped onto a column. If several languages need audio, give each its own file (`Show.E01.en.ogg`, …).

**Recommended format: Ogg Opus.** Make it with ffmpeg from the video or the extracted audio track:

```bash
ffmpeg -i Show.E01.mkv -map 0:a:0 -ac 2 -c:a libopus -b:a 96k Show.E01.es.ogg
```

A one-hour episode is about 45 MB.

**Pre-cut clips (alternative):** A subfolder with the same name as the subtitle file without `.srt` (`Show.E01.es/`), holding numbered files `0001.flac`, `0002.flac`, …. Clip *N* plays for phrase *N*. If both are present, the single audio file wins.

---

## 🔊 How playback works

The browser's normal media player starts badly right after a seek: the first sound can stutter or come out distorted. So the page doesn't use it for `.ogg`/`.opus` files. Instead:

1. When the folder loads, the page reads the Ogg file once and builds an index of every Opus packet and its exact time (≈0.2 s for an hour).
2. When you hover a phrase (or click it), the page takes only the packets for that phrase, plus 120 ms before it so the decoder can warm up. It decodes them in memory with WebCodecs and trims the result to the phrase's exact start and end.
3. The ready clip plays from memory through Web Audio, with an 8 ms fade in and out to avoid clicks. The audio context stays open, so the output device doesn't fall asleep between phrases.

The result matches an ffmpeg decode of the same interval sample for sample. Up to 300 cut phrases are kept in memory, so replays are instant.

**Pad, s** (top bar, default 0.15) adds that much audio before and after each phrase, so first and last syllables aren't clipped. Changes apply on the next play.

**Browsers:** In-browser cutting needs WebCodecs `AudioDecoder`, available in recent Chrome, Edge and Firefox. Without it, or for other formats (`.m4a`, `.mp3`, …), the page falls back to the normal player with seeking. Avoid MP3 there, because seeking in variable-bitrate MP3 is inaccurate. E-AC-3 / AC-3 tracks from video files are not playable in browsers, so convert them first.

---

## 🛠 How to use
1. **Open:** Click **Open folder** and pick the episode folder, or drag the folder onto the table. **Load files** opens individual `.srt` files (no audio).
2. **Listen:** Hover a phrase and press ▶ (■ stops). Pressing another phrase interrupts the current one.
3. **Align:** Leave **Align by time** on (it applies on the next load), then fix any remaining rows with the ⠿ handles.
4. **Edit and blur:** One click to reveal and type, double click to hide a phrase for practice.
5. **Save:** **Save result** downloads `project.srt`. Each block holds the texts of all columns separated by `---`. Open it in this tool again to get the columns back.
6. **Flashcards:** **Anki / Quizlet Export** downloads `flashcards_import.txt`, one row per line with columns separated by tabs (UTF-8).

---

## ⚠️ Known limitations
* `project.srt` uses the timeline column's timecodes for every row, and keeps formatting tags (`<b>`, `<i>`, `<br>`). It is meant for this tool rather than for video players.
* Manually moving cells shifts that column against the Timecodes column. Re-check alignment before saving.
* Hover-to-show ▶ needs a mouse. On touch screens the buttons are always visible.

---

## 🔒 Privacy
Your files never leave your computer. The page makes no network requests, and everything happens in your browser.
