# Restream Clipping Context (handoff)

_Snapshot taken 2026-11-05 through the Restream MCP server._

## TL;DR
- Account-level clipping settings (clip length, aspect ratio, captions, branding, auto-clip defaults) are **not exposed** by the Restream MCP. They can only be viewed or changed in the Restream web app.
- The account has **1 clips project** with **4 AI-generated clips**. None have been posted to platforms.
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
