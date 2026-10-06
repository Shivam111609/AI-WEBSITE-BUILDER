# 🚀 AI Website Builder

Describe the website you want in plain English and let AI build it for you. Type a prompt like *"Create a landing page for a coffee shop"*, watch the project get planned and generated, then preview it live, edit the code, export it, or publish it.

---

## ✨ Features

- 🤖 **Prompt to website**: generate a complete multi-file website from a single text prompt
- 💬 **Chat interface**: iterate on your site by chatting with the AI (versioned as v0, v1, ...)
- 📁 **File explorer**: browse every generated file in the **Files** tab
- 👨‍💻 **Code view**: inspect and copy the generated source code
- 👀 **Live preview**: open the generated site in a separate tab
- 🌐 **Publish**: make your site publicly accessible
- 📦 **Export**: download the project as a ZIP
- 🔐 **Authentication**: secure sign up / login with cookie-based sessions
- 🗂️ **Project history**: all your projects are stored in MongoDB

---

## 🛠️ Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | React, Vite, Tailwind CSS |
| Backend | Node.js, Express (ES Modules) |
| Database | MongoDB Atlas with Mongoose |
| Auth | Cookie-based sessions (`cookie-parser`) |
| AI | LLM via Vercel AI SDK (structured output with `generateObject`) |

> Adjust this table to match the exact libraries and AI provider you use.

---

## 📂 Project Structure

```
AI_Website_Builder/
├── client/                  # React frontend
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   └── ...
│   └── package.json
│
├── server/                  # Express backend
│   ├── config/
│   │   └── db.js            # MongoDB connection
│   ├── routes/
│   │   ├── authRoutes.js    # /api/auth
│   │   └── projectRoutes.js # /api/projects
│   ├── controllers/
│   ├── models/
│   ├── server.js            # Entry point
│   ├── .env                 # Environment variables (not committed)
│   └── package.json
│
└── README.md
```

---

## ✅ Prerequisites

- [Node.js](https://nodejs.org/) v18 or higher (v20/v22 LTS recommended)
- A [MongoDB Atlas](https://www.mongodb.com/atlas) account (free M0 cluster works)
- An API key from your AI provider (OpenAI, Google Gemini, OpenRouter, etc.)
- Git

---

## ⚙️ Installation

### 1. Clone the repository

```bash
git clone https://github.com/<your-username>/AI_Website_Builder.git
cd AI_Website_Builder
```

### 2. Install server dependencies

```bash
cd server
npm install
```

### 3. Install client dependencies

```bash
cd ../client
npm install
```

---

## 🔑 Environment Variables

### Server (`server/.env`)

```env
PORT=3000

# MongoDB Atlas connection string
MONGODB_URI=mongodb+srv://<username>:<password>@cluster0.xxxxx.mongodb.net/ai-website-builder

# Comma-separated list of allowed frontend origins (for CORS)
ORIGINS=http://localhost:5173

# Secret used to sign auth tokens/sessions
JWT_SECRET=your_super_secret_key

# AI provider API key
AI_API_KEY=your_ai_api_key

# AI model used for website generation (e.g. gpt-4o-mini, gemini-2.0-flash)
AI_MODEL=your_model_name
```

> Use a model that supports structured (JSON) output reliably, otherwise generation may fail with `No object generated`.

### Client (`client/.env`)

```env
VITE_BASE_URL=http://localhost:3000
```

> ⚠️ Never commit `.env` files. Make sure `.env` is listed in `.gitignore`.

---

## ▶️ Running the Project

Open two terminals.

**Terminal 1: Backend**

```bash
cd server
node server.js
# or, with auto-reload:
npx nodemon server.js
```

You should see:

```
Server is running at http://localhost:3000
```

**Terminal 2: Frontend**

```bash
cd client
npm run dev
```

Open **http://localhost:5173** in your browser.

---

## 🧭 How It Works

1. **Sign up / log in** to your account.
2. **Enter a prompt** describing the website you want.
3. The backend sends the prompt to the AI model, which first **plans the project structure**.
4. The AI generates the files (HTML, CSS, JS/React) as a structured JSON object.
5. Files are saved to the database as a new project version.
6. Use **Preview**, **Code**, **Files**, **Publish**, or **Export** to work with the result.
7. Keep chatting to refine the site; each change creates a new version.

---

## 🔌 API Endpoints

### Auth: `/api/auth`

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/register` | Create a new account |
| POST | `/login` | Log in |
| POST | `/logout` | Log out |
| GET | `/me` | Get the current user |

### Projects: `/api/projects`

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/` | List all your projects |
| POST | `/` | Create a project / generate a website |
| GET | `/:id` | Get a single project |
| POST | `/:id/chat` | Send a follow-up prompt |
| POST | `/:id/publish` | Publish the project |
| GET | `/:id/export` | Download the project |

> Update these routes to match your actual `authRoutes.js` and `projectRoutes.js`.

---

## 🩺 Troubleshooting

### `querySrv ECONNREFUSED _mongodb._tcp.cluster0...`

Your ISP/router DNS is blocking MongoDB's SRV lookup.

**Fix (in `server.js`, before connecting to the DB):**

```js
import dns from "node:dns";
dns.setServers(["8.8.8.8", "1.1.1.1"]);
```

Alternatively, set Windows DNS to `8.8.8.8` / `1.1.1.1`, or use the non-SRV `mongodb://` connection string from Atlas (Connect → Drivers → "2.2.12 or later").

### `tlsv1 alert internal error` / `Could not connect to any servers`

Your IP is not whitelisted in Atlas.

**Fix:** Atlas → Security → Network Access → Add IP Address → `0.0.0.0/0` (for development) and wait until the status is **Active**.

Also check that your cluster is not paused.

### `Generation failed: No object generated: could not parse the response`

The AI model returned something that is not valid JSON for the expected schema.

**Fixes:**

- Use a model with reliable structured-output support (e.g. `gpt-4o-mini`, `gemini-2.0-flash`, Claude Sonnet). Avoid tiny or free models.
- Increase `maxOutputTokens` (the response may be getting cut off).
- Simplify the output schema.
- Log `error.text` and `error.finishReason` in your controller to see what the model actually returned.

### CORS errors in the browser

Make sure the frontend URL is included in `ORIGINS` in `server/.env`, with no trailing slash:

```env
ORIGINS=http://localhost:5173,https://your-frontend.vercel.app
```

---

## 🚢 Deployment

- **Frontend**: Vercel or Netlify (set `VITE_BASE_URL` to your backend URL)
- **Backend**: Render, Railway, or any Node host (set all `.env` variables in the dashboard)
- **Database**: MongoDB Atlas (restrict Network Access to your server's IP in production)

When deploying, update `ORIGINS` to include your production frontend URL, and make sure cookies are configured with `secure: true` and a suitable `sameSite` value for cross-site requests.

---

## 🗺️ Roadmap

- [ ] Image generation for website assets
- [ ] Custom domain support for published sites
- [ ] Version history with rollback
- [ ] Template gallery
- [ ] Team collaboration

---

## 🤝 Contributing

1. Fork the repo
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Commit your changes: `git commit -m "Add your feature"`
4. Push to the branch: `git push origin feature/your-feature`
5. Open a Pull Request

---

## 📄 License

This project is licensed under the [MIT License](LICENSE).

---

## 👤 Author

**Raj**
GitHub: [@your-username](https://github.com/your-username)

⭐ If you found this project useful, give it a star!
