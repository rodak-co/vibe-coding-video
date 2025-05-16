#### 🟠 Slice A – Auth & single‑file upload

- [ ] Add `File` model (id, userId, filename, mime, createdAt, aiStatus default "idle", aiRaw Json?) in `main.db` schema.
- [ ] Run `wasp db migrate dev --name add_file_table`.
- [ ] Create server `/src/server/uploadFile.ts` action:

  - [ ] Accept multipart, write to `${UPLOAD_DIR}/${userId}/${uuid}.pdf`.
  - [ ] Insert DB row, set `aiStatus="processing"`.
  - [ ] Enqueue `processFile` job (to be implemented in Slice C).

- [ ] Scaffold `FileList` component with Shadcn `ScrollArea`; query files sorted by `createdAt DESC`.
- [ ] Add `<input type="file">` + drag‑drop; on change, call `uploadFile` then refetch list.
- [ ] Protected route `/app` that redirects anon users to `/login`.

---

#### 🟡 Slice B – First‑page viewer

- [ ] Decide render mode (client pdf.js for now).
- [ ] Add `ImageViewer` component:

  - [ ] Load file via presigned `/file/:id/page/1` endpoint or direct `pdfjsLib.getDocument`.
  - [ ] Render first page onto `<canvas>` in the main panel.

- [ ] Show placeholder “Select a file” when no file chosen.

---

#### 🟣 Slice C – Async AI processing job

- [ ] In `jobs/`, implement `processFile(fileId)` Wasp job:

  - [ ] Read binary from disk.
  - [ ] Call OpenAI Vision: basic prompt “Summarize this document in one paragraph.”
  - [ ] Persist result JSON to `aiRaw`, set `aiStatus="done"` (or `"error"` on failure).

- [ ] Add simple exponential‑backoff retry (max 3).
- [ ] Update `uploadFile` action to pass fileId to job queue.

---

#### 🟤 Slice D – Insights panel

- [ ] Create `InsightsPanel` component:

  - [ ] If `aiStatus!="done"`, show Shadcn spinner + “Processing…”.
  - [ ] When done, pretty‑print `aiRaw` using `pretty-print-json`.

- [ ] Wire `useQuery(fileId)` with `live: true` (or client polling every 3 s) so panel updates automatically.

---

#### 🔵 Slice E – Basic error & UX polish

- [ ] Validate file size ≤ 10 MB client‑side; display toast on violation.
- [ ] Server‑side guard for MIME types (pdf | image/\*).
- [ ] Global error boundary to show unexpected exceptions.
- [ ] Empty‑state copy for FileList (“No uploads yet”).

---

#### 🧪 Smoke & unit tests

- [ ] Vitest unit for `uploadFile` inserts correct DB row.
- [ ] Job test with mocked OpenAI response writes JSON and status.
- [ ] Cypress flow: sign‑up → upload sample PDF → AI result appears.

---

#### 🚀 Final‑run checklist

- [ ] `wasp start` runs without errors.
- [ ] Upload sample PDF, confirm first page renders.
- [ ] AI summary shows up within expected latency.
- [ ] README updated with setup + `.env.template`.

When every box is ticked, the Pure MVP slice is complete and ready for the next iteration.
