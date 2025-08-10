# 🎬 Video as Code

Automated PowerPoint-to-video conversion system that transforms `.pptx` presentations into professionally narrated videos using a microservices architecture and GitHub Actions automation.

---

## 📦 What It Does

This system uses a three-stage microservices pipeline to automate video creation:

1. **Extract**: Uses `pptx_extractor` microservice to extract slide images and presenter notes from uploaded `.pptx` files
2. **Synthesize**: Leverages `tts-microservices` with Azure Cognitive Services to convert slide notes into high-quality MP3 audio narration
3. **Compose**: Employs `video-producer-microservice` with FFmpeg to combine slides and audio into professional MP4 videos
4. **Deploy**: Automatically commits the final video to the `outputs/` directory

---

## 🚀 How to Use

### 🔁 Triggering Video Generation

```bash
# Add your PowerPoint presentation to the input directory
cp your-presentation.pptx input/
git add input/your-presentation.pptx
git commit -m "Add presentation for video generation"
git push origin main
```

The GitHub Action automatically triggers when `.pptx` files are pushed to the `input/` directory, running the complete video generation pipeline.

### 🔧 Manual Triggering

You can also trigger the workflow manually:
```bash
# Using GitHub CLI
gh workflow run "Video Pipeline (Integrated Microservices)"

# Or use the GitHub Actions UI in your browser
```

---

## 🛠 Requirements & Configuration

### 🔐 Required GitHub Secrets

Configure these secrets in your repository settings:

| Secret Name       | Description                                       | Example |
|-------------------|---------------------------------------------------|---------|
| `SPEECH_KEY`      | Azure Cognitive Services Speech API Key           | `abc123def456...` |
| `SPEECH_REGION`   | Azure region of your Speech resource              | `eastus`, `canadacentral` |

### 🔐 Required GitHub Variables

Configure these variables in your repository settings:

| Variable Name     | Description                                       | Options |
|-------------------|---------------------------------------------------|---------|
| `VOICE_GENDER`    | Voice selection for text-to-speech synthesis     | `male`, `female` (default: `female`) |

**Voice Mapping:**
- `female` → `en-US-AvaMultilingualNeural` (Azure Neural Voice)
- `male` → `en-US-BrandonMultilingualNeural` (Azure Neural Voice)

### 📋 Repository Permissions

**Critical**: Ensure your repository has the following workflow permissions:
- ✅ **Read and write permissions** for Actions
- ✅ **Allow GitHub Actions to create and approve pull requests** (if needed)

This allows the workflow to automatically commit generated videos back to the repository.

### 🐳 Container Dependencies

The pipeline uses three microservices from the `ghcr.io/40docs/` registry:
- `pptx_extractor:latest` - PowerPoint slide and note extraction
- `tts-microservices:latest` - Text-to-speech audio generation  
- `video-producer-microservice:latest` - Video assembly with FFmpeg

---

## 🗃 Directory Structure

```
input/                          # Source PowerPoint files
├── *.pptx                      # PowerPoint presentations (triggers pipeline)

assets/                         # Build-time assets (ephemeral)
├── images/                     # Extracted slide images (PNG format)
│   └── slide_*.png
├── audio/                      # Generated narration files (MP3 format) 
│   └── slide_*.mp3
└── bumpers/                    # Intro/outro video clips
    ├── bumper_in.mp4           # Opening bumper video
    └── bumper_out.mp4          # Closing bumper video

outputs/                        # Final video products (committed)
└── *.mp4                       # Generated narrated videos

.github/workflows/
└── trial.yml                   # Main video generation pipeline

# Build-time directories (created during pipeline execution)
slides/                         # Temporary slide extraction
notes/                          # Temporary note extraction  
audio/                          # Temporary audio generation
```

**Key Points:**
- Only `input/` and `outputs/` directories are tracked in git
- Intermediate assets (`assets/images/`, `assets/audio/`) are ephemeral and not committed
- Output filenames preserve input names: `presentation.pptx` → `presentation.mp4`

---

## 🏗️ Pipeline Architecture

The video generation pipeline consists of three containerized microservices orchestrated via GitHub Actions:

### Stage 1: PowerPoint Extraction 
```bash
# pptx_extractor microservice (port 5003)
- Extracts slide images as PNG files
- Extracts presenter notes as text files  
- Volume mounts: input/ → /app/uploads, slides/ → /app/slides, notes/ → /app/notes
- API: POST /extract with multipart file upload
```

### Stage 2: Text-to-Speech Synthesis
```bash
# tts-microservices microservice (port 5002) 
- Converts slide notes to MP3 audio using Azure Cognitive Services
- Voice selection based on VOICE_GENDER environment variable
- Volume mount: audio/ → /app/audio
- API: POST /text-to-speech with JSON payload {text, filename}
```

### Stage 3: Video Production
```bash
# video-producer-microservice (port 5003)
- Combines slide images with audio narration using FFmpeg
- Generates final MP4 video with synchronized audio/visual
- Volume mounts: assets/ → /app/assets, outputs/ → /app/outputs  
- API: POST /create-video with JSON payload {output_name}
```

### Pipeline Flow
1. **Trigger**: `.pptx` file pushed to `input/` directory
2. **Extract**: Launch pptx_extractor → upload PPTX → extract slides & notes  
3. **Synthesize**: Launch tts-microservices → convert each note to audio
4. **Assemble**: Reorganize assets → launch video-producer → create final MP4
5. **Deploy**: Commit generated video to `outputs/` directory

---

## 📌 Implementation Details

- **Infrastructure**: Uses GitHub-hosted runners with no additional infrastructure requirements
- **Persistence**: Only final MP4 videos are committed; all intermediate assets are ephemeral  
- **Naming Convention**: Output filenames preserve input names (`presentation.pptx` → `presentation.mp4`)
- **Container Registry**: All microservices hosted on GitHub Container Registry (`ghcr.io/40docs/`)
- **Volume Strategy**: Docker volume mounts enable data sharing between containers and runner filesystem

---

## 📅 Development Roadmap

### ✅ Completed Features
- **Microservices Architecture**: Converted monolithic FFmpeg logic to containerized microservices
- **Multi-Voice Support**: A/B testing capabilities with male/female voice options
- **Multi-Language Support**: Azure Cognitive Services multilingual neural voices
- **Containerization**: Full Docker containerization with GitHub Container Registry

### 🚧 In Progress
- **Enhanced Audio Quality**: Improved voice synthesis and audio processing
- **Performance Optimization**: Parallel processing and caching improvements

### 📋 Planned Features
- **Dynamic Lower Thirds**: Customizable text overlays and branding elements
- **Subtitle Generation**: Automatic SRT subtitle file creation
- **Auto-Publishing**: Direct integration with video platforms (YouTube, Vimeo)
- **Kubernetes Deployment**: Helm charts and K8s manifests for scalable deployment
- **Web Frontend**: GUI interface for easier presentation management
- **Advanced Transitions**: Slide animations and transition effects
- **Screen Recording**: Headless recording functionality for technical demonstrations
- **Template System**: Reusable video templates and themes
- **Batch Processing**: Multiple presentation processing in single workflow

---

## 🔧 Troubleshooting

### Common Issues

#### Workflow Not Triggering
```bash
# Verify file placement and git status
ls -la input/
git status
git log --oneline -5

# Check workflow file syntax
gh workflow view "Video Pipeline (Integrated Microservices)"
```

#### Missing Audio in Final Video
- Ensure presenter notes exist in PowerPoint slides
- Verify `SPEECH_KEY` and `SPEECH_REGION` secrets are configured
- Check Azure Cognitive Services quota and billing status

#### Container Pull Failures  
- Verify GitHub Container Registry access
- Check if microservice images exist: `ghcr.io/40docs/pptx_extractor:latest`

#### Permission Errors During Commit
- Verify repository has **Write** permissions enabled for GitHub Actions
- Check if branch protection rules are blocking automated commits

### Debug Commands
```bash
# Check workflow run details
gh run list --workflow="Video Pipeline (Integrated Microservices)"
gh run view <run-id> --log

# Validate secrets and variables
gh secret list
gh variable list
```

---

## 🤝 Contributing

When contributing to this repository:

1. **Test Locally**: Validate your PowerPoint files have presenter notes
2. **Small Changes**: Test with small PPTX files first to validate pipeline
3. **Documentation**: Update this README if you modify the pipeline architecture
4. **Container Updates**: Coordinate with maintainers before updating microservice versions
