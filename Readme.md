# Pastebin-Lite

A lightweight pastebin application that allows users to create, share, and view text pastes with optional expiry times and view limits.

## Features

- Create text pastes with unique shareable URLs
- Optional time-to-live (TTL) expiry
- Optional view-count limits
- Clean, responsive UI
- XSS-safe HTML rendering
- Deterministic time testing support

## Tech Stack

- **Runtime**: Node.js with Express.js
- **Deployment**: Vercel (serverless)
- **Database**: Vercel KV (Redis)
- **ID Generation**: nanoid

## Persistence Layer

This application uses **Vercel KV** (Redis) as its persistence layer. Vercel KV is a durable, serverless Redis database that works seamlessly with Vercel's serverless functions.

### Why Vercel KV?

- **Serverless-friendly**: Persists data across function invocations
- **Low latency**: Fast read/write operations
- **Simple setup**: Integrates directly with Vercel projects
- **Scalable**: Handles concurrent requests efficiently

### Data Model

Pastes are stored with the following structure:
```
Key: paste:<id>
Value: {
  content: string,
  maxViews: number | null,
  remainingViews: number | null,
  expiresAt: number | null,
  createdAt: number
}
```

## Running Locally

### Prerequisites

- Node.js 18+ installed
- npm or yarn package manager
- Vercel account (for KV database)

### Setup Steps

1. **Clone the repository**
   ```bash
   git clone <your-repo-url>
   cd pastebin-lite
   ```

2. **Install dependencies**
   ```bash
   npm install
   ```
   
   This will install all required packages and generate `package-lock.json` automatically.

3. **Set up Vercel KV**
   
   You have two options:

   **Option A: Use Vercel CLI (Recommended for local development)**
   ```bash
   # Install Vercel CLI
   npm install -g vercel
   
   # Login and link project
   vercel login
   vercel link
   
   # Pull environment variables (including KV credentials)
   vercel env pull .env.local
   ```

   **Option B: Manual setup**
   - Create a KV database in your Vercel dashboard (Storage → Create Database → KV)
   - Copy the environment variables from the KV dashboard (.env.local tab)
   - Create a `.env` file in the project root:
   ```
   KV_REST_API_URL=your-kv-url
   KV_REST_API_TOKEN=your-kv-token
   KV_REST_API_READ_ONLY_TOKEN=your-readonly-token
   ```

4. **Run the development server**
   ```bash
   npm run dev
   ```
   
   The server will start on port 3000 (or the next available port).

5. **Access the application**
   - Open http://localhost:3000 in your browser
   - The API will be available at http://localhost:3000/api/*

### Troubleshooting

- **KV Connection Issues**: Ensure your environment variables are correctly set
- **Port Already in Use**: The app will try the next available port automatically
- **Module Not Found**: Delete `node_modules` and run `npm install` again

## API Endpoints

### Health Check
```
GET /api/healthz
Response: { "ok": true }
```

### Create Paste
```
POST /api/pastes
Body: {
  "content": "string",
  "ttl_seconds": 60,    // optional
  "max_views": 5        // optional
}
Response: {
  "id": "abc123",
  "url": "https://your-app.vercel.app/p/abc123"
}
```

### Get Paste (JSON)
```
GET /api/pastes/:id
Response: {
  "content": "string",
  "remaining_views": 4,
  "expires_at": "2026-01-01T00:00:00.000Z"
}
```

### View Paste (HTML)
```
GET /p/:id
Returns: HTML page with paste content
```

## Design Decisions

### 1. View Counting
Both API fetches (`/api/pastes/:id`) and HTML views (`/p/:id`) decrement the view count. This ensures consistency and prevents bypassing the view limit.

### 2. XSS Protection
All user content is escaped using a custom `escapeHtml()` function before rendering in HTML pages, preventing script injection attacks.

### 3. Deterministic Time Testing
The application supports the `x-test-now-ms` header when `TEST_MODE=1` is set, allowing automated tests to verify time-based expiry without waiting.

### 4. Error Handling
- Invalid inputs return 400 with JSON error messages
- Missing or expired pastes return 404
- All errors include descriptive messages for debugging

### 5. Serverless Architecture
The application is structured as a single Express app that Vercel deploys as serverless functions. All routes are handled by one function for simplicity.

### 6. ID Generation
Uses `nanoid` with 10 characters for short, URL-safe IDs with sufficient collision resistance.

## Deployment

### Deploy to Vercel

1. **Install Vercel CLI**
   ```bash
   npm install -g vercel
   ```

2. **Login to Vercel**
   ```bash
   vercel login
   ```

3. **Deploy**
   ```bash
   vercel --prod
   ```

4. **Add KV Database**
   - Go to your project in Vercel Dashboard
   - Navigate to Storage → Create Database → KV
   - Connect it to your project
   - Redeploy if needed

### Environment Variables

The following environment variables are automatically set by Vercel when you connect KV:
- `KV_REST_API_URL`
- `KV_REST_API_TOKEN`
- `KV_REST_API_READ_ONLY_TOKEN`

For testing, you can also set:
- `TEST_MODE=1` (enables deterministic time testing)

## Testing

### Manual Testing

```bash
# Health check
curl https://your-app.vercel.app/api/healthz

# Create paste
curl -X POST https://your-app.vercel.app/api/pastes \
  -H "Content-Type: application/json" \
  -d '{"content":"Hello World","max_views":2}'

# Get paste
curl https://your-app.vercel.app/api/pastes/<id>

# Test expiry with deterministic time
curl https://your-app.vercel.app/api/pastes/<id> \
  -H "x-test-now-ms: 1735689600000"
```

### Automated Tests

The application is designed to pass automated tests that verify:
- Health check endpoint
- Paste creation and retrieval
- View limits enforcement
- TTL expiry with deterministic time
- Error handling
- Concurrent request handling

## Project Structure

```
pastebin-lite/
├── api/
│   └── index.js          # Main Express application
├── .env                  # Local environment variables (not committed)
├── .gitignore           # Git ignore rules
├── package.json         # Project dependencies and scripts
├── package-lock.json    # Locked dependency versions (auto-generated)
├── vercel.json          # Vercel deployment configuration
└── README.md            # This file
```

### Important Files

- **package-lock.json**: Auto-generated when you run `npm install`. Commit this file to ensure everyone uses the same dependency versions.
- **.env**: Contains your KV credentials. Never commit this file.
- **api/index.js**: The entire application logic in one file for simplicity.

## License

MIT
