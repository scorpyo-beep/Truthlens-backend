# TruthLens Backend API

Node.js/Express API for the TruthLens GitHub Pages front end.

## What it does

- `GET /health` — health and configuration status
- `POST /api/check` — runs a Google Fact Check Tools API lookup (if configured) and AI-assisted web research using the OpenAI Responses API with web search
- Returns structured claim-level verdicts, evidence summaries, source links, limitations, and confidence levels
- Rate limiting, basic security headers, request-size limits, optional origin allowlist
- Secrets are read from environment variables; no secrets are committed

## Deploy to Render

1. Create a **new GitHub repository** named `truthlens-backend`.
2. Upload all files in this folder to the repository root: `server.js`, `package.json`, `render.yaml`, `.env.example`, and `README.md`.
3. Go to Render and choose **New → Web Service**, then connect your `truthlens-backend` repository.
4. Use:
   - Runtime: Node
   - Build command: `npm install`
   - Start command: `npm start`
   - Health check path: `/health`
5. In Render → your service → **Environment**, set:
   - `OPENAI_API_KEY` = your OpenAI API key
   - `ALLOWED_ORIGINS` = your exact GitHub Pages origin, e.g. `https://YOUR-GITHUB-USERNAME.github.io` (origin only, no repository path)
   - `GOOGLE_FACT_CHECK_API_KEY` = optional Google Fact Check Tools API key
   - `OPENAI_MODEL` = model available to your API account (default `gpt-4.1-mini`)
6. Deploy and wait for the service to become live.
7. Visit `https://YOUR-SERVICE.onrender.com/health`. It should return JSON with `ok: true` and configuration flags.

**Important:** Never paste API keys into GitHub files, screenshots, or this chat. Enter them only in the backend host's environment settings. API usage may cost money; set usage limits in the provider account.

## Connect the GitHub Pages front end

In the TruthLens website, set the API base URL to your Render URL, for example:
`https://truthlens-api.onrender.com`

The front end must send:
```json
{
  "text": "Exact claim text to research",
  "category": "News article",
  "sourceUrl": ""
}
```
to `POST https://YOUR-SERVICE.onrender.com/api/check` with `Content-Type: application/json`.

The response contains `analysis` with:
- `overallVerdict`
- `confidence`
- `summary`
- `importantCaveat`
- `claims[]` with verdict, explanation, supporting and contradicting evidence, and source URLs
- `sourceAssessment`
- `researchSteps`
- `existingFactChecks[]` from Google's Fact Check Tools API when configured

## Notes and limitations

- OpenAI's web search is used to discover sources. The model may still miss evidence or misinterpret it; check each source yourself.
- Google Fact Check Tools API is optional and only returns fact-check reviews that have been published and indexed. It is not a universal truth database.
- This initial version does not download arbitrary submitted web pages. A later URL extraction service should include SSRF protections and strict host/network safeguards.
- This API is intended as a starter. Before a public launch, add authentication, stronger abuse controls, request logging with privacy protections, monitoring, automated tests, and a reviewed privacy policy.
- Do not use a single source trust score as a proxy for truth or political neutrality.
