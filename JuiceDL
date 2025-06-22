import os
import base64
import requests
from pathlib import Path
from flask import Flask, request
from flask_cors import CORS
import yt_dlp

# Flask setup
app = Flask(__name__)
CORS(app)

# GitHub upload helper
def upload_to_github(local_path, repo, dest_path, token):
    try:
        with open(local_path, "rb") as f:
            content = f.read()
        b64_content = base64.b64encode(content).decode("utf-8")

        url = f"https://api.github.com/repos/{repo}/contents/{dest_path}"
        headers = {
            "Authorization": f"Bearer {token}",
            "Accept": "application/vnd.github.v3+json"
        }
        data = {"message": f"Add {dest_path}", "content": b64_content}

        response = requests.put(url, headers=headers, json=data)
        if response.status_code in [200, 201]:
            print(f"✅ Uploaded to GitHub: {dest_path}")
        else:
            print(f"❌ GitHub upload failed: {response.status_code} - {response.text}")
    except Exception as e:
        print(f"❌ Upload error: {e}")

# Download folder setup
def get_download_folder(media_type):
    base = Path("downloads/audio") if media_type == "audio" else Path("downloads/video")
    base.mkdir(parents=True, exist_ok=True)
    return base

# Audio downloader
def download_audio(song_name, artist_name):
    query = f"{song_name} {artist_name} audio"
    output_path = get_download_folder("audio")
    ydl_opts = {
        'format': 'bestaudio/best',
        'outtmpl': str(output_path / '%(title)s.%(ext)s'),
        'quiet': True,
        'no_warnings': True
    }

    try:
        with yt_dlp.YoutubeDL(ydl_opts) as ydl:
            info = ydl.extract_info(f"ytsearch:{query}", download=False)['entries'][0]
            print(f"🎵 Downloading: {info['title']}")
            ydl.download([info['webpage_url']])

        latest_file = sorted(output_path.glob("*.m4a"), key=os.path.getmtime)[-1]
        print(f"✅ Saved to: {latest_file}")

        github_token = os.environ.get("GITHUB_TOKEN")
        if github_token:
            upload_to_github(
                local_path=str(latest_file),
                repo="999geo/Juice-Geo",
                dest_path=f"downloads/{latest_file.name}",
                token=github_token
            )
    except Exception as e:
        print(f"❌ Audio download failed: {e}")

# Video downloader
def download_video(input_text):
    output_path = get_download_folder("video")
    ydl_opts = {
        'format': 'best',
        'outtmpl': str(output_path / '%(title)s.%(ext)s'),
        'quiet': True,
        'no_warnings': True
    }

    try:
        with yt_dlp.YoutubeDL(ydl_opts) as ydl:
            if input_text.startswith("http://") or input_text.startswith("https://"):
                print(f"🎬 Downloading from URL: {input_text}")
                ydl.download([input_text])
            else:
                print(f"🔍 Searching for video: {input_text}")
                info = ydl.extract_info(f"ytsearch:{input_text}", download=False)['entries'][0]
                print(f"🎬 Downloading: {info['title']}")
                ydl.download([info['webpage_url']])
        print(f"✅ Saved to: {output_path}")
    except Exception as e:
        print(f"❌ Video download failed: {e}")

# Web routes
@app.route("/", methods=["GET"])
def home():
    return "<h2>🎧 Juice DL is Live. Use the form at <code>/download</code> to send a track.</h2>"

@app.route("/download", methods=["POST"])
def download_from_web():
    song = request.form.get("song", "").strip()
    artist = request.form.get("artist", "").strip()
    if not song or not artist:
        return "❌ Missing song or artist", 400
    download_audio(song, artist)
    return f"✅ Downloaded: {song} by {artist}"

# Start app
if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
