# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is the **video-as-code** repository, part of the 40docs platform. It automates the conversion of PowerPoint presentations (`.pptx`) into narrated videos using a microservices architecture and GitHub Actions.

## Architecture

The system uses a microservices-based pipeline orchestrated via GitHub Actions:

1. **pptx_extractor**: Extracts slides (PNG images) and presenter notes from uploaded `.pptx` files
2. **tts-microservices**: Converts slide notes to MP3 audio using Azure Cognitive Services 
3. **video-producer-microservice**: Uses FFmpeg to combine slides and audio into final MP4 videos

All services run as Docker containers from the `ghcr.io/40docs/` registry.

## Development Workflow

### Triggering Video Generation
```bash
# Place PowerPoint file in input/ directory and push to main branch
cp presentation.pptx input/
git add input/presentation.pptx
git commit -m "Add new presentation for video generation"
git push origin main
```

The GitHub Action will automatically:
- Extract slides and notes from the PPTX
- Generate audio narration for each slide
- Create a final video combining slides with audio
- Commit the result to `outputs/presentation.mp4`

### Manual Trigger
```bash
# Trigger workflow manually via GitHub Actions UI or CLI
gh workflow run "Video Pipeline (Integrated Microservices)"
```

## Required Configuration

### GitHub Secrets (Required)
- `SPEECH_KEY`: Azure Cognitive Services Speech API Key
- `SPEECH_REGION`: Azure region (e.g., `eastus`, `canadacentral`)

### GitHub Variables (Required)
- `VOICE_GENDER`: Set to `male` or `female` (defaults to `female`)
  - `male`: Uses `en-US-BrandonMultilingualNeural` voice
  - `female`: Uses `en-US-AvaMultilingualNeural` voice

### Repository Permissions
Ensure workflow has both `read` and `write` permissions for automatic commits.

## Directory Structure

```
input/              # Place .pptx files here (triggers pipeline)
assets/
  ├── images/       # Generated slide images (build-time only)  
  ├── audio/        # Generated MP3 narration (build-time only)
  └── bumpers/      # Intro/outro video clips
    ├── bumper_in.mp4
    └── bumper_out.mp4
outputs/            # Final MP4 videos (committed to repo)
```

## Pipeline Details

The GitHub Action workflow (`.github/workflows/trial.yml`) orchestrates three microservices:

1. **Extraction Phase**: Runs `pptx_extractor` container to extract slides and notes
2. **Audio Generation Phase**: Runs `tts-microservices` container for each slide note
3. **Video Assembly Phase**: Runs `video-producer-microservice` to create final MP4

Each phase uses volume mounts to share data between containers and the GitHub runner filesystem.

## Voice Configuration Logic

The pipeline dynamically selects Azure TTS voices based on `VOICE_GENDER`:
- Male: `en-US-BrandonMultilingualNeural`  
- Female: `en-US-AvaMultilingualNeural`
- Default: Female voice if variable not set

## Important Notes

- Only final MP4 videos are committed; intermediate assets are ephemeral
- Output filenames preserve input PPTX names (e.g., `slides.pptx` → `slides.mp4`)  
- The pipeline uses GitHub-hosted runners with no additional infrastructure requirements
- Large video files and proprietary codecs should not be committed to the repository

## Roadmap Features

The repository includes a roadmap with completed and planned features:
- ✅ Multiple voice support (A/B testing)
- ✅ Multiple language support  
- ✅ FFmpeg microservice conversion
- 🔄 Planned: Dynamic lower thirds, subtitles, auto-publishing, Kubernetes deployment