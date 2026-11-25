# Local services

The imported Hypit snapshot keeps its local service implementations with their owning
Provider packages rather than in this top-level directory:

* WhisperX: [`packages/provider-whisperx-local/runtime`](../packages/provider-whisperx-local/runtime/README.md)
* OpenCV: [`packages/provider-image-opencv-local/runtime`](../packages/provider-image-opencv-local/runtime/pyproject.toml)
* yt-dlp: [`packages/yt-dlp/runtime`](../packages/yt-dlp/runtime/README.md)

Use the selected Runtime Profile and explicit `hypit programs prepare` / `hypit runtime up`
commands to manage them. No Docker image or separate service source checkout is required.
See [standalone setup](../STANDALONE.md) for prerequisites and credentials.
