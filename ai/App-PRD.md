# AI Document Viewer – **Pure MVP Slice** PRD

## Overview

Build the thinnest end‑to‑end vertical slice of an AI‑enhanced document viewer in Wasp + Shadcn‑ui. The goal is to prove:

- user authentication
- file upload & storage
- server‑side OpenAI call
- displaying the first page of the document
- surfacing the raw AI response

This version is **for one logged‑in user only**, with no multi‑page navigation, collaboration, or UI polish beyond the essentials.

---

## Core Features

1. **Auth & Landing**

   - Email/password sign‑up & log‑in (Wasp built‑in).
   - After auth, redirect to `/app` (Main Page).

2. **Single‑file Upload**

   - Drag‑and‑drop or “Choose file” (PDF, PNG, JPEG).
   - Max 10 MB; reject larger.
   - On upload, store binary to `/uploads/${userId}/${uuid}`.
   - Insert a record in `File` table:

     ```
     id, userId, filename, mime, createdAt, aiStatus(enum: idle|processing|done|error), aiRaw(json?)
     ```

3. **Async AI Processing**

   - Immediately enqueue a Wasp job `processFile(id)`.
   - Job reads the binary → sends to OpenAI Vision → saves full JSON in `aiRaw`, sets `aiStatus` accordingly.
   - First‑pass prompt: “Summarize this document in a single paragraph.”

4. **Main Page Layout**

   ```
   +------------------------+-------------------------------+
   |   FileList (left)      |   Viewer + Insights (right)   |
   |   • Vertical scroll    |                               |
   |   • Shows filename &   |   ┌────────── Viewer ───────┐ |
   |     time ago           |   | first page as <img>   |  |
   |                        |   └────────────────────────┘ |
   |                        |   ┌──── AI Raw (pre tag) ──┐ |
   |                        |   | { …json… }             | |
   |                        |   └────────────────────────┘ |
   +------------------------+-------------------------------+
   ```

   - **FileList**: live query sorted by `createdAt DESC`. Selecting a row loads that file’s data.
   - **Viewer**: Render first page as image (use pdf.js or convert server‑side on upload; choose the simpler path).
   - **Insight panel**: show “Processing…” spinner until `aiStatus == done`, then prettify `aiRaw` as JSON.

5. **Basic Error Handling**

   - Upload failure, AI error, or unsupported format surface an alert banner.

---

## Non‑Goals (deferred to later slices)

- Multi‑page scrolling / thumbnails.
- Collaboration, sharing, or role‑based access.
- Pretty AI insight rendering (tables, highlights, etc.).
- Storage abstraction beyond local disk.
- Advanced security (rate limiting, S3 presigned URLs, etc.).

---

## Technical Notes

- **Tables (Prisma in `main.db`)**

  ```prisma
  model File {
    id        String   @id @default(cuid())
    userId    String
    filename  String
    mime      String
    createdAt DateTime @default(now())
    aiStatus  String   @default("idle")
    aiRaw     Json?
    @@index([userId, createdAt])
  }
  ```

- **Actions / Jobs**

  - `uploadFile(file)` → stores binary, inserts row, triggers `processFile`.
  - `processFile(id)` → OpenAI call; update row.

- **Front‑end**

  - Shadcn components: `ScrollArea`, `Card`, `Button`, `Input`.
  - React query `useQuery(files)` with polling or Wasp live queries for status updates.

---

## Acceptance Criteria

| ID  | Scenario                  | Success Metric                                                                      |
| --- | ------------------------- | ----------------------------------------------------------------------------------- |
| A1  | New user signs up         | Redirected to Main Page                                                             |
| A2  | User uploads a 2‑page PDF | File appears in list; first page renders; AI status shows “Processing…” then “Done” |
| A3  | AI completes              | Raw JSON visible without manual refresh                                             |
| A4  | Upload >10 MB             | User sees “File too large” alert                                                    |

Deliver this MVP, validate the workflow, then iterate.
