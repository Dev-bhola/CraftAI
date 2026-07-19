# CraftAI

A full-stack AI content creation platform with tools for writing, image generation, and image editing.

## About

CraftAI is a web application that bundles multiple AI-powered content tools behind a single dashboard. Users can generate articles, blog titles, and images, as well as remove backgrounds or objects from photos and get AI-driven resume reviews. The app uses a freemium model — free-tier users get 10 generations, while premium users unlock unlimited access to all tools and image features.

## Key Features

- **AI Article Writer** — Generate articles on any topic with configurable length (short / medium / long), powered by Gemini 2.0 Flash
- **Blog Title Generator** — Produce blog title suggestions by keyword and category (Technology, Business, Health, Lifestyle, etc.)
- **AI Image Generation** — Text-to-image generation via the Clipdrop API with style presets (Realistic, Ghibli, Anime, Cartoon, Fantasy, 3D, Portrait)
- **Background Removal** — Upload an image and remove its background using Cloudinary's AI transformations
- **Object Removal** — Specify an object name and remove it from an uploaded image via Cloudinary's `gen_remove` effect
- **Resume Reviewer** — Upload a PDF resume and receive AI-generated feedback on strengths, weaknesses, and areas for improvement
- **Community Gallery** — Publish generated images to a shared gallery; browse and like creations from other users
- **User Dashboard** — View all past creations (articles, titles, images, reviews) with total count and active plan status
- **Authentication & Authorization** — Clerk-based auth with free/premium plan gating enforced on both client and server
- **Freemium Usage Tracking** — Free-tier users are capped at 10 AI generations; usage count is stored in Clerk user metadata

## Tech Stack

### Frontend

| Technology | Purpose |
|---|---|
| React 19 | UI framework |
| Vite 7 | Build tool & dev server |
| Tailwind CSS 4 | Utility-first styling |
| React Router 7 | Client-side routing |
| Clerk React | Authentication UI & session management |
| Axios | HTTP client |
| React Markdown | Rendering AI-generated markdown content |
| Lucide React | Icon library |
| React Hot Toast | Toast notifications |

### Backend

| Technology | Purpose |
|---|---|
| Express 5 | HTTP server framework |
| Clerk Express | Server-side auth middleware & user metadata |
| Neon (Serverless Postgres) | Database via `@neondatabase/serverless` |
| OpenAI SDK (Gemini) | AI text generation (Gemini 2.0 Flash via OpenAI-compatible API) |
| Cloudinary | Image hosting, background removal, object removal |
| Clipdrop API | Text-to-image generation |
| Multer | File upload handling |
| pdf-parse | PDF text extraction for resume review |

## Setup & Installation

### Prerequisites

- Node.js (v18 or later recommended)
- A [Clerk](https://clerk.com) account (with a premium plan configured if testing premium features)
- A [Neon](https://neon.tech) Postgres database
- A [Cloudinary](https://cloudinary.com) account (with AI add-ons enabled)
- A [Clipdrop](https://clipdrop.co) API key
- A [Google AI (Gemini)](https://aistudio.google.com/apikey) API key

### 1. Clone the repository

```bash
git clone https://github.com/<your-username>/CraftAI.git
cd CraftAI
```

### 2. Set up the database

Create a `creations` table in your Neon Postgres database:

```sql
CREATE TABLE creations (
  id SERIAL PRIMARY KEY,
  user_id TEXT NOT NULL,
  prompt TEXT NOT NULL,
  content TEXT NOT NULL,
  type TEXT NOT NULL,
  publish BOOLEAN DEFAULT false,
  likes TEXT[] DEFAULT '{}',
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

### 3. Configure environment variables

**Server** — create `server/.env`:

```env
PORT=3000
DATABASE_URL=postgresql://<user>:<password>@<host>/<database>?sslmode=require
CLERK_SECRET_KEY=sk_test_...
GEMINI_API_KEY=your_gemini_api_key
CLIPDROP_API_KEY=your_clipdrop_api_key
CLOUDINARY_CLOUD_NAME=your_cloud_name
CLOUDINARY_API_KEY=your_cloudinary_api_key
CLOUDINARY_API_SECRET=your_cloudinary_api_secret
```

**Client** — create `client/.env`:

```env
VITE_CLERK_PUBLISHABLE_KEY=pk_test_...
VITE_BASE_URL=http://localhost:3000
```

### 4. Install dependencies

```bash
# Server
cd server
npm install

# Client
cd ../client
npm install
```

### 5. Run the application

```bash
# Terminal 1 — Start the backend
cd server
npm run server     # uses nodemon for hot-reload

# Terminal 2 — Start the frontend
cd client
npm run dev        # Vite dev server (default: http://localhost:5173)
```

## Folder Structure

```
CraftAI/
├── client/                     # React frontend (Vite)
│   ├── public/                 # Static assets
│   ├── src/
│   │   ├── assets/             # Images, icons, and static data (AiToolsData, testimonials)
│   │   ├── components/         # Reusable UI components
│   │   │   ├── AiTools.jsx     # Landing page tool cards
│   │   │   ├── CreationItem.jsx# Dashboard creation list item
│   │   │   ├── Hero.jsx        # Landing page hero section
│   │   │   ├── Navbar.jsx      # Top navigation bar
│   │   │   ├── SideBar.jsx     # Dashboard sidebar navigation
│   │   │   ├── Footer.jsx      # Landing page footer
│   │   │   ├── Plan.jsx        # Pricing section
│   │   │   └── Testimonial.jsx # Testimonials carousel
│   │   ├── pages/              # Route-level page components
│   │   │   ├── Home.jsx        # Landing page
│   │   │   ├── Layout.jsx      # Authenticated dashboard shell (sidebar + outlet)
│   │   │   ├── Dashboard.jsx   # User creations overview
│   │   │   ├── WriteArticle.jsx
│   │   │   ├── BlogTitles.jsx
│   │   │   ├── GenerateImages.jsx
│   │   │   ├── RemoveBackground.jsx
│   │   │   ├── RemoveObject.jsx
│   │   │   ├── ReviewResume.jsx
│   │   │   └── Community.jsx   # Public image gallery with likes
│   │   ├── App.jsx             # Route definitions
│   │   └── main.jsx            # Entry point (ClerkProvider + BrowserRouter)
│   ├── index.html
│   └── vite.config.js
│
└── server/                     # Express backend
    ├── configs/
    │   ├── cloudinary.js       # Cloudinary SDK configuration
    │   ├── db.js               # Neon Postgres connection
    │   └── multer.js           # Multer file upload config
    ├── controllers/
    │   ├── aiController.js     # AI feature handlers (article, title, image, bg removal, object removal, resume)
    │   └── userController.js   # User creations, published gallery, like toggle
    ├── Middlewares/
    │   └── auth.js             # Plan detection & free-usage tracking middleware
    ├── Routes/
    │   ├── aiRoutes.js         # /api/ai/* routes
    │   └── userRoutes.js       # /api/user/* routes
    ├── server.js               # Express app entry point
    └── package.json
```

## Future Improvements

- **Streaming responses** — Stream article generation in real-time instead of waiting for the full response
- **Creation history search & filters** — Allow users to filter dashboard creations by type, date, or keyword
- **Image download** — Add a one-click download button for generated / processed images
- **Rate limiting** — Add server-side rate limiting to protect API endpoints
- **Tests** — Add unit and integration tests for backend controllers and middleware
- **Error handling** — Return proper HTTP status codes (currently all responses use `200` with a `success` boolean)
