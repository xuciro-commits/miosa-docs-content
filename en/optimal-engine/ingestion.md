---
slug: "optimal-engine/ingestion"
title: "Sources and ingestion"
description: "Every way to get data into Optimal Engine - text, files, chat sessions, Notion exports, batch imports, and apps such as Slack, GitHub, and Linear - with the file types it parses and the limits that apply."
section: "Optimal Engine"
order: 8005
---

Every input is kept as a source package first, then classified, split into chunks, indexed, and mined for claims.
The original is never rewritten, so every fact can point back to the source it came from.

## Ways in

| Input | Route or command |
|---|---|
| Text | `POST /api/ingest`, `optimal capture`, `mix optimal.ingest` |
| A file | `POST /api/assets` with the bytes as base64 |
| A folder of files | `mix optimal.ingest_workspace` |
| A finished chat | `POST /api/sessions/commit`, `optimal session commit` |
| A durable memory | `POST /api/memory`, `POST /api/memory/remember`, `optimal remember` |
| A Notion export | `POST /api/import/notion`, `mix optimal.import_notion` |
| Many records at once | `POST /api/batch/import/signals`, `POST /api/batch/import/memories` |
| An app | Connectors, below |
| Another Engine's workspace | A workspace bundle; see [Operations](/docs/optimal-engine/operations#move-a-workspace) |

## Text

```bash
curl -s -X POST http://127.0.0.1:4200/api/ingest \
  -H "Authorization: Bearer $OPTIMAL_ENGINE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "workspace": "default:notes",
    "title": "Pricing decision",
    "text": "Decided: annual plans get two months free from March.",
    "node": "pricing",
    "extract_claims": true
  }'
```

| Field | Required | Meaning |
|---|---|---|
| `text` | yes | The content |
| `workspace` | no | Workspace id; `default` when left out |
| `title` | no | Title shown in results |
| `genre` | no | Override the genre the Engine would pick |
| `node` | no | Node slug the source belongs to |
| `extract_claims` | no | `true` by default |

The response is `{"ok": true, "signal_id", "genre", "type", "entities", "source_package_id"}`.
A missing `text` answers `400`; a source the pipeline cannot take answers `422` with `ok: false` and an `error`.

## Files

`POST /api/assets` takes a file as `content_base64` with its filename.
The Engine stores the bytes, hashed, under the workspace's `assets/` folder, or in S3-compatible storage when `OPTIMAL_ASSET_STORAGE=s3`.

| Kind | Extensions | Text comes from |
|---|---|---|
| Markdown and text | `.md` `.txt` `.rst` `.adoc` `.log` | the file |
| Data | `.yaml` `.yml` `.toml` `.json` `.csv` `.tsv` | the file |
| Web | `.html` `.htm` `.xhtml` | the file |
| Code | about 37 languages, such as `.ex` `.py` `.js` `.ts` `.go` `.rs` `.java` `.sql` | the file |
| Office | `.docx` `.pptx` `.xlsx` | the file |
| PDF | `.pdf` | `pdftotext` |
| Images | `.png` `.jpg` `.jpeg` `.gif` `.webp` `.svg` | `tesseract`, plus a description from the vision model |
| Audio | `.mp3` `.wav` `.m4a` `.ogg` `.flac` | a whisper.cpp server |
| Video | `.mp4` `.mov` `.webm` `.mkv` | `ffmpeg` keyframes and audio |

An unknown extension is read as plain text.
When a tool is missing, the file is kept with its metadata and a warning such as `ffmpeg not on PATH`, and you can parse it again later.

## Chat sessions

Commit a finished conversation so its lessons become memory:

```bash
curl -s -X POST http://127.0.0.1:4200/api/sessions/commit \
  -H "Authorization: Bearer $OPTIMAL_ENGINE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "workspace": "default:notes",
    "correlation_id": "chat-2026-10-23-01",
    "title": "Renewal planning",
    "messages": [
      {"role": "user", "content": "When is the supplier renewal?"},
      {"role": "assistant", "content": "The call moved to the 14th."}
    ],
    "learned": ["The supplier renewal call is on the 14th."]
  }'
```

The chat is stored as an episode.
Each `learned` item, up to 50, becomes a proposed claim in the review queue.
The response (`201`) has the `session_id`, its `uri`, the `summary`, and the `message_count`.
Read sessions back with `GET /api/sessions` and `GET /api/sessions/:id`.

## Notion exports

Export a Notion workspace as **Markdown & CSV**, then send the zip:

```bash
mix optimal.import_notion ./notion-export.zip --workspace default:notes
```

Over HTTP, `POST /api/import/notion` takes the zip as the request body and starts a job; follow it with `GET /api/import/notion/:job_id`.
Each page, database, and row becomes its own item with an `optimal://<workspace>/notion/<id>` URI, and page hierarchy and links become graph edges.
The importer checks sizes before it unpacks anything, refuses ZIP64 archives, and never turns an entry name into a path on disk.

## Apps (connectors)

A connector pulls records from an app on a schedule and stores them as sources.
It only reads; nothing is written back to the app.

| Group | Apps |
|---|---|
| Mail, files, and docs | Gmail, Google Drive, Notion, Dropbox, OneDrive, Confluence, a local sources folder |
| Chat and meetings | Slack, Microsoft Teams, Zoom, Fathom |
| Work tracking and code | GitHub, Linear, Jira |
| CRM and business | HubSpot, Salesforce, HighLevel, Stripe |
| Publishing and forms | beehiiv, Jotform |

Set `CONNECTOR_KEY` before you add one: it encrypts the app keys the Engine stores.

```bash
mix optimal.connector list
mix optimal.pull
```

Over HTTP, `GET` and `POST /api/connectors` list and add connectors, and `POST /api/connectors/:id/run` runs one now.
`docs/guides/connectors.md` in the repository lists what each one pulls and how it signs in.
Pulled records are stored as database rows with `optimal://connectors/<kind>/<id>` URIs, not as files, unless the connector is set to write files.

> **Note: How deep a connector goes**
>
> Every connector pulls the records its guide lists.
> Downloading every attachment and binary file is not complete for every app yet.

## Limits

| Limit | Value |
|---|---|
| JSON request body | 8 MB |
| Import file | 1 GiB (`OPTIMAL_IMPORT_MAX_BYTES`) |
| Workspace bundle | 5 GiB (`OPTIMAL_BUNDLE_MAX_BYTES`) |
| Text sent to the embedding model | 6,000 characters per chunk |
| Learned items per chat session | 50 |
