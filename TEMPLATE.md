
Action: read({"request":{"path":"/app/TEMPLATE.md"}})
Observation: /app/TEMPLATE.md:
1|# farm-ts — context capsule
2|
3|Read this once: it holds every template fact you need and you will not have to
4|re-open it. README.md carries the workflow, this file the facts. Skip
5|`components/ui/**`, `tsconfig*`, `package.json`, `node_modules/`, and
6|`vite.config.ts` — their contents are reproduced below, so opening them only
7|spends turns.
8|
9|## 1. Environment state at start
10|
11|Every service (frontend, backend, mongodb) is ALREADY RUNNING under supervisor
12|and the preview URL is healthy when your session begins — the "preview must start
13|before the first ask_human" precondition is ALREADY MET, and nothing you run can
14|make it more true. Do not run `supervisorctl status`, curl a health endpoint, or
15|restart anything at session start. This applies to the testing subagent too.
16|
17|Ports: frontend `3000`, backend `8001` (Vite proxies `/api/*` → `8001`). Alias:
18|`@/*` → `frontend/src/*`. There is no cross-language alias — the boundary is HTTP.
19|`python` already resolves to the backend venv interpreter.
20|
21|Testing subagent, in one line each: seed facts and credentials are in
22|`memory/SPEC.md` + `memory/test_credentials.md`; screenshot `path` must be a bare
23|filename and `quality` is JPEG-only; `page.goto` needs an absolute
24|`http://localhost:3000/…` URL. Details in §9.
25|
26|## 2. Preinstalled — never install these
27|
28|- **Frontend icons**: `lucide-react` for UI glyphs. Brand logos: `@icons-pack/react-simple-icons`
29|  (`SiInstagram`, `SiGithub`, `SiYoutube`, `SiX`…; filled marks, `size`/`color` props). Lucide 1.x
30|  has no brand icons; the legacy names (`Instagram`, `Github`, `Linkedin`, `Twitter`…) still
31|  resolve from `lucide-react` through `src/lib/lucide-react.tsx`.
32|- **Frontend fonts**: 16 `@fontsource` families ship in the image. Variable
33|  (`@fontsource-variable/<name>`): `dm-sans`, `geist`, `geist-mono`,
34|  `instrument-sans`, `inter`, `jetbrains-mono`, `lora`, `manrope`, `outfit`,
35|  `playfair-display`, `plus-jakarta-sans`, `sora`, `space-grotesk`. Plain
36|  (`@fontsource/<name>`): `ibm-plex-sans`, `ibm-plex-mono`, `poppins` — these
37|  three have no `-variable` build, so `@fontsource-variable/ibm-plex-sans` does
38|  not resolve. Never `yarn add` a font, and never `@import` Google Fonts: a CDN
39|  import ships a runtime dependency inside the delivered app.
40|- **Frontend data/UI**: `@tanstack/react-query`, `sonner`, `motion`,
41|  `react-router-dom`, `date-fns`, `next-themes`, `react-day-picker`,
42|  `@base-ui/react`, `class-variance-authority`, `clsx`, `tailwind-merge`,
43|  `tw-animate-css`.
44|- **Frontend charts**: `recharts` (with `react-is` pinned to the React 19 line). `<Tooltip formatter={(value: number,
45|  name: string) => …}>` and `labelFormatter={(label: string) => …}` type-check as on recharts 2: the template's
46|  `recharts` alias (`src/lib/recharts.tsx`) restores Tooltip's generic typing; the raw package is `recharts-upstream`.
47|- **Backend** (`backend/requirements.txt`): `fastapi`, `uvicorn`, `motor`,
48|  `pymongo`, `pydantic` v2, `python-dotenv`, `httpx`, `requests`, `pandas`,
49|  `numpy`, `openpyxl`, `emergentintegrations`, `boto3`, `typer`, `pytest` +
50|  `pytest-asyncio`/`xdist`; auth and uploads are covered too — `pyjwt`,
51|  `python-jose`, `passlib`, `cryptography`, `email-validator`,
52|  `python-multipart`.
53|- **Absent**, install before use: `leaflet`/`@types/leaflet`, any
54|  other icon pack, any font package outside the §2 manifest.
55|
56|This list is why you never need to read `package.json` or list `node_modules`.
57|
58|## 3. Dir map
59|
60|| Path | Purpose |
61||---|---|
62|| `backend/server.py` | FastAPI bootstrap: `app = FastAPI()`, `api_router = APIRouter(prefix="/api")`, routes registered on the **router**, `app.include_router(api_router)` last. `status` is the pattern to copy |
63|| `backend/lib/db.py` | ships the motor client + `db` handle and self-loads `.env`; import it from `server.py` *and* every router — defining `db` in `server.py` and importing it back is a circular import. Also holds `INDEXES` (one entry per collection) which `ensure_indexes()` applies at startup |
64|| `backend/models/*.py` | Pydantic v2 request/response models once `server.py` gets crowded — package exists with `__init__.py`, just add modules |
65|| `backend/routers/*.py` | one `APIRouter` per resource, mounted from `server.py` — package exists with `__init__.py`, just add modules |
66|| `backend/seed.py` | **create this** for seed data; run `cd /app/backend && python seed.py`. Idempotent, not imported by `server.py`, and gets env + client via `from lib.db import db, ensure_indexes`; if it drops collections, `await ensure_indexes()` afterwards (drop discards indexes) |
67|| `backend/lib/dates.py` | `today_iso(tz=None)` — server-side "today"; pod clock is UTC |
68|| `backend/.env` | `MONGO_URL`, `DB_NAME`, `CORS_ORIGINS`; loaded by `python-dotenv`, read via `os.environ` |
69|| `frontend/src/main.tsx` | mounts `StrictMode` + `QueryClientProvider` + `BrowserRouter` around `<App />`. One Router and one `QueryClientProvider` per app: don't mount either again anywhere (a second Router breaks routing; `<Routes>` in `App.tsx` just works) |
70|| `frontend/src/App.tsx` | the `<Routes>` table — one `<Route>` per page, added in the same edit that creates the page. A page with no `<Route>` is unreachable, and any URL without a matching `<Route>` renders a **blank page** — `<Routes>` matches nothing and mounts nothing |
71|| `frontend/src/pages/*.tsx` | one screen per file, default-exported, imported into `App.tsx` as `@/pages/<Name>`; `Home.tsx` ships as the worked example |
72|| `frontend/src/lib/api.ts` | `apiGet/apiPost/apiPut/apiPatch/apiDelete<T>` over base `/api`, throwing `ApiError` |
73|| `frontend/src/lib/utils.ts` | `cn()` |
74|| `frontend/src/components/ui/` | shadcn `base-nova` on **@base-ui/react** (index in §11) |
75|| `frontend/src/index.css` | Tailwind v4 entry + theme tokens (no `tailwind.config.js`) |
76|| `@emergentbase/overlay` (npm) | platform infra (branded error overlay vite plugin) — not in your source tree. Probe: `curl -s localhost:3000/__emergent_overlay__/health` → 503 = JSON with the current build/runtime errors, 200 = none known (no `-f`; a 200 with `degraded` in the body means the probe itself failed). It reflects state since the last page load, so load the page once for authority. Runtime crashes leave `crash-*.json` + digest `.md` in `.emergent/recordings/` (newest three kept) — read the `.md` first. Needs frontend HMR (§10): with it off you still get the error card, but no crash dumps and no bug button. With `EMERGENT_OVERLAY_RRWEB="buffer"` on the frontend `environment=` line the dumps include the recent interaction timeline (typed input masked); with `"off"`, error context and stack only. Bug button, for behavior you can't reproduce: set `EMERGENT_OVERLAY_FAB="on"` on that line in `/etc/supervisor/conf.d/supervisord.conf` (the one edit allowed there), `sudo supervisorctl reread && sudo supervisorctl update`, then ask the user to repeat the steps and use the button at the bottom-left of the preview — their session + note land as a `user-report-*` pair in `.emergent/recordings/`; read the newest `.md`. Leaving it on is fine; the user can hide it |
77|| `memory/SPEC.md`, `memory/test_credentials.md` | write seed facts + credentials here before delegating — the testing subagent reads them first |
78|| `backend/pytest.ini`, `backend/tests/`, `tests/` | pytest + Playwright scaffolding, pre-configured — don't edit or recreate; browser checks land in `.emergent/scripts/checks/` |
79|## 4. The typed-fetch boundary
80|
81|Nothing infers across Python↔TypeScript. Each endpoint has **two** declarations
82|you keep in sync in the same edit: a Pydantic model in `backend/` and a TS
83|interface in `frontend/`.
84|
85|```python
86|# backend/routers/things.py — every route hangs off a router, never off app
87|class Thing(BaseModel):
88|    id: str = Field(default_factory=lambda: str(uuid.uuid4()))
89|    name: str
90|
91|@router.get("/things/{id}", response_model=Thing)         # ← braces, FastAPI style
92|async def get_thing(id: str):
93|    doc = await db.things.find_one({"id": id})
94|    if not doc:
95|        raise HTTPException(status_code=404, detail="not found")
96|    return Thing(**doc)
97|```
98|
99|```ts
100|// frontend — mirror the model, call the relative path
101|interface Thing { id: string; name: string }
102|const thing = await apiGet<Thing>(`/things/${id}`);   // → /api/things/<id>
103|```
104|
105|```python
106|# backend/lib/db.py — the edit that adds a query adds its index; ensure_indexes() applies them at startup
107|INDEXES["things"] = [
108|    IndexModel([("id", ASCENDING)], name="id", unique=True),
109|    IndexModel([("owner_id", ASCENDING), ("created_at", DESCENDING)], name="owner_created"),
110|]
111|```
112|
113|Index every field a query filters, sorts, or dedupes on — `id` included, it is a
114|plain field, not `_id` — with compound keys ordered equality, sort, range. For
115|documents that expire add a separate single-field index on a datetime field
116|with `expireAfterSeconds`. Changing the keys or options of an existing index is
117|rejected at startup (logged, old index kept): drop it first,
118|`await db.things.drop_index("id")` from `seed.py`, then restart.
119|
120|Ids are string `uuid4`, never Mongo `ObjectId` (not JSON-serializable, leaks into
121|response bodies). Mongo is async motor: `await db.things.insert_one(...)`,
122|`await db.things.find().to_list(1000)`. FastAPI rejects a bad body with `422` and
123|a `{"detail": [...]}` payload before your handler runs — `ApiError.body` carries it.
124|
125|**Datetimes are the same class of trap as `ObjectId`, and more expensive.** BSON
126|stores UTC but motor hands back **naive** `datetime` objects. Subtracting or
127|comparing one against an aware `datetime.now(timezone.utc)` raises `TypeError`
128|and surfaces as a 500 (the classic "stop the timer" bug). Normalise on read —
129|`dt.replace(tzinfo=timezone.utc)` — and store aware UTC on write, so Pydantic
130|serialises with the offset and JavaScript `new Date(...)` parses it correctly.
131|
132|### The app is also served with no backend — never gate a page on a fetch
133|
134|When an environment is paused, the platform builds `frontend/` and serves it as a
135|**static bundle on the preview CDN**. There is no backend behind it, so every
136|`apiGet`/`apiPost` rejects. That build is what people see in a shared preview.
137|
138|So a failed fetch must degrade to a *partial* page, never a blank one or an error
139|screen: render the shell, nav and static copy unconditionally, and let only the
140|data-dependent region show an empty/skeleton state. A page whose entire return is
141|`error ? <Error/> : <Content/>` renders as an outage on the CDN even though the
142|app is fine — and it reads to the user as a broken app.
143|
144|Also gate on the error state, not just on missing data: react-query keeps the last
145|successful `data` through a failed refetch, so `data ? …` alone will keep showing
146|stale content as live after the backend goes away.
147|
148|### Auth apps: sessions are httpOnly cookies; `lib/session.ts` owns the cache
149|
150|- Sessions are httpOnly cookies the backend sets on login/signup and clears on
151|  logout (all auth routes under `/api/auth/*`). Never return tokens in JSON and
152|  never store or attach tokens in the frontend — cookies ride same-origin
153|  fetches automatically, and `GET /api/auth/me` answers "who am I".
154|- Frontend: after successful login/signup, `beginSession()`. Every sign-out
155|  control must `await endSession()`; never hand-roll logout — clearing only the
156|  server session leaks the previous account's react-query cache to the next
157|  login on this browser.
158|
159|## 5. base-ui contract — read before using any `components/ui` component
160|
161|- No `asChild`. Swap the rendered element with `render={<MyEl />}`:
162|  `<DialogClose render={<Button />} />`. For link-styled buttons prefer
163|  `<Link className={buttonVariants({ variant, size })}>` over `render={<Link/>}`.
164|- `Select`'s `onValueChange` gives `string` (the wrapper coerces base-ui's
165|  `null`) — type handlers `(value: string)`, never `string | null`. And
166|  `SelectValue` renders the RAW value, not the item's label (unlike radix) —
167|  pass a children function that maps value → label:
168|  `<SelectValue>{(v) => LABELS[v as string]}</SelectValue>`.
169|- Selected/checked state is `data-selected` / `data-checked`, not
170|  `data-state="active"`.
171|- `DropdownMenuContent`, `PopoverContent`, `SelectContent` self-portal — never
172|  wrap them in your own portal.
173|- Controlled inputs: set `value` + `onChange` together; a value-only `Input`
174|  produces a caret/hydration mismatch.
175|- **StrictMode double-invokes effects in dev** — on-mount side effects fire twice; mutations belong behind user actions.
176|
177|## 6. Stack traps
178|
179|| Flag / fact | Implication |
180||---|---|
181|| frontend `tsc --noEmit` checks **0 files** | root tsconfig is `"files": []` + project references — the working command is `yarn typecheck` from `frontend/` |
182|| unused imports / locals | not typecheck or build errors (`noUnusedLocals` is off); prune them when you touch a file, `yarn lint` still reports them |
183|| frontend `verbatimModuleSyntax` | types must use `import type { X }` |
184|| frontend `erasableSyntaxOnly` | no `enum`, no constructor parameter properties (declare fields, assign in the body) |
185|| frontend `strict: true` | never loosen it to clear an error; fix the type |
186|| backend route on `app` instead of `api_router` | lands outside `/api`, so the Vite proxy never reaches it — always the router |
187|| backend is fully async | `async def` handlers, `await` every motor call; a sync handler blocks the loop |
188|| lint is oxlint | `react/rules-of-hooks` = error; `react/only-export-components` = warn → hoist inline components out of render |
189|| logout without `endSession()` | previous account's react-query cache renders for the next login — route every sign-out through `lib/session` |
190|
191|The dev servers surface compile errors as you write. These are facts for writing
192|correct code, not commands to run mid-build.
193|
194|## 7. Common traps
195|
196|- `dark` is a Tailwind v4 `@custom-variant` — NEVER `@apply dark`; for
197|  dark-by-default replace the opening `<html>` tag in `frontend/index.html`
198|  with the commented `class="dark"` variant beside it.
199|- Link-styled buttons: `<Link className={buttonVariants({ variant, size })}>` —
200|  never `<Button render={<Link/>}>`.
201|- uvicorn runs with directory=/app/backend: imports are `from lib.x import y`,
202|  never `from backend.lib.x`.
203|- Standalone scripts (seed.py etc.) do not inherit server.py's dotenv — import
204|  `lib.db` or call `load_dotenv` yourself before touching `os.environ`.
205|- Fonts: only the §2 manifest is installed. Variable families import as
206|  `@fontsource-variable/<name>`; the only plain `@fontsource/<name>` are
207|  `ibm-plex-sans`, `ibm-plex-mono`, `poppins` — any other font import breaks the
208|  CSS build (blank page).
209|- `recharts` + `react-is` and `openpyxl` are preinstalled; `leaflet`/`@types/leaflet`
210|  is NOT — install before use.
211|- Node is 24.x — check package `engines` bounds before installing dependencies.
212|
213|## 8. Theming
214|
215|`frontend/src/index.css` is the Tailwind v4 entry: `@import`s (the last one is
216|the active `@fontsource` family) → `@custom-variant dark` → `@theme inline`
217|aliases (incl. `--font-sans` / `--font-heading`) → `:root` light tokens →
218|`.dark` → `@layer base`. Retheme = swap the font import, those two font lines,
219|and the hex values in `:root` + `.dark`; leave the other aliases and
220|`@layer base` alone. One `create_file` overwrite beats three `search_replace`es.
221|
222|## 9. Verify before you finish
223|
224|- **Typecheck once, as a named gate step** when the build is complete:
225|  `cd /app/frontend && yarn typecheck`, and
226|  `cd /app/backend && python -c 'import server'` for backend import sanity.
227|- **Screenshots** (`mcp_screenshot_tool_ts`): `path` must be a **bare filename**
228|  (`home.png`) — a directory prefix is written but never returned, landing under
229|  `/root/.emergent/automation_output/<ts>/`. `quality` is **JPEG-only**: passing
230|  it with a `.png` path fails the whole browser run (`options.quality is
231|  unsupported for the png screenshots`). Set the viewport before capturing; the
232|  image returns inline, so never `find` it on disk.
233|- **Browser scripts**: every `page.goto` needs an absolute
234|  `http://localhost:3000/...` URL — a relative `'/'` throws `Cannot navigate to
235|  invalid URL`.
236|- **Write the handoff**: seed facts into `memory/SPEC.md`, any credentials into
237|  `memory/test_credentials.md`. The testing subagent reads both first; if they
238|  are empty it re-derives them from your source and seed script.
239|- Then run the gate your system prompt defines (curl smoke + one happy-path
240|  browser pass). Anything deeper belongs to the testing subagent.
241|
242|## 10. Restart — after a config change only
243|
244|Both dev servers hot-reload (uvicorn `--reload`, Vite HMR) unless the supervisor conf says
245|otherwise: frontend HMR is off when its `environment=` line has `DISABLE_HOT_RELOAD=true`, backend
246|reload is on only when its command has `--reload`. Restart ONLY after changing `.env`,
247|`requirements.txt`, or `vite.config.ts` (or after any edit when reload is off) — never at session start.
248|For the supervisor edit in the overlay row (§3), use that row's `reread && update` instead.
249|
250|```bash
251|sudo supervisorctl restart frontend backend
252|for i in $(seq 30); do curl -sf http://localhost:8001/api/ >/dev/null && { echo "backend up"; break; }; sleep 1; done
253|for i in $(seq 30); do curl -sf http://localhost:3000 >/dev/null && { echo "frontend up"; break; }; sleep 1; done
254|```
255|
256|## 11. components/ui index
257|
258|Every wrapper spreads its remaining props onto the underlying primitive, so
259|rest-props = that primitive's props. Import from `@/components/ui/<file>`. Read
260|§5 first — the behavioural contract matters more than these prop lists.
261|
262|| File | Exports — own props (defaults) |
263||---|---|
264|| `badge.tsx` | `Badge` (span) `variant`: default \| secondary \| destructive \| outline \| ghost \| link; `badgeVariants` |
265|| `button.tsx` | `Button` `variant`: default \| outline \| secondary \| ghost \| destructive \| link; `size`: default \| xs \| sm \| lg \| icon \| icon-xs \| icon-sm \| icon-lg; `buttonVariants` |
266|| `calendar.tsx` | `Calendar` (react-day-picker `DayPicker` props) + `buttonVariant` (ghost), `showOutsideDays` (true), `captionLayout` ("label"), `locale`; `CalendarDayButton` |
267|| `card.tsx` | `Card` `size`: default \| sm; `CardHeader/Title/Description/Action/Content/Footer` (div props) |
268|| `checkbox.tsx` | `Checkbox` = base-ui `Checkbox.Root` (`checked`, `defaultChecked`, `onCheckedChange`, `indeterminate`); indicator built in |
269|| `dialog.tsx` | `Dialog`, `DialogTrigger/Portal/Close/Overlay/Title/Description`; `DialogContent` + `showCloseButton` (true); `DialogHeader`, `DialogFooter` |
270|| `dropdown-menu.tsx` | `DropdownMenu` = base-ui `Menu.Root`; `…Trigger/Portal/Group/RadioGroup/Sub/SubTrigger/Separator/Label/Shortcut`; `…Content`, `…SubContent` (self-portal) + `align`, `alignOffset`, `side`, `sideOffset`; `…Item`/`…CheckboxItem`/`…RadioItem` + `inset`, `variant`: default \| destructive |
271|| `input.tsx`, `label.tsx`, `textarea.tsx` | `Input` / `Label` / `Textarea` — plain element props |
272|| `popover.tsx` | `Popover` = base-ui `Popover.Root`; `PopoverTrigger/Title/Description/Header`; `PopoverContent` (self-portals) + `align` ("center"), `alignOffset`, `side` ("bottom"), `sideOffset` (4) |
273|| `select.tsx` | `Select` (wrapped `Select.Root`), `SelectGroup/Value/Content/Label/Item/Separator/ScrollUpButton/ScrollDownButton`; `SelectTrigger` + `size`: sm \| default |
274|| `sheet.tsx` | `Sheet`, `SheetTrigger/Close/Portal/Overlay/Title/Description/Header/Footer`; `SheetContent` + `side`: top \| right (default) \| bottom \| left, `showCloseButton` (true) |
275|| `sonner.tsx` | `Toaster` — sonner `ToasterProps` (`richColors`, …), matches the app's light/dark theme; defaults to `position="bottom-right"` — keep it there. `top-*` positions sit on the header nav and swallow clicks; `bottom-left` is where the preview's bug button lives. Mount once, then `toast()` from `sonner` |
276|| `table.tsx` | `Table` (in an `overflow-x-auto` div), `TableHeader/Body/Footer/Row/Head/Cell/Caption` |
277|| `tabs.tsx` | `Tabs` + `orientation`: horizontal (default) \| vertical; `TabsList` + `variant`: default \| line; `TabsTrigger`, `TabsContent`; `tabsListVariants` |
278|
279|Everything above is installed — run `npx shadcn@latest add <Noor>` only for a
280|component that is *not* in this index.
281|
