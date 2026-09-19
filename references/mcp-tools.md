# MCP tool return shapes

All hub tools return JSON objects. Failures use:

```json
{"ok": false, "code": "PATH_DENIED", "error": "Access denied: path must stay inside the knowledge hub root"}
```

Common `code` values: `PATH_DENIED`, `NOT_FOUND`, `TOO_LARGE`, `UNSUPPORTED_TYPE`, `INVALID_QUERY`, `INVALID_PATH`, `INVALID_LIMIT`, `HUB_UNAVAILABLE`, `EMPTY_PDF`, `EXTRACTOR_UNAVAILABLE`.

## list_hub_files

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

Includes `is_text`, `is_pdf`, and `readable` (text or PDF, size ≤ 2 MB).

## get_latest_files

`files` is the same entry shape, newest first. `limit` max 50.

## read_hub_file

```json
{
  "ok": true,
  "path": "files/COMP2022/lecture.md",
  "size_bytes": 120,
  "encoding": "utf-8",
  "content": "# Lecture\n..."
}
```

PDF example:

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

```json
{
  "ok": true,
  "query": "stack",
  "truncated": false,
  "matches": [
    {
      "path": "files/COMP2022/lecture.md",
      "name": "lecture.md",
      "snippets": [{"line": 2, "text": "Stacks are LIFO."}]
    }
  ]
}
```
