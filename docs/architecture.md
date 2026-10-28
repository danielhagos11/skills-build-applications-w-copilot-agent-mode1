# OctoFit Tracker architecture

OctoFit Tracker is a three-tier application for a school fitness program. The starter code includes API contracts and intentionally incomplete implementation points so you can use GitHub Copilot agent mode in a test-driven loop.

![Three-tier OctoFit architecture](../.github/images/octofit-architecture.svg)

## Tier boundaries

- **Presentation tier**: React 19, Vite, React Router, Bootstrap, and a shared API client in `octofit-tracker/frontend/src`.
- **Logic tier**: Express and TypeScript routes/services in `octofit-tracker/backend/src` expose `/api/*` behavior on port `8000`.
- **Data tier**: Mongoose models and seed data in `octofit-tracker/backend/src/models` and `src/scripts/seed.ts` persist to MongoDB on port `27017`.

## Ports and environment rules

- MongoDB: `27017`
- Express API: `8000`
- Vite frontend: `5173`
- The browser calls relative `/api` paths. Vite proxies them to `http://localhost:8000`, so the unauthenticated API port stays private in Codespaces. `VITE_API_BASE_URL` can override the base URL.

## API contract

| Capability | Method and path | Expected behavior |
| --- | --- | --- |
| Activity tracking | `GET /api/activities`, `POST /api/activities` | List and create valid activity records. POST accepts `userId`, `teamId`, `type`, and `minutes`; `userId` and `teamId` must be well-formed IDs for an existing user and team; otherwise respond with a deliberate 4xx (never a 500 from a cast error). A created activity stores the submitted `user`, `team`, `type`, and `minutes` plus computed `points` and `occurredAt`. |
| Teams | `GET /api/teams`, `POST /api/teams` | List and create teams with members. Created and listed teams keep the submitted `name` and `mascot`. Reject a team without a name with `400`. |
| Leaderboard | `GET /api/leaderboard` | Return teams ordered by the sum of their activity points, highest first. Each entry has `teamName`, `totalPoints`, and a 1-based `rank`. |
| Workout suggestions | `GET /api/workouts/suggestions/:userEmail` | Recommend workouts from recent activity patterns for the matching seeded user's email address. The first suggestion's `focus` matches that user's goal or recent activity, with a `reason` explaining why. An email with no matching user returns a deliberate 4xx (for example, `404`). |

## Implementation decisions

Record your plan here before editing code.

- Presentation tier decision: Keep route components focused on presentation and use the shared API client for relative `/api` requests. Each capability view will expose accessible loading, empty, success, and error states.
- Logic tier decision: Keep request validation and business rules in explicit service functions called by Express routes. Return deliberate 4xx responses for invalid identifiers, missing records, and invalid payloads.
- Data tier decision: Use Mongoose schemas for required fields and relationships, and use stable upserts in the seed script without clearing unrelated application data.
- Activity tracking approach: Validate `userId` and `teamId`, confirm both records exist, calculate points from the submitted activity, and store the linked references, type, minutes, points, and `occurredAt`.
- Team management approach: Require a team name, preserve the submitted mascot and members, and return created and listed teams using the established API response shape.
- Leaderboard approach: Aggregate activity points by team, join team names, sort totals in descending order, and assign 1-based ranks in the service layer.
- Workout suggestions approach: Resolve the user by email, use the user's goal and recent activity to prioritize matching workouts, include a reason for the first suggestion, and return a deliberate 4xx for an unknown user.

## Test plan

Record the commands you run and what each failure means.

- Data-tier command: `npm run test:data --prefix octofit-tracker/backend` — verifies required Mongoose fields, relationships, and idempotent representative seed data.
- API command: `npm run test:api --prefix octofit-tracker/backend` — verifies request validation, deliberate 4xx responses, activity and team creation, leaderboard ordering, and user-specific workout suggestions.
- Frontend command: `npm test --prefix octofit-tracker/frontend` — verifies API-backed rendering and accessible loading, empty, success, and error states for every route.
- Backend build command: `npm run build --prefix octofit-tracker/backend` — verifies the backend TypeScript type-checks without emitting build artifacts.
- Frontend build command: `npm run build --prefix octofit-tracker/frontend` — verifies the frontend TypeScript type-checks and Vite produces a production bundle.
