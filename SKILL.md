---
name: knowledge-hub-mcp
description: >
  Read course materials from the local knowledge hub through MCP tools only.
  Use when the user asks what is stored, which notes mention a topic, or to
  read hub text files or lecture PDFs. Do not use the terminal or curl for hub files.
version: 0.2.0
author: AnPan (ppanan2025-bot)
license: MIT
platforms: [linux]
metadata:
  hermes:
    tags: [knowledge-hub, mcp, course-notes, search]
    related_skills: [mock-exam-skill]
---

# Knowledge Hub MCP

The knowledge hub lives on the same Hermes host. Course files are under a
fixed root: `/opt/knowledge-hub/data`. Hermes must use the MCP tools below.
Do not `cat`, `find`, `grep`, or `curl` the hub, and do not read `.env` files.

These tools are read-only. They cannot write, delete, rename, or chmod.

## When to Use

- The user asks what lectures, tutorials, or notes are stored.
- They name a unit code (for example COMP2022) and want hub files, not a web search.
- They ask which notes mention a topic.
- They want the text of a markdown note or the extracted text of a lecture PDF.

Do not use this skill for disk/memory status (`get_server_status`) or for
editing files. PDFs are readable through `read_hub_file` as extracted text
only, never as raw bytes. If extraction returns `EMPTY_PDF`, the file is
probably a scanned image.

## Tools

Call MCP tools. Names on Hermes look like `mcp_simplest-mcp_<tool>`.

1. `list_hub_files(path="", recursive=False)`
   See units and files. Start here. Example path: `files/COMP2022`.
2. `get_file_metadata(path)`
   Check size and whether the file is readable text or a PDF under 2 MB.
3. `get_latest_files(limit=10)`
   What was added or updated recently. Metadata only.
4. `search_hub(query)`
   Find a topic in text files and extractable PDFs. Query must be at least 2 characters.
5. `read_hub_file(path)`
   Read one UTF-8 text file, or extract text from a PDF ≤ 2 MB.

## Procedure

1. If the unit or folder is unknown, call `list_hub_files` on `files` or `files/UOSCODE`.
2. If the user asks about a topic, call `search_hub` with their keywords.
3. Before reading, call `get_file_metadata`. If `readable` is false, say so.
   Do not invent lecture content.
4. Call `read_hub_file` for text or PDF. Use the returned `content`. If the
   code is `EMPTY_PDF`, say the PDF has no extractable text.
5. Quote paths from the tool JSON. Never follow a path outside the hub.

## Pitfalls

- Do not use `terminal` to inspect `/opt/knowledge-hub`.
- Do not load API keys or `.env` files.
- Do not mix this with workspace listing (`list_project_files`); that jail is `/home/hermes/workspace`.
- If a tool returns `ok: false`, report the `code` and `error`. Do not retry with `../` or absolute host paths like `/etc`.

See `references/mcp-tools.md` for return shapes.
