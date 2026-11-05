# Restream Context: Streams, Recordings & Clipping (handoff)

_Snapshot taken 2026-11-05 through the Restream MCP server._

## TL;DR
- Account-level clipping settings (clip length, aspect ratio, captions, branding, auto-clip defaults) are **not exposed** by the Restream MCP. They can only be viewed or changed in the Restream web app.
- The account has **1 clips project** with **4 AI-generated clips**. None have been posted to platforms.
- The account streams two recurring shows (Dojo Coding Studio, Deporte+) to 22 channels. Recordings expire after ~15 days.
- `remotion-media-mcp` has **no Restream integration** yet.

## Account
| Field | Value |
|---|---|
| Organization | de+ |
| Workspace | Default |
| Clips projects | 1 |

## Existing clips project
| Field | Value |
|---|---|
| Title | Stream via RTMP (OBS, Vmix, Zoom) with Restream |
| Source type | Event (recording of an RTMP stream) |
| Created | 2025-11-29 |
| Generating now | No |
| Clips | 4 (0 posted, 0 favourited) |
| Language / topic | Spanish. A talk about Costa Rican football: foreign coaches, youth development, federation leadership |

### Clips
| # | Title | Virality |
|---|---|---|
| 1 | La federación "anti-tica": por qué se apuesta por entrenadores extranjeros a pesar de los malos resultados | 88 |
| 2 | Padres y agentes que arruinan carreras: cómo el "éxito rápido" está destruyendo a las promesas del fútbol juvenil | 88 |
| 3 | Por qué entrenadores extranjeros fracasan en Costa Rica y México: adaptarse o morir | 86 |
| 4 | Traer entrenadores de élite para capacitar técnicos costarricenses: ¿una inversión que transformaría el fútbol? | 84 |

Notes:
- The API returns `sourceStartSeconds` / `sourceEndSeconds` = null and `viralityScoreExplanation` = null for all clips.
- Each clip includes a full transcript and a thumbnail URL. Fetch them with `get_user_clips_project_details`.

## Streams (events)
- The account has **241 finished events** in total. It has **0 scheduled** events right now.
- Every recent event uses **Studio** as its source.
- There are two recurring shows:
  - **Dojo Coding Studio**: about twice a week, starting ~01:00 UTC (~7 pm Costa Rica the evening before). Each runs ~1.5–2.3 h.
  - **Deporte+**: Sundays, starting ~14:00–16:00 UTC (~8–10 am Costa Rica). Each runs ~1 h. Spanish, Costa Rican football.

### Most recent finished events (newest first)
| Date (UTC start) | Title | Duration |
|---|---|---|
| 2026-11-04 | Dojo Coding Studio | ~1h40m |
| 2026-11-01 | Hoy se juega el CLÁSICO NACIONAL \| Deporte+ | ~1h00m |
| 2026-10-30 | Dojo Coding Studio | ~1h38m |
| 2026-10-28 | Dojo Coding Studio | ~2h20m |
| 2026-10-25 | Se jugó la IDA en Copa Centroamericana \| Deporte+ | ~1h01m |
| 2026-10-23 | Dojo Coding Studio | ~1h58m |
| 2026-10-21 | Dojo Coding Studio | ~1h59m |
| 2026-10-18 | Alvaro Sanchez y Jason Arce nos acompañan \| Crimen Organizado en el Futbol \| Deporte+ | ~1h01m |
| 2026-10-11 | Mauricio Wright y Oscar Valverde en Deporte+ | ~1h13m |
| 2026-10-04 | La Selección con Claudio Ciccia y Andrey Campos | ~1h03m |
| 2026-10-01 | Dojo Coding Studio | ~1h45m |

## Recordings (downloadable, they EXPIRE)
Restream keeps recordings for ~15 days. Only these 6 are still available. **Download any you want to clip or edit before they expire.**

| Event date | Title | Length | Files | Expires (UTC) |
|---|---|---|---|---|
| 2026-11-04 | Dojo Coding Studio | 1h40m | video + audio | 2026-11-19 |
| 2026-11-01 | Deporte+ (Clásico Nacional)* | 1h00m | video, secondary video, audio | 2026-11-16 |
| 2026-10-30 | Dojo Coding Studio | 1h38m | video + audio | 2026-11-14 |
| 2026-10-28 | Dojo Coding Studio | 2h20m | video + audio | 2026-11-12 |
| 2026-10-25 | Deporte+ (Copa Centroamericana IDA) | 1h01m | video, secondary video, audio | 2026-11-09 |
| 2026-10-23 | Dojo Coding Studio | 1h58m | video + audio | **2026-11-07 (2 days left)** |

\* The recording for the 11-01 event is titled "Se jugó la IDA en Copa Centroamericana", but its event is "Hoy se juega el CLÁSICO NACIONAL". It looks like a reused title. Match recordings by event, not by title.

The Deporte+ recordings include a **secondary video** track, which is probably a separate camera or vertical layout.

Recording tools: `list_user_recordings` (optionally filter by `eventId`), `get_recording_download_url`. Transcripts: `list_event_transcriptions` (calling it starts transcription if none exists), `get_event_transcript`.

Any of these event recordings can be used as a new clips project source: `create_user_clips_project` with `sourceType: "Event"` and the `eventId`. Only one event (an older RTMP stream) has been clipped so far.

## Connected channels (destinations)
22 channels across 4 brands/people:

| Brand | Channels |
|---|---|
| **Dojo Coding / trydojo.io** | YouTube "Dojo Coding", TikTok "Dojo Coding", TikTok "trydojo.io", Instagram "trydojo.io", LinkedIn "Dojo Coding", Facebook "DOJO", Custom RTMP "DojoOS PROD" |
| **Deporte+ (Deportemascr)** | YouTube "Deportemascr", Facebook "DeportemasCR", Instagram "deportemascr", TikTok "Deportemascr", YouTube "La Mesa De los 10" |
| **Personal / hosts** | YouTube "jeaustin campos", Instagram "jeaustincamposb"; Pablo Izaguirre (YouTube, Facebook, TikTok, Instagram); Roy Myers / "El Maravilloso Myers" (YouTube, Facebook, Instagram); Facebook "Kenneth Paniagua" |

Use `list_channels` to get IDs and URLs. Use `get_event` / `list_event_destinations` to see which channels a given event went to.

## Clipping controls available through the MCP
These parameters only apply when you **create a new project** (`create_user_clips_project`):

| Param | Purpose |
|---|---|
| `sourceType` (required) | `Event` (needs `eventId`), `VideoStorageFile` (needs `videoStorageFileId`), or `PublicUrl` (needs `publicUrl`, which must be directly downloadable) |
| `selectedTimeRange` | `{ startOffsetSeconds, endOffsetSeconds }`: clip only part of the source |
| `singleClipMode` | `true` = generate a single clip |
| `forceReprocess` | Re-run processing on a source that was already processed |
| `title` | Optional project title |

## Relevant tools (Restream MCP)
| Tool | Use |
|---|---|
| `get_current_user` | Account / org / workspace |
| `list_user_clips_projects` | Discover project IDs (`sortBy`: `CreatedAt` / `LastActivity`) |
| `get_user_clips_project_details` | Clips, transcripts, scores, thumbnails, posted clips |
| `create_user_clips_project` | New project (params above) |
| `create_user_clip_download` | Temporary download URL for one clip |
| `list_user_storage_files` | Find `videoStorageFileId` values |
| `list_user_events` | Find `eventId` values |

## How to rediscover IDs
Internal IDs are left out of this file on purpose. To get them:
1. Call `list_user_clips_projects`. It returns the single project's ID.
2. Call `get_user_clips_project_details` with that ID. It returns the clip IDs.
3. Call `create_user_clip_download` with a clip ID. It returns a temporary download URL.

Do not show internal IDs to the user unless they explicitly ask for them.

## Integration idea for this repo
To use these clips in Remotion, download each one with `create_user_clip_download`, save the MP4 into `public/`, and reference it from a composition. This follows the same pattern as the existing `generate_video_from_text` tool in `src/index.ts`, which writes MP4s to `public/`.
