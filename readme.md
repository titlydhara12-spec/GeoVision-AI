# 🛰️ Remote Sensing Vision-Language Assistant — Website

## 👥 Project Team

| Name | Role |
|---|---|
| Titly Dhara | Project Lead |
| Archit Kumawat | Team Member |
| Khushal Jangid | Team Member |
| Srishti Sharma | Team Member |

A real website where a user can:
- **Search any place on Earth** (e.g. "Bengaluru, India") and the map flies there
- **Pan/zoom** the live satellite map to any area
- Click **"Use this map area"** and the site automatically fetches the real
  satellite image for exactly what's on screen
- Or **upload their own image** instead
- Then **ask a question in plain English**:
  - "How many buildings are visible?"
  - "Where is the vegetation?"
  - "Identify the water bodies."
  - "Is this a good place to start a business?"
  - "Compare these two images and identify major changes." (needs a 2nd image)

---

## 1. Project structure

```
rs-vqa-web/
├── app.py                 # Flask backend (routes + API)
├── cv_tools.py             # All the image-analysis logic (AI/ML part)
├── requirements.txt
├── Dockerfile              # for free deployment on Hugging Face Spaces
├── templates/
│   └── index.html          # the webpage
└── static/
    ├── css/style.css
    └── js/app.js            # map, search, upload, ask logic
```

Open this whole folder in VS Code.

---

## 2. Run it locally

### Step 1 — Python environment
```bash
cd rs-vqa-web
python3 -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

### Step 2 — Run the server
```bash
python app.py
```
Open **http://localhost:7860** in your browser.

### Step 3 — Try it
1. Type a place in the search box (e.g. "Mumbai") and press Enter — the map flies there.
2. Zoom in until you can clearly see buildings/streets (city block level works best).
3. Click **"Use this map area."**
4. Ask: *"How many buildings are visible?"* or *"Is this a good place to start a business?"*
5. For comparison: open "Add a second image," pan the map to a different time/place view (or upload two images of the same area from different dates), then ask *"Compare these two images."*

The first "general" question (one that isn't building/vegetation/water/business/
compare) downloads the BLIP AI model once (~1GB) — do this ahead of time on
good Wi-Fi, before your demo.

---

## 3. Where the data comes from (free, no API key)

| Purpose | Service | Notes |
|---|---|---|
| Map + visible satellite imagery | Esri World Imagery (public tile service) | Free, global, no signup |
| Place search ("go to Bengaluru") | Nominatim (OpenStreetMap) | Free, no signup, be reasonable with request rate |
| Actual image pixels for AI analysis | Esri World Imagery "export" endpoint | Free, returns a real PNG for a bounding box |

No API keys, no billing, nothing to configure — this is exactly why it's
realistic to finish in a hackathon.

---

## 4. Deploying it live (free)

### Recommended: Hugging Face Spaces (Docker)
This is the best free option here because it gives ~16GB RAM on the free CPU
tier, which comfortably fits the AI model (many other free hosts only give
512MB, which is too little).

1. Create a free account at https://huggingface.co
2. Click **New Space** → SDK: **Docker** → visibility: Public.
3. Upload every file in this folder (keep the folder structure: `templates/`,
   `static/`, `Dockerfile`, `app.py`, `cv_tools.py`, `requirements.txt`) —
   either via the web UI's "Add file" or with `git push` if you're
   comfortable with git.
4. The Space builds automatically (takes a few minutes) and gives you a
   public URL like `https://huggingface.co/spaces/yourname/rs-vqa-web`.
5. Share that URL — anyone can open it, no installation needed.

### Backup for demo day: ngrok (no deployment needed)
If Wi-Fi at the venue is unreliable for reaching your deployed URL:
```bash
# terminal 1
python app.py
# terminal 2 (install ngrok first: https://ngrok.com/download)
ngrok http 7860
```
This gives a temporary public link straight to your laptop.

### Alternative: Render.com
Also free, works for the web part, but its free tier has a small memory
limit that can struggle with the AI model — if you deploy there, it's safer
to skip the "general question" BLIP fallback and rely on the 5 specific
query types (building/vegetation/water/change/business), which don't need
the AI model at all.

---

## 5. Honest scope notes for your pitch

- **Building count**, **vegetation %**, **water %**, and **change detection**
  use classical, training-free computer vision (edge detection, color
  thresholds, image differencing) — fast, explainable, works on CPU.
- **Business suitability** is a transparent heuristic combining building
  density, vegetation %, and water % into a 0–100 score with a plain-English
  explanation. It is clearly **not** financial or professional advice —
  it's a starting signal, and the app says so in its answer.
- **Open-ended questions** fall back to BLIP, a real pretrained
  vision-language model — this is what makes it feel like an assistant,
  not four fixed buttons.
- **What it doesn't do (be upfront if asked):** it doesn't scan all of India
  to find the single best spot for a business — it scores whichever specific
  area the user has selected. A user can check a few candidate areas one at
  a time and compare the scores. Scanning a whole country automatically is a
  reasonable "future work" answer if judges ask.

---

## 6. Common issues

| Problem | Fix |
|---|---|
| `pip install` fails on torch | `pip install torch --index-url https://download.pytorch.org/whl/cpu` then re-run `pip install -r requirements.txt` |
| Satellite image fails to load | You may be zoomed out too far (huge bounding box) — zoom in and retry |
| Map tiles don't show | Check your internet connection; Esri tiles are fetched live in the browser |
| First question is slow | Normal — BLIP model is downloading/loading. Ask a warm-up question before your demo. |
| Hugging Face Space fails to build | Check the build log — usually a missing system library; the provided `Dockerfile` already includes what OpenCV needs |