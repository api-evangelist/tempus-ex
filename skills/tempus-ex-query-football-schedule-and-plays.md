---
name: Query football schedules, games and plays
description: >-
  Use the FusionFeed GraphQL API to enumerate NFL/NCAA games for a season and
  search plays within a game using Fusion Query Language (FQL).
api: https://feed.fusion.tempus-ex.com/v2/graphql
operations: [nfl.schedule, ncaa.football.schedule, playsConnection, node]
auth: 'Authorization: token <API_KEY>  (or Bearer <JWT>)'
---

# Query football schedules, games and plays

FusionFeed is GraphQL-first. POST GraphQL to
`https://feed.fusion.tempus-ex.com/v2/graphql` with an `Authorization: token <API_KEY>`
header (or `Bearer <JWT>`). Always name your operations and pass dynamic inputs
as GraphQL variables (never string-interpolate). Connections are Relay-style
(`edges { node { ... } }`, `first`/`last`).

## Steps

1. **List games for a season.** For the NFL, query the schedule:
   ```graphql
   query NFLSchedule($season: Int!) {
     nfl {
       schedule(season: $season, type: REGULAR) {
         games { id time week }
       }
     }
   }
   ```
   For NCAA, scope by conference: `ncaa { football(conference: PAC12) { schedule(season: $season, type: REGULAR) { games { id week } } } }`.

2. **Fetch one game by id.** Game ids are opaque (e.g. `CA~...`). Resolve with the
   top-level `node` field and an inline fragment:
   ```graphql
   query Game($id: ID!) { node(id: $id) { ... on NFLGame { id time week } } }
   ```

3. **Search plays with FQL.** Use `playsConnection` with a `predicate` expression:
   ```graphql
   query Touchdowns {
     nfl {
       playsConnection(first: 500, predicate: { expression: "team(\"NYJ\") has TOUCHDOWN" }) {
         edges { node { quarter time description } }
       }
     }
   }
   ```
   The REST equivalent is `GET /v2/nfl/plays?first=500&predicate=<url-encoded-json>`.

## Rules
- Rate limits: a 429 (HTTP) or a GraphQL error means back off exponentially; watch the `txm-limit-consumption` response header.
- Round time-based variables (e.g. to the nearest second) to maximize edge cache hits.
- Version is pinned in the path (`/v2/`); no breaking changes occur within a major version.
- See conventions/tempus-ex-conventions.yml and errors/tempus-ex-problem-types.yml.
