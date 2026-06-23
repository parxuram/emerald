# EMERALD — Frontend Connection Guide

## Prerequisites

- The EMERALD notebook running in Google Colab (GPU runtime)
- `emerald_frontend.html` opened locally in Chrome or Edge
- A free [ngrok](https://ngrok.com) account with your authtoken

---

## Setup Steps

### 1. Configure the ngrok token in Colab

In the API cell near the bottom of the notebook, replace the placeholder with your ngrok authtoken:

```python
NGROK_AUTHTOKEN = "your_token_here"
```

### 2. Run the notebook

Go to **Runtime → Run all** (`Ctrl+F9`). When the API cell completes, it prints a public URL:

```
EMERALD API is LIVE
URL → https://abc123.ngrok.io
```

Copy this URL. It changes every time the Colab session restarts.

### 3. Open the control panel

Double-click `emerald_frontend.html` to open it in your browser.

### 4. Connect

In the **"1 · Connect to Colab"** field, paste the ngrok URL and click **Connect**. The status indicator in the top-right turns green when the connection is successful.

### 5. Set the video path

In the **"2 · Video path"** field, enter the Colab-side path to your video:

```
/content/drive/MyDrive/your_folder/video.mp4
```

### 6. Send config and start

Click **↑ Send Config to Colab**, then **▶ Start Pipeline**. The live annotated frame appears in the main panel within a few seconds.

---

## Troubleshooting

| Symptom | Resolution |
|---|---|
| "Could not connect" | Verify the ngrok URL is current and the API cell ran without errors |
| Connected but no frame | Click **Start Pipeline** — frames only stream while the pipeline is active |
| URL stopped working | The Colab session disconnected; rerun the API cell and copy the new URL |
| No violations appearing | Click **↻ Refresh Violations** in the log panel |

---

> Evidence frames are stored in Colab at `violation_output/frames/`. Copy this folder to Google Drive before ending your session to retain the output.S