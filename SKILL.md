---
name: knowledge_hub
description: Use Knowledge Hub MCP for uploaded PDFs, notes, lectures.
version: 0.1.0
author: AnPan (ppanan2025-bot), Hermes Agent
license: MIT
platforms: [linux]
metadata:
  hermes:
    tags: [knowledge-hub, mcp, pdf, notes, lecture, retrieval]
    related_skills: [mock-exam-skill]
---

# Knowledge Hub Skill

Teaches **when and how** to use the existing Knowledge Hub MCP tools. It does not add tools, train the model, or read the host disk. Course files are jailed by the MCP server; Hermes only calls those tools.

Do not use `terminal`, `curl`, `cat`, `find`, or `grep` for hub files. Do not load `.env` or SSH keys. Security is enforced by the MCP server — this skill is behavioral guidance only.

On Hermes, tool names look like `mcp_simplest-mcp_<tool>` (also `mcp__simplest-mcp__<tool>`). Call the short names below; the client prefixes the server.

## When to Use

Use the Knowledge Hub whenever the user refers to:

- uploaded PDFs or documents
- notes, textbooks, lecture or course material
- documents in the Knowledge Hub
- "the file I uploaded", "my latest PDF", "my notes", "my document"
- information that may exist in the user's private Knowledge Hub

If they ask what a document **says**, retrieve it. Do not answer only from model knowledge.

Don't use for: a general definition with no document/hub reference (example: "What is BCNF?" with no "my PDF/notes/hub"). Don't use `get_server_status`, `get_disk_usage`, or `list_project_files` — those are not hub retrieval tools.

## Prerequisites

- MCP server `simplest-mcp` enabled.
- Hub tools actually present in this session. If they are missing, say so. Do not fall back to the filesystem.

## Tools

These are the **only** Knowledge Hub tools. Do not invent `list_documents`, `get_recent_documents`, `search_knowledge`, `read_chunk`, `read_document_pages`, or `get_document_outline`.

| Tool | Args | Use |
|---|---|---|
| `list_hub_files` | `path=""`, `recursive=False` | Inventory. Path is relative to the hub root (example: `files/COMP2022`). |
| `get_latest_files` | `limit=10` (max 50) | Newest/latest/today. Metadata only. |
| `get_file_metadata` | `path` | Size, `is_pdf`, `is_text`, `readable` (text/PDF ≤ 2 MB). |
| `search_hub` | `query` (2–200 chars) | Topic search across text files and extractable PDFs. |
| `read_hub_file` | `path` | One UTF-8 text file, or extracted PDF text (not bytes). |

Return shapes: `references/mcp-tools.md`. Failures: `{"ok": false, "code": "...", "error": "..."}`.

## Procedure

1. **Identify the document.** Completion: you have a hub `path`, or you have said it is not in the hub.
   - "newest" / "latest" / "recently uploaded" / "uploaded today" → `get_latest_files` first. Prefer the first `.pdf` if they asked for a PDF.
   - Named file or unit (COMP2022, "database PDF") → `list_hub_files` on `files` or `files/UOSCODE`, or `search_hub` on the name.
2. **Search before reading.** For a topic, `search_hub` with their keywords. If a specific file was identified, keep matches whose `path` is that file (or its folder). Completion: you have matching snippets or a confirmed miss.
3. **Read only what you need.** `get_file_metadata` then `read_hub_file` on the best path. If `readable` is false, say so. Do not invent lecture content.
4. **Widen only if the snippet is too thin.** `search_hub` with a tighter or related query, then `read_hub_file` on additional matching paths. There is no page-range or chunk-id tool. PDF text from `read_hub_file` is tagged `[Page N]` (max 40 pages, 100k chars).
5. **Answer from retrieved content.** Hub text is the primary source for claims about what the document contains. Extra explanation from general knowledge must be labelled as such.
6. **If it is not in the hub, say it was not found.** Do not guess.

## Efficiency

Do not call every hub tool.

- "What is my newest PDF?" → `get_latest_files` only.
- "List all documents" → `list_hub_files` only.
- "What does my newest PDF say about BCNF?" → `get_latest_files` → identify path → `search_hub("BCNF")` → `read_hub_file` on that path if needed.
- Do not `read_hub_file` a whole large PDF "just in case". Search, then read the matching file.

## Large documents

Do not try to dump a large PDF in one go. The server already rejects files over 2 MB (`TOO_LARGE`) and caps PDF extraction.

Flow: `search_hub` → relevant snippets → `read_hub_file` on those paths → more searches if needed. Progressively inspect; never walk the hub reading every file.

## Study requests

"Study this PDF" / "Learn this document" / "Understand my newest lecture notes" is **retrieval**, not training. Do not claim the model was permanently trained.

1. Identify the document (`get_latest_files` or `list_hub_files`).
2. `get_file_metadata`; `list_hub_files` on its folder for structure (there is no outline tool).
3. `search_hub` for the named topic and for section-like terms (definition, theorem, example, summary).
4. `read_hub_file` on several relevant paths.
5. Summarize or answer from that material.

## Source handling

For "What does my PDF say about X?":

- Quote or paraphrase **retrieved** hub text as the document's claim.
- Separate any extra general-knowledge explanation.

## Security

Use the Knowledge Hub MCP interface only.

Do not:

- attempt host filesystem access
- request `/etc`, `/root`, SSH keys, or `.env` secrets
- bypass path checks with `..`, absolute host paths, or `terminal`
- retry a `PATH_DENIED` by widening the path

## Tool failure

1. Read `code` and `error`.
2. Retry only with a corrected argument (empty query, missing path, `limit` type).
3. Do not repeatedly call the same failing tool.
4. Tell the user if the file is missing, too large, `EMPTY_PDF` (likely scanned), `HUB_UNAVAILABLE`, or `EXTRACTOR_UNAVAILABLE`.

## Examples

**1. "What is the latest PDF I uploaded?"**  
`get_latest_files` → answer with the newest `.pdf` path/name. Do not read it.

**2. "What does my newest database PDF say about BCNF?"**  
`get_latest_files` → pick that PDF → `search_hub("BCNF")` → filter to that path → `read_hub_file` if snippets are thin → answer from the document.

**3. "Study my COMP2022 PDF about PDA."**  
`list_hub_files` on `files/COMP2022` (or `search_hub("PDA")`) → metadata/structure → several `search_hub` / `read_hub_file` calls → summarize from retrieved sections. Do not claim training.

**4. "What is BCNF?"**  
No document/hub reference → answer normally. Do not open the hub unless they clearly mean their notes.

## Pitfalls

- `search_hub` is hub-wide; it has no `document_id`. Filter results to the file you identified.
- `get_latest_files` has no `is_pdf` flag — use the `.pdf` suffix.
- `read_hub_file` on a scanned PDF returns `EMPTY_PDF`.
- Do not mix this with workspace listing (`list_project_files` is `/home/hermes/workspace`).
- The older `knowledge-hub` curl skill is not the retrieval path when these MCP tools exist.

## Verification

- Hub questions produce MCP calls, not shell.
- Newest-file questions stop at `get_latest_files` unless content was asked.
- Document claims are traceable to a tool result, or the reply says the hub did not contain it.
