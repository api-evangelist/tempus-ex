---
name: Find, clip and thumbnail game video
description: >-
  Use the FusionFeed GraphQL API to list a game's audio/video streams, get HLS
  manifest URLs, generate time-bounded clips and download URLs, and fetch
  preview thumbnails.
api: https://feed.fusion.tempus-ex.com/v2/graphql
operations: [avStreamsConnection, hlsManifestURL, downloadURL, av-streams-preview]
auth: 'Authorization: token <API_KEY>  (or Bearer <JWT>)'
---

# Find, clip and thumbnail game video

FusionFeed exposes video primarily over HLS (~10s latency; SRT on request).
Authenticate every request with `Authorization: token <API_KEY>`.

## Steps

1. **List a game's streams.** Resolve the game `node` and read `avStreamsConnection`:
   ```graphql
   query Streams($id: ID!) {
     node(id: $id) {
       ... on NFLGame {
         avStreamsConnection(first: 100) {
           edges { node { id name contentDescriptor hlsManifestURL } }
         }
       }
     }
   }
   ```
   Identify a stream by the stable `contentDescriptor` (e.g. `"Program"`), not the
   human-facing `name`.

2. **Inspect renditions.** Read `variantsConnection` to see available `bitrate` /
   `tracks { rfc6381Codec, height }` before choosing a variant.

3. **Clip a time window.** Pass `time` (RFC 3339) and `durationSeconds` to
   `hlsManifestURL(time: "2021-09-10T00:24:10Z", durationSeconds: 30)`; use
   `downloadURL(time:..., durationSeconds:...)` for a downloadable file.

4. **Thumbnails (REST).** `GET /v2/av-streams/{streamId}/preview.jpg?t=<RFC3339>&fit=640x360`.
   This endpoint requires auth, so web apps must fetch it and render via a data URI
   (it cannot be used directly in an `<img src>`).

5. **Live push (optional).** For realtime data during a game, open a WebSocket to
   `/v2/graphql-ws` (graphql-ws) and send `{"authorization":"token <API_KEY>"}` in
   the connection-init params. See asyncapi/tempus-ex-fusionfeed-asyncapi.yml.

## Rules
- HLS playlists carry `#EXT-X-PROGRAM-DATE-TIME` for precise sync with telemetry/stats.
- Camera calibration (`projectionViewMatrix`) enables AR overlays for fixed and moving cameras.
- Respect rate limits (429 / `txm-limit-consumption`) with exponential backoff.
