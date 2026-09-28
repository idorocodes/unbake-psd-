# Unbake PSD

> Turn a flat PNG/JPG flyer back into an editable, layered `.psd` — right in your browser.

Unbake PSD takes a flattened flyer image, detects and removes the text, reconstructs the background underneath, and reassembles everything into a real Photoshop file you can open and edit — no Photoshop install required, thanks to an embedded Photopea editor.

---

## How it works

```
┌─────────────────┐     ┌──────────────────────┐     ┌─────────────────────┐
│   User upload    │ ──> │   Python CV engine    │ ──> │   PSD compiler        │
│   (PNG / JPG)    │     │   (EasyOCR + LaMa)    │     │   (ag-psd)            │
└─────────────────┘     └──────────────────────┘     └─────────────────────┘
                                                                  │
                                                                  ▼
                                                    ┌─────────────────────────┐
                                                    │   Photopea (embedded)   │
                                                    │   in-browser editing    │
                                                    └─────────────────────────┘
```

1. **Upload** — the user drops a flyer into the app.
2. **Detect** — `EasyOCR` finds every text block: its position, size, and dominant color.
3. **Clean** — the detected regions are masked out and `LaMa` inpaints the background so no text remains.
4. **Assemble** — the cleaned background and the extracted text are combined into a layered `.psd`.
5. **Edit** — the `.psd` loads directly into an embedded Photopea instance for in-browser editing, no install needed.

---

## Features

- **Automatic text detection** — finds text position, size, and color without any manual tagging.
- **Background reconstruction** — generative inpainting removes text cleanly instead of just blurring it out.
- **Real PSD output** — produces an actual layered `.psd`, not a flattened re-export.
- **In-browser editor** — Photopea is embedded directly in the app; edits happen with zero setup.
- **Free & open-source stack** — no paid APIs or SaaS dependencies required to run it.

---

## Tech stack

| Layer | Tools |
|---|---|
| Frontend | React, Tailwind CSS |
| Backend | Python (FastAPI/Flask), OpenCV |
| Text detection | EasyOCR |
| Background reconstruction | LaMa (inpainting) |
| PSD generation | `ag-psd` |
| Editor | Photopea (embedded via iframe) |

---

## Project structure

```
unbake-psd/
├── backend/
│   ├── app.py              # API routes (upload, status)
│   ├── ocr_engine.py       # Text detection
│   ├── mask_utils.py       # Mask generation from OCR boxes
│   ├── inpaint_engine.py   # Background reconstruction
│   ├── pipeline.py         # Orchestrates OCR → mask → inpaint
│   ├── psd_builder.py      # Assembles the final .psd
│   ├── schemas.py          # Request/response models
│   └── requirements.txt
├── frontend/
│   ├── src/
│   │   ├── App.jsx
│   │   ├── api/
│   │   │   └── client.js           # uploadFlyer(), pollJobStatus()
│   │   └── components/
│   │       ├── UploadDropzone.jsx
│   │       ├── ProcessingStatus.jsx
│   │       └── PhotopeaEditor.jsx
│   └── package.json
├── tests/
│   └── test_ocr.py
├── README.md
└── LICENSE
```

---

## Getting started

### Backend

```bash
cd backend
python -m venv venv
source venv/bin/activate   # Windows: venv\Scripts\activate
pip install -r requirements.txt
python app.py
```

### Frontend

```bash
cd frontend
npm install
npm run dev
```

The frontend expects the backend running at `http://localhost:8000` (adjust the base URL in `src/api/client.js` if needed).

---

## API overview

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/upload` | Accepts an image file, returns a `job_id` |
| `GET` | `/api/status/{job_id}` | Returns current step, completion status, and the output `.psd` URL once ready |

---

## Photopea integration

Once a `.psd` is ready, the frontend loads it into Photopea via the hash-based config API:

```javascript
const config = {
  files: [psdUrl],
  server: {
    version: 1,
    url: "/api/save",
    formats: ["psd", "png"],
  },
};

iframe.src = `https://www.photopea.com#${encodeURIComponent(JSON.stringify(config))}`;
```

Edits and exports from Photopea are sent back through `postMessage` and handled by `handleSaveCallback()` on the frontend.

---

## Known limitations

- **Font matching**: exact custom fonts can't always be recovered from a raster image. Text layers fall back to a standard open-source font, and the correct `.ttf` can be reassigned manually in Photopea.
- **Processing time**: on free-tier CPU compute, a single flyer typically takes 3–8 seconds to process. The UI surfaces live status ("Detecting text…", "Cleaning background…", "Building PSD…") rather than a blank wait.

---


## Contributing

Issues and pull requests are welcome. If you're adding a new backend step, keep it as its own module under `backend/` with a single clear entry-point function, matching the existing structure.

---

## License

Distributed under the MIT License. See `LICENSE` for details.
