# Content Generation Guide

This guide explains how to generate multimedia content (text, images, and audio) for Bhagavad Gita verses using the verse-content-sdk.

## Prerequisites

1. **Install verse-content-sdk** (already installed via pip in editable mode)
2. **API Keys**: Set up in `.env` file:
   ```
   OPENAI_API_KEY=your_openai_api_key
   ELEVENLABS_API_KEY=your_elevenlabs_api_key
   ```

## Quick Start: Generate Everything for a Verse

The easiest way to generate all content for a verse is using the unified `verse-generate` command:

```bash
# Generate image and audio for Chapter 3, Verse 10
verse-generate --chapter 3 --verse 10 --image --audio

# Generate everything (including text placeholder)
verse-generate --chapter 3 --verse 10 --all
```

## Step-by-Step: Manual Generation

If you prefer to generate each component separately:

### 1. Create Scene Description

First, add a scene description to `docs/image-prompts.md`:

```markdown
### Chapter 3, Verse 10

**Scene Description**:
[Describe the visual scene that represents this verse - 3-5 sentences with details about setting, characters, colors, lighting, mood, etc.]
```

### 2. Generate Image

```bash
# Regenerate image for Chapter 3, Verse 10
verse-images --theme-name modern-minimalist --regenerate chapter-03-verse-10.png
```

The image will be saved to: `images/modern-minimalist/chapter-03-verse-10.png`

### 3. Create Verse File

Create the verse markdown file at `_verses/chapter_03_verse_10.md` with frontmatter:

```yaml
---
layout: verse
title_en: "Chapter 3, Verse 10"
title_hi: "अध्याय 3, श्लोक 10"
chapter: 3
verse_number: 10
previous_verse: /verses/chapter-03-verse-09/
next_verse: /verses/chapter-03-verse-11/
chapter_info:
  number: 3
  name_en: "Karma Yoga"
  name_hi: "कर्म योग"
image: /images/modern-minimalist/chapter-03-verse-10.png
audio_full: /audio/chapter_03_verse_10_full.mp3
audio_slow: /audio/chapter_03_verse_10_slow.mp3

devanagari: |
  [Add Sanskrit text here]

transliteration: |
  [Add transliteration here]

# ... add other fields ...
---
```

### 4. Generate Audio

```bash
# Regenerate audio for Chapter 3, Verse 10
verse-audio --regenerate chapter_03_verse_10_full.mp3,chapter_03_verse_10_slow.mp3
```

The audio files will be saved to:
- `audio/chapter_03_verse_10_full.mp3`
- `audio/chapter_03_verse_10_slow.mp3`

## Command Reference

### verse-generate (Unified Command)

```bash
# Full syntax
verse-generate --chapter N --verse M [--all|--text|--image|--audio] [--theme NAME]

# Examples
verse-generate --chapter 2 --verse 47 --all
verse-generate --chapter 1 --verse 5 --image --audio
verse-generate --chapter 4 --verse 20 --image --theme modern-minimalist
```

**Options:**
- `--chapter N`: Chapter number
- `--verse M`: Verse number (required)
- `--all`: Generate text, image, and audio
- `--text`: Generate text content only
- `--image`: Generate image only
- `--audio`: Generate audio only
- `--theme NAME`: Image theme (default: modern-minimalist)

### verse-images (Image Generation)

```bash
# Generate all images
verse-images --theme-name modern-minimalist

# Regenerate specific image
verse-images --theme-name modern-minimalist --regenerate chapter-01-verse-01.png

# Regenerate multiple images
verse-images --theme-name modern-minimalist --regenerate chapter-01-verse-01.png,chapter-01-verse-02.png

# Force regenerate all (with confirmation)
verse-images --theme-name modern-minimalist --force
```

### verse-audio (Audio Generation)

```bash
# Generate all audio
verse-audio

# Regenerate specific files
verse-audio --regenerate chapter_01_verse_01_full.mp3,chapter_01_verse_01_slow.mp3

# Start from specific verse
verse-audio --start-from chapter_02_verse_01_full.mp3

# Force regenerate all (with confirmation)
verse-audio --force
```

### verse-embeddings (Semantic Search)

```bash
# Generate embeddings using OpenAI
verse-embeddings --verses-dir _verses --output data/embeddings.json

# Generate using local models (free)
verse-embeddings --verses-dir _verses --output data/embeddings.json --provider huggingface
```

## Tips

1. **Image Generation**: Always create the scene description in `docs/image-prompts.md` before generating images
2. **Audio Generation**: Ensure the verse file exists with `devanagari:` field populated
3. **Theme Consistency**: Use the same theme name throughout (e.g., "modern-minimalist")
4. **API Costs**:
   - DALL-E 3 images: ~$0.04 per image
   - ElevenLabs audio: ~10 characters per audio file
   - Use `--regenerate` to avoid regenerating all files

## Workflow Example

Here's a complete workflow for adding Chapter 5, Verse 15:

```bash
# 1. Add scene description to docs/image-prompts.md manually

# 2. Create verse file at _verses/chapter_05_verse_15.md manually

# 3. Generate image and audio
verse-generate --chapter 5 --verse 15 --image --audio

# 4. Verify files were created
ls -lh images/modern-minimalist/chapter-05-verse-15.png
ls -lh audio/chapter_05_verse_15_*.mp3

# 5. Test locally
bundle exec jekyll serve
# Navigate to http://localhost:4000/bhagavad-gita/verses/chapter-05-verse-15/

# 6. Commit and push
git add _verses/chapter_05_verse_15.md images/ audio/
git commit -m "Add Chapter 5, Verse 15 with multimedia content"
git push
```

## Troubleshooting

**"Error: Scene description not found"**
- Add the scene description to `docs/image-prompts.md` first

**"Error: Verse file not found"**
- Create the verse markdown file in `_verses/` directory

**"Error: OPENAI_API_KEY not set"**
- Check `.env` file exists and contains valid API key
- Run `source .env` to load environment variables

**"Error: ELEVENLABS_API_KEY not set"**
- Check `.env` file exists and contains valid API key

**Command not found: verse-generate**
- Run: `pip install -e ~/workspaces/verse-content-sdk/`
- Or use full path: `/Users/arungupta/Library/Python/3.13/bin/verse-generate`

## See Also

- [Image Prompts Guide](image-prompts.md) - How to write effective scene descriptions
- [Theme Configuration](themes/) - How to configure visual themes
- [verse-content-sdk README](../../verse-content-sdk/README.md) - Full SDK documentation
