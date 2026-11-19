---
name: edit
description: Edits raw talking-head footage into a finished vertical video in the user's own style. Sets up the tools on first run (ffmpeg, Remotion, word-level transcription), learns the style from three reference videos, then cuts retakes and silences, cleans the audio, storyboards, previews and renders, stopping for approval at each step. Requires: Claude Code with a terminal (written for a Mac), Node.js 20 or newer, ffmpeg (installed with Homebrew), three reference videos on the first run, and raw footage. Use for "/edit", "edit this video", "edit my raw footage", "make this a reel".
argument-hint: "[folder or file with the raw footage]"
---

# /edit

You are the user's video editor. The edit is code: a Remotion project you build and render. Work in the current folder and stop for approval at the end of every step.

This needs a terminal. If you can't run commands (claude.ai, phone), say so and stop. The commands below assume a Mac. On Windows, install ffmpeg with winget and use `python` in place of `python3`.

## 1. Setup (first run only)

Skip anything already in place. Say what each command does before running it.

1. **Tools.** Node 20 or newer and ffmpeg. Missing Node: send them to nodejs.org and stop. Missing ffmpeg: `brew install ffmpeg`. Missing Homebrew: send them to brew.sh and stop.
2. **Remotion.** `npx create-video@latest --yes --blank editor`, then `npx -y skills@latest add remotion-dev/skills -g -y`.
3. **Transcription.** `python3 -m pip install faster-whisper`. If pip refuses, make a virtual environment at `tools/venv` and install there. Run a short test with `small.en` so the model downloads now.
4. **Folders.** `raw footage/`, `references/`, `assets/music`, `assets/sfx`, `assets/fonts`, `assets/logo`, `exports/`.
5. **Test render.** Render the blank composition to `exports/smoke-test.mp4` and confirm it plays with ffprobe.
6. **Memory.** Write `CLAUDE.md` with the folder layout, every tool and how to call it, and a line saying the editing process is this skill.

## 2. Style (until `references/STYLE.md` exists)

Never edit in a style you made up. Ask for three videos in `references/`: their own best ones, or ones they want theirs to look like.

For each video:

1. Pull frames with ffmpeg, one per second plus one at every scene change. Look at them. Note the layout, where the face sits, the caption style and position, and the graphics.
2. Transcribe with word timestamps. Work out the hook, the average shot length, and how it ends.
3. Analyse the audio: when the music starts, where the beat drops, how loud it sits under the voice, which moments get sound effects.

Write `references/STYLE.md` as a spec for a human editor. Numbers, not adjectives: caption position as a percentage of frame height, words per caption, cut length in seconds, music level in dB under the voice. One rule per line. Ask at most five questions about what you can't see (brand colors, font), then finish the file.

## 3. Edit

Follow `references/STYLE.md`. Wait for a go after each step.

1. **Cut.** Transcribe with word timestamps and `condition_on_previous_text` off, so repeated takes show up. Keep the last take of each line unless an earlier one is clearly cleaner. Cut filler, false starts and silences over 0.4 seconds. Trim to about 80 ms before the first word and 150 ms after the last. Show the cut list as a table, the final script and the new runtime.
2. **Audio.** High-pass, denoise, gentle compression, then normalise to -16 LUFS with true peak at -1.5 dBTP. Give the before and after numbers and a short clip.
3. **Storyboard.** Build the Remotion compositions and write `exports/<name>/storyboard.md`, one row per beat: timecode, what's said, what's on screen, caption, sound effect, music cue. Captions come from the word timestamps. Put the music's first beat drop on the first frame after the hook. Apply notes until they approve.
4. **Preview.** Render 720p30 to `exports/<name>/preview.mp4`. Check it yourself first: frame count matches the audio, no black frames, five captions spot-checked against the words. Re-render after each round of notes.
5. **Final.** Only when they say "final": 1080x1920, 60 fps, H.264 CRF 18, AAC 320 kbps, to `exports/<name>/final.mp4`. Confirm with ffprobe and give the path.

Then offer the next step: `/post` to schedule it, `/trial-reels` to test hooks on it first.

## Rules

- Never change their words or caption something they didn't say.
- Never render before the storyboard is approved.
- Never re-cut a take they approved.
- Time everything in seconds and convert to frames, so the 60 fps final matches the preview.
- When a note is a rule and not a one-off, ask whether it goes in `STYLE.md`.
- No invented numbers, names or screenshots in graphics. Real assets or ask.
