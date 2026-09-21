# MCP tool return shapes

Hub tools return JSON. Failures:

```json
{"ok": false, "code": "PATH_DENIED", "error": "Access denied: path must stay inside the knowledge hub root"}
```

Codes: `PATH_DENIED`, `NOT_FOUND`, `TOO_LARGE`, `UNSUPPORTED_TYPE`, `INVALID_QUERY`, `INVALID_PATH`, `INVALID_LIMIT`, `HUB_UNAVAILABLE`, `EMPTY_PDF`, `EXTRACTOR_UNAVAILABLE`.

There are no `document_id`, `chunk_id`, or page-range tools.

## list_hub_files

`path` relative to the hub root (`""` = root). `recursive` default false.

```json
{
  "ok": true,
  "root": "/opt/knowledge-hub/data",
  "path": "files/COMP2022",
  "recursive": false,
  "truncated": false,
  "files": [
    {
      "name": "lecture.md",
      "path": "files/COMP2022/lecture.md",
      "type": "file",
      "size_bytes": 120,
      "modified_at": "2026-09-19T00:00:00+00:00"
    }
  ]
}
```

## get_file_metadata

Includes `is_text`, `is_pdf`, and `readable` (supported text or PDF, size ≤ 2 MB).

## get_latest_files

`files` is the same entry shape, newest first. `limit` max 50. No `is_pdf` field — use the filename suffix.

## read_hub_file

Text:

```json
{
  "ok": true,
  "path": "files/COMP2022/lecture.md",
  "size_bytes": 120,
  "encoding": "utf-8",
  "content": "# Lecture\n..."
}
```

PDF (`encoding`: `pdf-text`, pages tagged `[Page N]`, max 40 pages):

```json
{
  "ok": true,
  "path": "files/COMP2022/lecture/slides.pdf",
  "size_bytes": 1155420,
  "encoding": "pdf-text",
  "page_count": 12,
  "content": "[Page 1]\nStacks are LIFO..."
}
```

## search_hub

Hub-wide. Query 2–200 characters. Filter `matches[].path` yourself if the user named one file.

```json
{
  "ok": true,
  "query": "BCNF",
  "truncated": false,
  "matches": [
    {
      "path": "files/COMP2022/lecture.md",
      "name": "lecture.md",
      "snippets": [{"line": 2, "text": "BCNF is a normal form..."}]
    }
  ]
}
```

PDF hits may include `"source": "pdf"`.
