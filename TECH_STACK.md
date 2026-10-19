# Technology Stack — Napkin2Code

Comprehensive overview of the technologies, libraries, architectures, and design decisions used in **Napkin2Code**.

---

## 1. High-Level Architecture

Napkin2Code follows a **lightweight, local-first hybrid architecture**:
- **Backend**: Python micro-service using Flask for orchestration, image processing, and secure communication with AI models.
- **Frontend**: Vanilla Web standards (HTML5 / CSS3 / ES6+ JavaScript) with **zero Node.js build dependencies**.
- **AI / Vision**: Multimodal Gemma 4 models running via the hosted Google Gemini API.
- **Storage**: Browser-local storage (`localStorage`) with JSON backup capabilities—no server-side database required.

```
┌────────────────────────────────────────────────────────┐
│                   Browser Client                       │
│  - Vanilla JS (ES6+)      - HTML5 Canvas & Webcam API  │
│  - Glassmorphic CSS       - Sandboxed iframe Preview   │
│  - Browser localStorage   - Drag & Drop Upload         │
└───────────────────────────┬────────────────────────────┘
                            │  HTTP (REST API / JSON)
                            ▼
┌────────────────────────────────────────────────────────┐
│                   Flask Backend                        │
│  - Python 3.13 / 3.14     - Local API Guard & CSRF     │
│  - Jinja2 Templates       - Image Enhancement (Pillow) │
│  - In-Memory Preview Cache                             │
└───────────────────────────┬────────────────────────────┘
                            │  Google GenAI SDK
                            ▼
┌────────────────────────────────────────────────────────┐
│             Hosted AI Engine (Google)                  │
│  - Gemini API Client                                   │
│  - Gemma 4 Instruction-Tuned Models                    │
│    (gemma-4-26b-a4b-it / gemma-4-31b-it)               │
└────────────────────────────────────────────────────────┘
```

---

## 2. Backend Stack

| Component | Technology | Version | Purpose |
|---|---|---|---|
| **Language** | Python | 3.13 / 3.14 | Core backend runtime |
| **Web Framework** | Flask | `>=3.1, <4` | HTTP routing, REST API endpoints, template serving |
| **WSGI Toolkit** | Werkzeug | `>=3.1` | Request handling, secure HTTP exception management |
| **AI Client SDK** | `google-genai` | `>=1.50, <3` | Official Google GenAI SDK communicating with Gemini endpoints |
| **Image Processing** | Pillow (PIL) | `>=11.3, <13` | Normalization, contrast enhancement, aspect-ratio scaling, thumbnailing |
| **Environment Config** | `python-dotenv` | `>=1.1, <2` | Secure `.env` loading for API credentials |

### Key Backend Modules
- **`app.py`**: Web routes, API endpoints (`/api/health`, `/api/discover`, `/api/analyze`, `/api/generate`, `/api/edit`, `/api/preview`), and preview sandboxing.
- **`config.py`**: Dynamic runtime discovery of Gemma 4 models, API credential isolation, and `.model.json` caching.
- **`core/image_processing.py`**: Image validation, contrast adjustment, sharpening, and preprocessing before sending to vision models.
- **`engine/vision_ai.py`**: Sketch layout parsing, UI element detection, and annotation extraction into structured JSON specs.
- **`engine/codegen.py`**: Prompt engineering and single-file HTML/CSS/JS code synthesis.
- **`engine/prompts.py`**: Specialized system prompts for layout interpretation and code generation.

---

## 3. Frontend Stack

| Layer | Implementation | Details |
|---|---|---|
| **Structure** | Vanilla HTML5 + Jinja2 | Semantic markup, component templates (`base.html`, `scan.html`, `review.html`, `builder.html`, `projects.html`) |
| **Styling** | Vanilla CSS3 | Custom Design System, **Glassmorphism** aesthetic, CSS custom properties (variables), responsive flex/grid layouts |
| **Interactivity** | Vanilla JavaScript (ES6+) | No framework (no React/Vue/Angular), native asynchronous `fetch()`, modular UI controllers |
| **Hardware Access** | MediaDevices / Canvas API | Native `navigator.mediaDevices.getUserMedia` for real-time webcam sketch capture, snapshot freeze, and canvas rendering |
| **File I/O** | HTML5 Drag & Drop + File API | Drag-and-drop sketch upload, image preview, base64 encoding |
| **Icons** | SVG Vector Assets | Embedded SVG sprite sheet (`static/icons.svg`) for crisp scaling without third-party icon fonts |
| **Build Pipeline** | **None (Zero Node.js)** | Runs directly in modern browsers without Babel, Webpack, Vite, or npm installs |

---

## 4. AI & Vision Stack

- **Provider**: Google AI Studio / Gemini API.
- **Models**:
  - `models/gemma-4-26b-a4b-it` (instruction-tuned, vision & code)
  - `models/gemma-4-31b-it` (fallback candidate)
- **Model Discovery Engine**:
  - Dynamically queries available models associated with the configured API key.
  - Automatically verifies image understanding at runtime using a synthetic test probe image before persisting the selection to `.model.json`.
- **Vision Pipeline**:
  - Sketch Analysis: Raw pencil/paper drawing $\rightarrow$ Layout hierarchy & element list (buttons, forms, cards, navigation).
  - Annotation Extraction: Pen edits $\rightarrow$ Detected modifier instructions (e.g., "make dark mode", "add search bar").
- **Code Synthesis**:
  - Converts detected JSON UI specifications into self-contained, responsive HTML5 + CSS3 + JavaScript prototypes.

---

## 5. Storage & Persistence

- **Client-Side Project Database**: Browser `localStorage`
  - Stores project metadata, sketch snapshots, detected JSON specs, and generated code drafts.
  - Keeps user sketches and code completely private on the user's machine.
- **Backup & Portability**:
  - JSON-based Project Export & Import functionality.
  - Downloadable single-file HTML prototypes.
- **Server Cache**:
  - Ephemeral thread-safe in-memory cache for live preview rendering with unique preview tokens.

---

## 6. Security & Sandboxing Architecture

1. **Credential Isolation**:
   - `GEMINI_API_KEY` lives strictly in backend `.env` and is never exposed to browser templates, network responses, or client JavaScript.
2. **Preview Sandbox**:
   - Generated code renders inside an `<iframe>` governed by strict Content Security Policy (`sandbox="allow-scripts"`).
   - Scripts inside the preview cannot access parent cookies, local storage, or network endpoints.
3. **Local API Guard**:
   - Restricts API traffic strictly to `127.0.0.1` and `localhost`.
   - Requires custom header `X-Napkin-Request: 1` on state-changing requests to block Cross-Site Request Forgery (CSRF).
   - Strict response security headers: `X-Content-Type-Options: nosniff`, `X-Frame-Options: SAMEORIGIN`, `Referrer-Policy: no-referrer`.

---

## 7. Quality Assurance & Testing

- **Unit & Integration Tests**: Python standard `unittest` suite (`tests/test_app.py`, `tests/test_architecture.py`).
- **Regression Suite**: Browser regression checks (`tests/browser_regression.py`) verifying responsive layouts (390px, 768px, 1440px).
- **Mocking**: Simulated AI responses and synthetic camera streams for reliable offline testing without consuming API quota.
