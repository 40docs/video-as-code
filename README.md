# 🎬 Video as Code

This repo takes a `.pptx` PowerPoint file, extracts slides and presenter notes, converts the notes to speech, and compiles everything into a narrated video — all automated via GitHub Actions.

---

## 📦 What It Does

1. Extracts images and notes from uploaded `.pptx`
2. Converts slide notes to `.mp3` audio using Azure Cognitive Services
3. Uses `ffmpeg` to generate a final `.mp4` video, combining each slide with its narration
4. Commits the final video to the `outputs/` folder

---

## 🚀 How to Use

### 🔁 Triggering a Build

Push a `.pptx` file to the `input/` directory on `main` branch.  
This kicks off a GitHub Action that runs the full video generation pipeline.

---

## 🛠 Requirements

### 🔐 GitHub Secrets (Required)

| Secret Name       | Description                                       |
|-------------------|---------------------------------------------------|
| `SPEECH_KEY`      | Azure Cognitive Services Speech API Key           |
| `SPEECH_REGION`   | Azure region of your Speech resource (e.g. `eastus`, `canadacentral`) |

### GitHub Repository Permissions for `workflow`

Ensure both `read` and `write` permissions are enabled. This workflow recommits the finished product back to the github repository hosting the solution.

![image](https://github.com/user-attachments/assets/d67015b6-5373-4d98-bb7e-c8367ffbb47c)

---

## 🗃 Directory Structure

```bash
input/              # Place your .pptx files here (auto-detected)
assets/images/      # Populated during build (not committed)
assets/audio/       # Populated during build (not committed)
outputs/            # Final generated .mp4 lives here
scripts/create_video.py  # FFmpeg-based video compiler
```

---

## 📌 Notes

- Only the final video is committed. Audio and image intermediates are ignored.
- PPTX filenames are preserved in the output (e.g., `slides.pptx` → `slides.mp4`).
- This setup uses GitHub-hosted runners — no infra needed.

---

## 📅 Roadmap

- [x] Convert FFmpeg build logic to a Flask microservice
- [ ] Add support for Kubernetes deployment (manifests)
- [ ] Add Animation/Transition support
- [ ] Add Headless recording functionality for Technical Demos
