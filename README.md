# Mini Tournament Hub Docs

## Project Overview

Mini Tournament Hub is a backend for live text-response tournaments. Users can register, create public or private tournaments, invite participants, run multiple rounds, and receive live updates during submission and voting phases.

The MVP describes a **multi-round tournament without elimination**:

- participants submit one text response in the current round;
- voting is performed sequentially over other participants' submissions;
- a missed vote is automatically counted as `DISLIKE`;
- round results produce a cumulative leaderboard;
- phase and round transitions are handled by the backend;
- live state is delivered through Socket.IO, while the REST snapshot remains the authoritative recovery source.

## Repository Contents

```text
docs/
  mvp requirements.pdf
  git flow.pdf
  websocket integration.pdf
  tech/
    product overview.pdf
    business rules.pdf
    data model.pdf
    rest api.pdf
    websocket api.pdf
    architecture and operations.pdf
    limitations.pdf
tracker.png
```

## Documents

### Requirements and Product

- [MVP Requirements](docs/mvp%20requirements.pdf) - MVP boundaries, core user scenarios, phase rules, tournament constraints, and the recovery contract.
- [Product Overview](docs/tech/product%20overview.pdf) - product summary, roles, MVP scope, excluded scope, and the main user flow.
- [Business Rules and Lifecycle](docs/tech/business%20rules.pdf) - tournament lifecycle, statuses, allowed actions, participation rules, submissions, and voting.
- [Limitations](docs/tech/limitations.pdf) - known MVP limitations and functionality intentionally left outside the current implementation.

### Technical Documentation

- [Data Model](docs/tech/data%20model.pdf) - core domain entities: users, tournaments, participants, rounds, submissions, votes, and prompts.
- [REST API](docs/tech/rest%20api.pdf) - REST endpoints, base path `/api/v1`, auth, users, tournaments, submissions, votes, and recovery endpoints.
- [WebSocket API](docs/tech/websocket%20api.pdf) - Socket.IO handshake, room format `tournament:{tournamentId}`, client/server events, and presence.
- [Architecture and Operations](docs/tech/architecture%20and%20operations.pdf) - backend modules, services, real-time delivery pipeline, BullMQ jobs, Redis presence, and PostgreSQL consistency.
- [WebSocket Integration](docs/websocket%20integration.pdf) - the selected real-time communication approach: REST for mutations and snapshots, WebSocket for server-pushed events.

### Development Process

- [Git Flow](docs/git%20flow.pdf) - branch, commit, and merge request naming rules, with changes linked to tracker tasks.
- [Tracker snapshot](tracker.png) - a screenshot of the migrated tracker with project tasks.

## Architecture

The project uses a hybrid REST + WebSocket approach:

- **REST** handles authentication and state-changing operations: create, join, leave, start tournament, submit answer, vote, and request recovery snapshots.
- **Socket.IO** delivers live events to clients subscribed to a tournament room.
- **PostgreSQL** stores the authoritative domain state.
- **Redis** is used for presence and real-time infrastructure scenarios.
- **BullMQ** is used for asynchronous tournament event delivery.

After reconnecting, the client must reconnect to Socket.IO, emit `tournament:join`, and fetch the current state through a REST snapshot. WebSocket event history is not stored or replayed.

## Main Flow

1. A user registers or logs in.
2. The user creates a draft tournament or joins an existing one.
3. Private tournaments use an invite token.
4. The owner starts the tournament when the minimum participant count is reached.
5. The round enters `SUBMISSION`, and participants submit text answers.
6. The round enters `VOTING`, and the backend reveals submissions sequentially.
7. Participants vote `LIKE` or `DISLIKE`.
8. The backend finalizes voting, calculates results, and updates the leaderboard.
9. The next round is created, or the tournament is completed.
