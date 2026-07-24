[README.md](https://github.com/user-attachments/files/30329743/README.md)
# TMDB API — Automated Test Suite

![Postman Tests](https://github.com/Nano277353/postmantest/actions/workflows/postman-tests.yml/badge.svg)

Automated API test suite for [The Movie Database (TMDB)](https://www.themoviedb.org/) REST API, built with **Postman** and executed in CI via **Newman** and **GitHub Actions**.

Tests run automatically on every push, on every pull request, and once daily on a schedule — so any breaking change to the API contract is caught without manual intervention.

---

## Test Coverage

The collection contains two chained requests covering **8 assertions** in total.

### 1. `GET /trending/movie/day` — Trending Movies

| # | Test | Type |
|---|------|------|
| 1 | Status code is 200 | Functional |
| 2 | Response has `results` array | Schema / contract |
| 3 | Response time is under 1000ms | Performance |
| 4 | Each result has `id`, `title`, `vote_average` | Schema / contract |
| 5 | First movie `id` saved to a collection variable | Data extraction |

### 2. `GET /movie/{{movie_id}}` — Movie Details

| # | Test | Type |
|---|------|------|
| 6 | Status code is 200 | Functional |
| 7 | Returned movie `id` matches the requested ID | Data integrity |
| 8 | Response has `title`, `overview`, `release_date`, `runtime` | Schema / contract |

### Request Chaining

The second request is **dependent on the first**. The trending endpoint's test script extracts the first movie's ID and stores it with `pm.collectionVariables.set()`:

```javascript
const firstMovieId = jsonData.results[0].id;
pm.collectionVariables.set("movie_id", firstMovieId);
```

That value is then consumed by the next request as `{{movie_id}}`, and the response is validated against the ID that was actually requested. This verifies a real multi-step workflow rather than testing endpoints in isolation.

---

## Repository Structure

```
.
├── .github/
│   └── workflows/
│       └── postman-tests.yml              # CI pipeline definition
├── postman/
│   ├── tmdb-tests.postman_collection.json # Collection + test scripts
│   └── tmdb-env.postman_environment.json  # Environment variables
└── README.md
```

---

## CI/CD Pipeline

The workflow in `.github/workflows/postman-tests.yml`:

1. Checks out the repository
2. Installs Newman and the `htmlextra` reporter
3. Runs the collection, injecting the API key at runtime from a GitHub Secret
4. Uploads an HTML test report as a build artifact (uploaded even when tests fail, via `if: always()`)

**Triggers:** push to the default branch · pull requests · daily cron at 06:00 UTC

### Secrets Management

No credentials are committed to this repository. The `api_key` value in the environment file is intentionally left blank and is overridden at runtime:

```bash
--env-var "api_key=${{ secrets.TMDB_API_KEY }}"
```

The real key is stored as a GitHub Actions repository secret (`TMDB_API_KEY`), which is never exposed in logs or in version history.

### Viewing Results

Test results are visible in the **Actions** tab. Each run also produces a downloadable HTML report under the run's **Artifacts** section.

---

## Running Locally

**Prerequisites:** [Node.js](https://nodejs.org/) installed.

```bash
# Install Newman and the HTML reporter
npm install -g newman newman-reporter-htmlextra

# Run the collection with your own TMDB API key
newman run postman/tmdb-tests.postman_collection.json \
  -e postman/tmdb-env.postman_environment.json \
  --env-var "api_key=YOUR_TMDB_API_KEY" \
  --reporters cli,htmlextra \
  --reporter-htmlextra-export newman-report/report.html
```

A free TMDB API key can be requested from your [TMDB account settings](https://www.themoviedb.org/settings/api).

Alternatively, import both JSON files into the Postman desktop app, set `api_key` in the environment, and use the Collection Runner.

---

## Tech Stack

| Tool | Purpose |
|------|---------|
| Postman | Test authoring, assertions (Chai syntax), request chaining |
| Newman | CLI test runner for headless / CI execution |
| GitHub Actions | Continuous integration and scheduled monitoring |
| newman-reporter-htmlextra | HTML test reporting |

---

## Attribution

This product uses the TMDB API but is not endorsed or certified by TMDB.
