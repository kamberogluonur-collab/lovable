# GUNCELLEMELER.md — Nextera site update log

> **How to use this file (for Lovable):** When the owner writes "GUNCELLEMELER.md içindeki U-XXX'i uygula", implement **only that update block**. Do not re-read or re-apply the master prompt, do not touch anything outside the block's "Files" list, do not refactor, rename, restyle or "improve" other parts of the site. Reuse existing components, tokens and utilities (GradientText, Horizon Line, chapter label, Embla carousel, ResponsiveImage, Seo). When done, reply with max 5 lines: files changed + anything that needs the owner's input. Then mark the block's status as `✅ Uygulandı (date)`.

## Credit-saving rules (apply to every update)

1. Work in a single pass; do not ask confirmation questions unless something in the block is impossible.
2. Touch only the files listed in the block. New files go exactly where the block says.
3. No global refactors, no dependency upgrades, no new libraries unless the block explicitly allows one.
4. Do not regenerate unchanged pages, content JSON, or tests of other features.
5. Do not produce long explanations, screenshots analysis or alternative proposals.
6. If a value is missing (e.g. a video ID), leave a clearly named placeholder in the data file — do not invent it.

---

## U-001 · Ana sayfaya YouTube kanal bölümü

**Status:** ⏳ Bekliyor
**Scope:** Home page only.
**Allowed new dependency:** none.

### Files
- `src/content/youtube.ts` — **new** (data only)
- `src/components/home/YouTubeSection.tsx` — **new**
- `src/components/home/VideoCard.tsx` — **new**
- `src/components/home/LiteYouTube.tsx` — **new** (click-to-load player)
- `src/pages/Home.tsx` (or the existing home page file) — insert the section only

### Placement
On the home page, insert the section **after the About ("Birlikte büyüyen işletmeler…") section and before the closing CTA**. Nothing else on the page moves.

### Data file `src/content/youtube.ts`

```ts
export const youtubeChannel = {
  name: 'Nextera Danışmanlık',
  url: 'https://www.youtube.com/@NexteraDan%C4%B1%C5%9Fmanl%C4%B1k', // @NexteraDanışmanlık
  subscribeUrl: 'https://www.youtube.com/@NexteraDan%C4%B1%C5%9Fmanl%C4%B1k?sub_confirmation=1',
};

export type ChannelVideo = {
  id: string;            // YouTube video ID (the part after watch?v=)
  title: string;         // exact YouTube title
  description?: string;  // optional, one short line, owner-provided
  uploadDate?: string;   // 'YYYY-MM-DD' (needed for VideoObject schema)
  pillar?: 'hibe-tesvik' | 'ihracat' | 'pazarlama' | 'yonetim' | 'insan-kaynaklari';
  featured?: boolean;    // exactly one video should be featured
};

// OWNER: paste your videos here (newest first). Leave the array empty to show only the channel CTA.
export const channelVideos: ChannelVideo[] = [
  // { id: 'VIDEO_ID_1', title: 'VIDEO_BASLIGI_1', uploadDate: '2026-01-01', pillar: 'hibe-tesvik', featured: true },
  // { id: 'VIDEO_ID_2', title: 'VIDEO_BASLIGI_2', uploadDate: '2026-01-01', pillar: 'ihracat' },
];
```

Do not fetch the channel, do not call the YouTube API, do not add an API key.

### Design (reference: owner's screenshot "video kampüs turu" layout, translated into the Nextera v3 design system)

**Block 1 — split intro (12-col grid, desktop 5/7, mobile stacked)**
- Left:
  - Chapter label with Horizon Line: `NEXTERA YOUTUBE`
  - H2: "Danışmanlığı **ekrandan** izleyin." (gradient on "ekrandan")
  - Lead (Inter 18px, `--text`): "Hibe, teşvik, ihracat, pazarlama, yönetim ve insan kaynakları üzerine uzman anlatımlarımızı videolarla keşfedin."
  - Buttons: primary (ink → purple on hover, 6px radius) "▶ İzlemeye başla" → scrolls/plays the featured video; secondary text link "Kanala abone ol →" → `subscribeUrl` (new tab, `rel="noopener"`).
- Right: **featured video** (the `featured: true` item, else the first) — 16:9, radius 12px, hairline border, soft shadow `0 24px 48px -32px rgba(14,10,24,.25)`. Thumbnail + centred play button (64px circle, Nextera Mor `#5C2FFC`, white triangle, 1.06 scale + gradient ring on hover). Title as a small caption under the frame (Inter 14px, `--muted`).

**Block 2 — video carousel**
- Row header: chapter label `SON VİDEOLAR` on the left; on the right two round arrow buttons (40px, 1px `--rule` border, ink arrow; hover border purple) + text link "Tüm videolar →" (`channel.url`, new tab).
- Embla carousel (existing dependency), `align: 'start'`, drag-free off, slide gap 20px. Visible cards: 4 (≥1280px), 3 (≥1024px), 2 (≥640px), 1.15 (mobile, next card peeks). Keyboard arrows work; arrows disabled at ends; a thin Horizon Line progress bar under the track.
- **VideoCard:** white card, radius 12px, 1px `--rule` border; thumbnail 16:9 on top with a 44px centred play button; **3px bottom accent line on the thumbnail** coloured by `pillar` (hibe-tesvik `#5C2FFC`, ihracat `#7E5BFD`, pazarlama `#C995C9`, yonetim `#EAB054`, insan-kaynaklari `#270F33`; default = Horizon gradient); body padding 16px: title (Inter 600, 16px, 2-line clamp, ink), optional description (Inter 14px, `--muted`, 2-line clamp). Hover: card lifts 4px, thumbnail scales 1.04, play button fills with the gradient ring.
- Clicking a card plays it **in the featured frame** (smooth scroll to Block 1 on mobile) instead of opening YouTube; the playing card gets an "Oynatılıyor" chip.
- Background: this section sits on a soft `--paper` tonal band with 140px fades top and bottom (counts as one tonal band on the home page).

**Empty state:** if `channelVideos` is empty → show only Block 1 with a static branded frame (Mürekkep/Derin Mor gradient, YouTube-style play icon, text "Nextera Danışmanlık YouTube kanalı") that links to the channel. No carousel.

### Performance & privacy (important)
- **Do not load the YouTube iframe on page load.** `LiteYouTube` shows only the thumbnail; the iframe is created on click.
- Thumbnails: `https://i.ytimg.com/vi_webp/{id}/hqdefault.webp` in `<picture>` with JPG fallback `https://i.ytimg.com/vi/{id}/hqdefault.jpg`; featured frame uses `maxresdefault` with `hqdefault` as `onError` fallback. `loading="lazy"`, `decoding="async"`, explicit `width`/`height` (aspect 16/9) → no CLS.
- Iframe: `https://www.youtube-nocookie.com/embed/{id}?autoplay=1&rel=0&modestbranding=1&playsinline=1`, `title` = video title, `allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture"`, `allowfullscreen`, `loading="lazy"`.
- Add `<link rel="preconnect" href="https://i.ytimg.com">` only on the home route. Preconnect to `www.youtube-nocookie.com` on first hover/focus of any play button (not before).
- The section component itself is lazy-loaded when within 400px of the viewport; the prerendered HTML still contains the heading, lead, channel link and video titles (for SEO/no-JS).
- Home route JS must not grow by more than ~6 KB gzip; report the diff.

### SEO & accessibility
- JSON-LD: one `VideoObject` per video that has `uploadDate` (name, description = description or title, thumbnailUrl, uploadDate, embedUrl, contentUrl = watch URL); skip items without `uploadDate`. Add `sameAs: channel.url` to the existing Organization schema.
- Play buttons are `<button>` with `aria-label="Videoyu oynat: {title}"`; carousel has `aria-roledescription="carousel"` and slide labels "3 / 8"; visible focus rings (Nextera Mor).
- Respect `prefers-reduced-motion` (no hover scaling, no smooth scroll).

### Acceptance
- [ ] Section appears between About and CTA on the home page only.
- [ ] No YouTube iframe/script requests until a play button is clicked (check Network tab).
- [ ] Works with 0, 1 and 8+ videos (empty state / no carousel / carousel with arrows).
- [ ] Mobile: stacked layout, swipe carousel, next card peeks, featured player on top.
- [ ] Lighthouse mobile on `/`: no drop in Performance score; CLS unchanged.

---

## U-002 · Başarı hikayelerine firma logoları

**Status:** ⏳ Bekliyor
**Scope:** Success-story visuals only (home carousel, `/basari-hikayeleri/` index cards, story detail hero, service-page story carousels).
**Allowed new dependency:** none.

### Files
- `src/content/client-logos.ts` — **new** (data only)
- `public/logos/clients/` — **new folder** (owner uploads logo files here)
- `src/components/stories/ClientLogoBadge.tsx` — **new**
- The existing story card / slide / detail-hero components — add the badge only

### Data
Do **not** edit `nextera-content.json`. Map logos by client name in a separate file:

```ts
// src/content/client-logos.ts — key = successStories[].display.clientName (exact)
export const clientLogos: Record<string, { src?: string; dark?: boolean }> = {
  'NovaTech Yazılım':                { src: '/logos/clients/novatech-yazilim.svg' },
  'Asteris Endüstriyel Ürünler':     { src: '/logos/clients/asteris-endustriyel-urunler.svg' },
  'Verda Food Processing':           { src: '/logos/clients/verda-food-processing.svg' },
  'ModaTex Apparel':                 { src: '/logos/clients/modatex-apparel.svg' },
  'ChemPro Cleaning Solutions':      { src: '/logos/clients/chempro-cleaning-solutions.svg' },
  'Metalpro Automation Systems':     { src: '/logos/clients/metalpro-automation-systems.svg' },
  'TechGrid Industrial Solutions':   { src: '/logos/clients/techgrid-industrial-solutions.svg' },
  'TechMotion Automation':           { src: '/logos/clients/techmotion-automation.svg' },
  'EnergoTech Industrial Solutions': { src: '/logos/clients/energotech-industrial-solutions.svg' },
  'FreshNatura Agro Processing':     { src: '/logos/clients/freshnatura-agro-processing.svg' },
  'Aydora Dairy Processing':         { src: '/logos/clients/aydora-dairy-processing.svg' },
  'Eggora Poultry Farm':             { src: '/logos/clients/eggora-poultry-farm.svg' },
  'OlivePure Natural Oils':          { src: '/logos/clients/olivepure-natural-oils.svg' },
  'SivaFeed AgroTech':               { src: '/logos/clients/sivafeed-agrotech.svg' },
};
// Accepted formats: .svg (preferred), .png or .webp with transparent background. `dark: true` = logo is light-coloured and needs a dark chip.
```

**Never draw, generate or recreate a company logo.** If a file is missing (404 or no `src`), show the fallback: the client name as plain text (Sora 600) in the same chip — a typographic label, not a logo imitation.

### Design — "the photo stays untouched"
- The photo keeps its current crop, size, colours and `object-fit`; no extra overlay gradient, no blur, no darkening of the photo.
- The logo sits in a **floating chip** on top of the photo: bottom-left, 16px inset (12px on mobile), white 92 % background with `backdrop-filter: blur(8px)`, 1px `rgba(14,10,24,.08)` border, radius 10px, padding 8px 12px, subtle shadow `0 8px 24px -12px rgba(14,10,24,.35)`. If `dark: true` → chip uses `--ink` at 88 %.
- Logo inside: `max-height: 28px` (home carousel & detail hero 36px), `max-width: 140px`, `object-fit: contain`, original colours, never stretched or recoloured; `alt="{clientName} logosu"`, `loading="lazy"` (detail hero: eager).
- On the home carousel the chip fades in 150 ms after the slide's photo reveal; reduced-motion = static.
- Chip must not cover faces/key subject: default bottom-left; allow a per-client override `position?: 'bottom-left' | 'bottom-right' | 'top-left'` in the data file.

### Acceptance
- [ ] All story surfaces show the chip; photos are pixel-identical to before (same crop and size).
- [ ] Missing logos fall back to the text chip without layout shift (reserve 44px chip height).
- [ ] No logo is AI-generated or redrawn.

---

## U-003 · Hizmet sayfası görsellerinin konu uyumu denetimi ve düzeltme

**Status:** ⏳ Bekliyor
**Scope:** Photos on the 42 service pages and the 19 success stories (+ their alt texts). No layout or component changes.
**Allowed new dependency:** none.

### Files
- `src/assets/photos/**` (replace mismatched files) and/or the image references in `src/pages-data/<slug>.ts` / wherever each page's images are defined
- `src/content/photo-credits.ts` (update credits)
- `docs/image-audit.md` — **new** (the audit report)

### Rule
Every photo must **visibly show the page's subject**. A visitor must understand the topic from the image alone. Generic office/handshake/laptop photos are only acceptable on pages whose subject *is* office work (e.g. bordro, SEO). Hibe/teşvik pages must show what the support is *for* (the machine, the farm, the solar field, the lab…), not paperwork.

### Procedure (one pass, no questions)
1. For each of the 42 service pages list every photo (hero, section images, results/R4 image, related-card thumbnail) in `docs/image-audit.md`: `page · slot · current file · what it shows · matches? (✅/❌) · action`.
2. Replace every ❌ with a real photo (Unsplash/Pexels, free licence) using the subject + search terms below. Keep the same aspect ratio and slot; run it through the existing image pipeline (AVIF/WebP, same size budgets); update alt text (Turkish, describes subject + service, e.g. "Tarlada hasat yapan traktör — TKDK kırsal kalkınma destekleri").
3. No photo may be reused on two service pages. No AI images, no visible brand logos, no watermarks, no readable personal documents.
4. Then check the 19 success-story photos against their sector (table B) the same way.
5. Report: number checked, number replaced, list of replaced files.

### A) Required subject per service page

| # | Page | Must show (hero / main) | Also acceptable | Search terms |
|---|---|---|---|---|
| 01 | Hibe ve Teşvik (hub) | Modern factory production line | Engineers in a plant, industrial zone aerial | modern factory production line; industrial engineers plant |
| 02 | KOSGEB | Small manufacturing workshop / SME owner at machine | Craftsman workshop, small CNC shop | small workshop cnc; craftsman workshop |
| 03 | TÜBİTAK | R&D laboratory, researcher with prototype | Electronics prototype bench | research laboratory scientist; electronics prototype |
| 04 | Ticaret Bakanlığı Destekleri | Export operations: containers being loaded | Export office with shipping docs | container loading export; shipping documents office |
| 05 | İhracat Teşvikleri | Container port with cranes | Cargo ship at quay | container port cranes; cargo ship port |
| 06 | Yatırım Teşvik Belgesi | New factory / heavy machinery installation | Factory construction, CNC machinery | industrial machinery installation; factory construction |
| 07 | Ar-Ge ve Tasarım Merkezi | Engineers at CAD / design centre | R&D open lab | engineers cad design; product design studio |
| 08 | AB ve Uluslararası Hibeler | Project meeting in a modern conference room / researchers collaborating | R&D lab, project documents | project meeting conference room; researchers collaboration |
| 09 | Turquality | Flagship retail store / branded product showroom | Product design showroom | flagship store interior; furniture showroom |
| 10 | Fuar Destekleri | Trade fair exhibition hall with booths | Stand construction, B2B meeting at fair | trade show exhibition hall; expo booth |
| 11 | Kırsal Kalkınma (TKDK) | **Tractor / agricultural machinery in the field** | Greenhouse, dairy farm, agro-processing | tractor harvest field; agricultural machinery; modern greenhouse |
| 12 | Marka, Tasarım ve Patent | Product designer sketching / prototype | Patent-style technical drawings | industrial designer sketch; technical drawing prototype |
| 13 | SGK Teşvik | Factory workers / shift team | HR payroll desk | factory workers team; shift workers |
| 14 | Sağlık Turizmi | Modern hospital / clinic interior | Doctor consultation | modern hospital interior; clinic consultation |
| 15 | İhracat Danışmanlığı (hub) | Aerial container ship at sea | Port, trucks | aerial container ship; logistics trucks |
| 16 | Yurtdışı Teşvikleri | Overseas warehouse / showroom | European logistics centre | warehouse logistics center; showroom |
| 17 | Pazara Giriş Belgesi | Quality control / testing laboratory | Product certification inspection | quality control laboratory; product testing |
| 18 | Yurt Dışı Pazar Araştırması | Analyst with market data / world map | Market street abroad | market research analysis; world map business |
| 19 | Pazar Araştırması Desteği | Business traveller at airport | Meeting with distributor abroad | business traveler airport; business trip meeting |
| 20 | İthalat Danışmanlığı | Warehouse with pallets and forklift | Goods receiving | warehouse pallets forklift; goods receiving |
| 21 | Gümrük Danışmanlığı | Customs checkpoint / container inspection | Trucks at border | customs border trucks; container inspection |
| 22 | Pazarlama (hub) | Marketing team strategy workshop | Brainstorm wall | marketing team brainstorming |
| 23 | Sosyal Medya | Content creator filming with smartphone | Social content shoot | content creator smartphone filming |
| 24 | SEO | Analytics dashboard on laptop | Analyst at screens | analytics dashboard laptop |
| 25 | Marka Danışmanlığı | Brand identity moodboard / packaging | Designer with colour swatches | brand identity moodboard; packaging design |
| 26 | Google Ads & Meta | Performance marketer at ad dashboards | Team reviewing campaign | digital advertising dashboard |
| 27 | Dijital Pazarlama | Digital team with multiple screens | Customer on mobile | digital marketing team |
| 28 | E-Ticaret | E-commerce packing station with boxes | Product photography setup | ecommerce packing boxes; product photography |
| 29 | Yönetim (hub) | Executive boardroom meeting | Strategy presentation | executive boardroom meeting |
| 30 | Kurumsal Eğitim | Corporate training / workshop | Mentoring | corporate training seminar |
| 31 | Kurumsallaşma | Process mapping on whiteboard / management meeting | Organised modern office | process planning whiteboard |
| 32 | Aile Şirketi | Multi-generation family in their business | Handover in workshop | family business generations |
| 33 | Aile Anayasası | Family meeting at table, signing documents | Heritage company building | signing agreement meeting table |
| 34 | İnsan Kaynakları (hub) | Team meeting in a Turkish office | Onboarding session | turkish business team office; istanbul office meeting |
| 35 | Outsource Bordro | Payroll/accounting specialist at work | Accounting office | accountant payroll laptop |
| 36 | İşe Alım | Job interview | Candidate welcome | job interview office |
| 37 | Çalışan Memnuniyeti Anketi | Employees giving feedback / survey on tablet | Team retrospective | employee feedback tablet; team meeting feedback |
| 38 | Yabancı Çalışma İzni | Passport & work-permit documents on a desk (no readable data), HR desk | Airport arrival hall (no identifiable faces) | passport documents desk; work permit documents; airport arrivals |
| 39 | OKR | Team setting goals on board | Sprint planning | team goal planning whiteboard |
| 40 | Güneş Enerjisi (GES) | **Solar panel field / rooftop solar on factory** | Technician installing panels | solar panel farm; rooftop solar factory; solar installation |
| 41 | Marka Reklam Filmi | Film set with camera crew | Editor at timeline | film production set camera; video editing |
| 42 | Kurumsal Psikolojik Danışmanlık | Calm one-to-one conversation (non-clinical) | Team walking outdoors, quiet space | calm conversation office; workplace wellbeing |

### B) Success stories — required sector subject

| Client | Must show |
|---|---|
| NovaTech Yazılım | Software team office / recruitment interview |
| Asteris Endüstriyel Ürünler (İhracat) | Industrial products / furniture production & export loading |
| Asteris (Ticaret Bakanlığı) | Export logistics / trade fair |
| Asteris (Yatırım teşvik – Bursa) | Metal processing / CNC machining |
| Asteris (KOSGEB – İzmir) | Machinery & automation workshop |
| Asteris (Ar-Ge Merkezi – Kocaeli) | Automation R&D lab |
| Asteris (KOSGEB – Konya) | Industrial production floor |
| Verda Food Processing | Food processing & packaging line |
| ModaTex Apparel | Textile / garment production |
| ChemPro Cleaning Solutions | Chemical / cleaning-products production |
| Metalpro Automation Systems | Industrial automation / robotics |
| TechGrid Industrial Solutions | Industrial IoT / smart factory |
| TechMotion Automation | Sensors & automation engineering |
| EnergoTech Industrial Solutions | Energy-efficiency systems / control panels |
| FreshNatura Agro Processing | Fruit/vegetable agro-processing |
| Aydora Dairy Processing | Dairy processing (milk tanks, cows) |
| Eggora Poultry Farm | Poultry farm / egg production |
| OlivePure Natural Oils | Olive harvest / olive-oil press |
| SivaFeed AgroTech | Feed mill / silos |

The 6 Asteris stories must each have a different photo.

### Acceptance
- [ ] `docs/image-audit.md` lists every photo with ✅/❌ and action.
- [ ] Every page's hero clearly matches table A (e.g. page 11 shows agricultural machinery, page 40 shows solar panels).
- [ ] No reused photos, credits updated, alt texts updated, image budgets respected.

---

## U-004 · Blog altyapısı (yönetim panelinden içerik)

**Status:** ⏳ Bekliyor
**Scope:** New blog data source + `/blog/` list and `/blog/<slug>/` detail pages + a minimal admin to publish posts. (This block explicitly overrides the master prompt's "no blog" rule.)
**Allowed new dependency:** none (use existing Supabase client, react-hook-form, zod).

### Data source (adapter)
Create `src/features/blog/source.ts` with one interface and two adapters; select via `BLOG_SOURCE` in `site.ts`:
- `'supabase'` (default): table + storage below, admin at `/yonetim/`.
- `'wordpress'`: if the owner keeps the current WordPress as the admin, read `https://nexteradanismanlik.com/wp-json/wp/v2/posts?_embed&per_page=…&orderby=date` and map `title.rendered`, `excerpt.rendered` (strip HTML), `date`, `slug`, `_embedded['wp:featuredmedia'][0].source_url`. In this mode skip the Supabase admin.

```ts
interface BlogPost { slug: string; title: string; excerpt: string; coverUrl: string; coverAlt: string;
  publishedAt: string; category?: 'hibe-tesvik'|'ihracat'|'pazarlama'|'yonetim'|'insan-kaynaklari';
  readingMinutes?: number; contentHtml?: string; seoTitle?: string; seoDescription?: string }
```

### Supabase (default mode)
- Table `posts`: `id uuid pk`, `slug text unique`, `title text`, `excerpt text`, `content_md text`, `cover_path text`, `cover_alt text`, `category text`, `status text check in ('draft','published')`, `published_at timestamptz`, `seo_title text`, `seo_description text`, `created_at`, `updated_at`.
- RLS: public `select` only where `status='published' and published_at <= now()`; insert/update/delete only for users in table `admins(user_id)`.
- Storage bucket `blog-images` (public read, admin write).
- On upload the admin compresses in the browser (canvas → WebP, quality 0.8) into two sizes: 1600 px and 800 px wide (`cover_path` = base path; files `…-1600.webp`, `…-800.webp`).

### Admin `/yonetim/` (noindex, excluded from sitemap)
Supabase email login; list of posts (status, date), create/edit form: title, auto-slug (Turkish-safe, editable), category, excerpt (max 180 chars), cover upload with preview + alt text (required), markdown editor with preview (textarea + preview is enough), SEO title/description, publish date, draft/publish toggle. Minimal UI in the v3 style; no extra libraries.

### Public pages
- `/blog/` — H1 "Blog", category tabs, grid of posts (cover, category, date, title, excerpt), pagination 12/page.
- `/blog/<slug>/` — article layout (editorial, max 68ch), cover image, reading time, "İlgili hizmet" link by category, share links, `Article` + `BreadcrumbList` JSON-LD.
- Freshness: pages are prerendered at build **and** revalidate on the client (fetch latest on mount; if a slug was not prerendered, render client-side instead of 404). If the hosting supports a deploy hook, call it on publish (Supabase database webhook on `posts` update where status becomes 'published'); otherwise add a note in `/docs/blog.md` that a republish refreshes the prerendered HTML.
- Add `/blog/` to header nav and footer; add published posts to `sitemap.xml` at build.

### Acceptance
- [ ] A post published in the admin appears on `/blog/` and on the home section (U-005) without code changes.
- [ ] Covers are served as WebP in two sizes with `srcset`; alt text required.
- [ ] RLS verified: anonymous users cannot read drafts or write.

---

## U-005 · Ana sayfaya "Son yazılar" blog bölümü

**Status:** ⏳ Bekliyor (U-004'ten sonra uygulanır)
**Scope:** Home page only.
**Allowed new dependency:** none.

### Files
- `src/components/home/LatestPosts.tsx` — **new**
- Home page file — insert the section only

### Placement
After the YouTube section (U-001) and before the closing CTA.

### Content & data
- Reads the **latest 5 published posts** via `source.ts` (`limit 5`, order by `publishedAt desc`); prerendered at build, then refreshed on the client so new posts appear automatically.
- Header row: chapter label `BLOG`, H2 "Güncel **içgörüler**" (gradient on "içgörüler"), text link "Tüm yazılar →" (`/blog/`).

### Layout — the newest post sits higher
- Desktop (≥1024px), 12-col grid:
  - **Newest post (featured):** cols 1–7, tall card, image 4:3 (or 16:10), **positioned 32px higher than the rest of the row** (negative top offset / `translateY(-32px)` with matching bottom padding so nothing overlaps the header), larger title (Sora 28px), excerpt 3 lines, "Yeni" chip with gradient border, date + category.
  - **Posts 2–5:** cols 8–12 as a vertical list of 4 compact horizontal cards (thumbnail 120×90 left, category · date, title 2-line clamp), separated by hairlines. If only 3 posts exist, show 2 rows; if 1, featured only.
- Tablet: featured full width on top (still raised with a -16px offset), others in a 2-column grid.
- Mobile: featured card first, then posts 2–5 as a horizontal swipe row (Embla, 1.15 cards visible).
- Cards: white, radius 12px, 1px `--rule` border; image hover scale 1.04; title underline animation; whole card is one link.
- Images: `srcset` from the 800/1600 WebP versions (WordPress mode: `source_url`), `loading="lazy"`, explicit aspect ratio (no CLS); featured image `sizes="(min-width:1024px) 58vw, 100vw"`.
- Empty state (0 posts): section is not rendered.
- Skeleton while client refresh runs only if there is no prerendered data.

### Acceptance
- [ ] Shows the 5 newest posts; newest is visibly raised and larger.
- [ ] Publishing a new post in the admin updates the section on next visit.
- [ ] Home route JS grows ≤ 5 KB gzip; no CLS.

---

## U-006 · Hakkımızda sayfası: yeni içerik + SSS

**Status:** ⏳ Bekliyor
**Scope:** `/hakkimizda/` only (this block replaces the page's previous content and its "Hakkımızda metni eklenecek" placeholder).
**Allowed new dependency:** none.

### Files
- `src/content/about.ts` — **new** (all texts below, verbatim)
- `src/pages/About.tsx` (existing about page) — rebuild the page body
- Reuse existing: GradientText, chapter label, Horizon Line, StatCounter, creative-card styles, accordion, CtaPanel, Seo

### Content (use exactly as written)

```ts
export const about = {
  hero: {
    label: 'Hikayemiz',
    title: 'Karmaşık süreçleri anlaşılır sonuçlara dönüştürüyoruz',   // gradient: "anlaşılır sonuçlara"
    paragraphs: [
      'Nextera olarak, iş dünyasının değişen dinamiklerine uyum sağlayan; hibe, teşvik, yönetim, dijital ve ihracat danışmanlık hizmetleriyle markaların sürdürülebilir başarısını inşa ediyoruz.',
      'Yatırımlarınızın geri dönüşünü maksimize etmek, fırsatları erken yakalamanızı sağlamak ve süreçleri sizin yerinize yönetmek için buradayız. Her sektörde, her ölçekte işletmeye özel çözümler tasarlıyoruz.',
    ],
    cta: { label: 'Bizimle çalış', href: '/iletisim/' },
    badge: { value: '30+', label: 'Tamamlanan Proje' },
  },
  stats: [
    { value: '30+', label: 'Tamamlanan Proje', text: 'Farklı sektörlerde sonuçlandırdığımız danışmanlık projesi' },
    { value: '20+', label: 'Uzman Ekip', text: 'Tam zamanlı danışman ve sektör uzmanı' },
    { value: '12+', label: 'Farklı Sektör', text: "KOBİ'den kurumsala, geniş sektörel deneyim" },
    { value: '3',   label: 'Faaliyet Bölgesi', text: 'Marmara, İç Anadolu ve Doğu Anadolu' },
  ],
  values: {
    label: 'Değerlerimiz',
    title: 'Çalışmamızı tutarlı kılan ilkelerimiz',   // gradient: "ilkelerimiz"
    items: [
      { title: 'Veri odaklı strateji', text: 'Her kararı somut verilerle alıyor, sezgisel değil ölçülebilir büyüme tasarlıyoruz.' },
      { title: 'Şeffaf süreç', text: 'Başvurudan raporlamaya kadar tüm adımları görünür kılıyor, sorumluluğu paylaşıyoruz.' },
      { title: 'Hız ve disiplin', text: 'Fırsatlar zamana duyarlıdır; ekibimiz hızlı, planlı ve tutarlı bir şekilde sonuç üretir.' },
      { title: 'Etik ve uyum', text: 'Mevzuat ve etik standartlara tam uyum içinde çalışır, uzun vadeli güven inşa ederiz.' },
    ],
  },
  journey: {
    label: 'Yolculuğumuz',
    title: 'Kısa sürede kat ettiğimiz mesafe',   // gradient: "kat ettiğimiz mesafe"
    steps: [
      { when: '2024', title: 'Kuruluş', text: "Nextera Danışmanlık, İstanbul'da uçtan uca hizmet sunan bir danışmanlık firması olarak kuruldu." },
      { when: '2025 / 1. Çeyrek', title: 'İlk büyüme dalgası', text: 'Hibe, teşvik ve ihracat danışmanlığı hatları açıldı; ilk kurumsal müşterilerle çalışmalara başladık.' },
      { when: '2025 / 2. Yarı', title: 'Ekip ve hizmet genişlemesi', text: 'Yönetim, insan kaynakları, finansal raporlama ve dijital pazarlama ekipleri eklendi; tam zamanlı kadro 20+ kişiye ulaştı.' },
      { when: '2026', title: 'Bugün', text: "30+ tamamlanan proje, 12+ farklı sektör, Marmara - İç Anadolu - Doğu Anadolu'da aktif danışmanlık." },
    ],
  },
  faq: {
    label: 'Sıkça Sorulan Sorular',
    title: 'Nextera hakkında merak edilenler',   // gradient: "merak edilenler"
    items: [
      { q: 'Nextera Danışmanlık kimdir, hangi alanlarda hizmet verir?', a: '2024 yılında İstanbul’da kurulan Nextera Danışmanlık; hibe ve teşvik, ihracat, ithalat, yönetim, insan kaynakları, dijital pazarlama ve finansal danışmanlık alanlarında hizmet verir. Bugüne kadar 30’dan fazla projede çözüm sunduk.' },
      { q: 'Ekibi kaç kişidir, nerelerde faaliyet gösterir?', a: '20’den fazla tam zamanlı danışman ve sektör uzmanından oluşan bir ekibiz. İstanbul merkezli olarak Marmara, İç Anadolu ve Doğu Anadolu bölgelerinde faaliyet gösteriyoruz.' },
      { q: 'Hizmetlerden kimler yararlanabilir?', a: 'Genç ve kadın girişimcilerden KOBİ’lere, kurumsal şirketlerden büyük kuruluşlara kadar her ölçekte işletme hizmetlerimizden yararlanabilir. Her projeyi müşterimizin yapısına ve ihtiyacına göre şekillendiriyoruz.' },
      { q: 'İhracat ve ithalatta nasıl destek sağlar?', a: 'İhracatta firmanızın mevcut durumunu, hedef ülkeleri ve pazarları analiz ederek yol haritanızı oluşturuyoruz. İthalatta ise doğru satıcıyı bulmaktan gümrük süreçlerine kadar her adımda destek sağlıyoruz.' },
      { q: 'Kurumsallaşma ve yönetim danışmanlığı ne kazandırır?', a: 'Şirket kaynaklarınızın hedeflerinize uygun şekilde kullanılmasına ve bu hedeflere sürdürülebilir biçimde ulaşmanıza yardımcı olmayı amaçlıyoruz.' },
      { q: 'İK ve organizasyonel gelişim hangi süreçleri kapsar?', a: 'Organizasyon yapınızın düzenlenmesini ve uygun pozisyonlara doğru kişilerin işe alınmasını kapsar. Amacımız zaman ve maliyet kayıplarını azaltırken verimliliği ve çalışan bağlılığını artırmaktır.' },
      { q: 'Nextera ile çalışmanın avantajları nelerdir?', a: 'Farklı ölçeklerdeki firmalara hizmet verebilen, genç ve deneyimli bir ekiple çalışırsınız. Süreci veri odaklı, şeffaf ve mevzuata tam uyum içinde yürütürüz.' },
      { q: 'Süreç nasıl başlar?', a: 'İletişim formunu göndermenizin ardından uzaktan bir ön görüşme yapıyoruz. Bunu ihtiyaç analizi, saha ziyareti ve proje uygulaması izliyor.' },
      { q: 'Danışmanlık ne kadar sürer?', a: 'Süre projenin kapsamına göre değişir: kısa projeler birkaç hafta sürerken kapsamlı dönüşüm projeleri 6–12 ay sürebilir.' },
      { q: 'Ücretler nasıl belirlenir?', a: 'Ücretler; proje kapsamı, süresi, şirket büyüklüğü ve gereken uzmanlığa göre belirlenir. Fiyatlandırmayı ilk görüşmede sizinle paylaşıyoruz.' },
    ],
  },
  cta: {
    label: 'Birlikte Çalışalım',
    title: 'Geleceğinizi şekillendiren danışmanlıkla tanışın',   // gradient: "danışmanlıkla"
    text: 'Hibe, teşvik, ihracat ve dijital alanlarda stratejik çözümlerle işletmenizi bir adım öteye taşıyoruz.',
    buttons: [{ label: 'Ön görüşme ayarla', href: '/iletisim/' }, { label: 'Hizmetleri keşfet', href: '/hizmetler/' }],
  },
};
```

### Page composition (v3 design system; white-dominant; max 2 tonal bands)
1. **Breadcrumb** Ana Sayfa / Hakkımızda. **"Bu sayfada"** TOC (desktop left rail): Hikayemiz · Rakamlarla Nextera · Değerlerimiz · Yolculuğumuz · SSS.
2. **Hero "Hikayemiz"** — the page `<h1>` is `hero.title` (gradient on "anlaşılır sonuçlara"). Editorial split: text cols 1–7 (label, H1, two paragraphs, primary button "Bizimle çalış"); right cols 8–12 a tall real photo (Nextera-like consulting team in an İstanbul office, no logos) with the **badge "30+ Tamamlanan Proje"** as a small white chip overlapping the photo's lower-left corner (Space Grotesk value with gradient, label in Inter). Not the old two-photo collage.
3. **Stats band "Rakamlarla Nextera"** (visually hidden H2 for outline) on a `--mist` tonal band with soft fades: 4 columns separated by vertical hairlines; value in Space Grotesk 72px with gradient + count-up once; label Inter 600; text `--muted`. Mobile: 2×2 grid.
4. **Değerlerimiz** — chapter label + H2 (gradient on "ilkelerimiz"); 4 **creative cards** in a 2×2 bento (desktop) with a large outline number 01–04, title, text; hover: gradient border + 4px lift; mobile: single column. Each card gets a small line icon (lucide: `BarChart3`, `Eye`, `Zap`, `ShieldCheck`) — only these four.
5. **Yolculuğumuz** — chapter label + H2; **horizontal timeline**: one 1px line with 4 nodes, `when` above in Space Grotesk, title + text below; the Horizon Line fills from left to right as the section scrolls into view; the last node ("Bugün") has a pulsing gradient ring (once). Mobile: vertical timeline. Use an ordered list (`<ol>`) semantically.
6. **SSS** — chapter label + H2 (gradient "merak edilenler"); two-column layout on desktop: left column sticky intro (label, H2, a one-line hint "Sorunuz burada yoksa bize yazın →" linking to `/iletisim/`), right column the accordion. Accordion rows: hairline separators, question in Inter 600 18px, plus/minus icon that rotates, smooth height animation; first item open. Mobile: single column.
7. **CTA "Birlikte Çalışalım"** — reuse the site's CtaPanel with `cta` content (gradient on "danışmanlıkla").

### SEO (Google must read it)
- All FAQ answers must be in the prerendered HTML even when collapsed: use `<details>/<summary>` or keep panels in the DOM with `hidden="until-found"` — **no conditional rendering**. Each question is an `<h3>` inside the summary/button.
- JSON-LD on this page (one `@graph`): `AboutPage` + `Organization` (reuse the site's @id; add `foundingDate: "2024"`, `address: { addressLocality: "İstanbul", addressCountry: "TR" }`, `numberOfEmployees: { "@type": "QuantitativeValue", minValue: 20 }`, `areaServed: ["Marmara Bölgesi","İç Anadolu Bölgesi","Doğu Anadolu Bölgesi"]`) + `FAQPage` with the 10 Q&As exactly as rendered + `BreadcrumbList`.
- Title: "Hakkımızda | Nextera Danışmanlık". Meta description: "2024’te İstanbul’da kurulan Nextera Danışmanlık; 20+ uzman, 30+ tamamlanan proje ve 12+ sektör deneyimiyle hibe, teşvik, ihracat, yönetim, İK ve dijital pazarlama danışmanlığı sunar."
- Heading outline: one H1 (hero), H2 per section, H3 per value card / timeline step / FAQ question.

### Site-wide consistency fix (small, required)
The company was founded in 2024, so remove every "12+ Yıllık Sektör Deneyimi" / "12+ yıllık deneyim" text anywhere on the site (home about block, key-facts rows, footer, schema) and replace it with **"12+ Farklı Sektör"**. Do not change anything else in those components.

### Acceptance
- [ ] `/hakkimizda/` renders all texts above verbatim, in this order.
- [ ] View-source shows all 10 FAQ answers; Rich Results Test validates FAQPage + Organization.
- [ ] No "12+ yıllık" wording remains anywhere (search the codebase).

---

## U-007 · Ana sayfa: hero’nun hemen altına kısa Hakkımızda bölümü

**Status:** ⏳ Bekliyor (U-006 ile birlikte veya sonra)
**Scope:** Home page only.
**Allowed new dependency:** none.

### Files
- `src/components/home/AboutTeaser.tsx` — **new** (reads from `src/content/about.ts`)
- Home page file — insert directly **below the hero**; **remove the old home About block** (the "Birlikte büyüyen işletmeler için buradayız" section further down) so the page does not have two about sections. Nothing else moves.

### Design (compact — must not push the practice-area section far down)
- Height target: ≤ 520px desktop.
- Left (cols 1–6): chapter label `HAKKIMIZDA`; H2 = `about.hero.title` (gradient on "anlaşılır sonuçlara"); first paragraph of `about.hero.paragraphs`; text link **"Hikayemizi keşfedin →"** → `/hakkimizda/` (plus the whole block's H2 is also a link to the page).
- Right (cols 7–12): the 4 `about.stats` as a 2×2 mini grid with hairline dividers (value Space Grotesk 44px gradient + label; no long text), count-up once when visible.
- Background white; a single 1px ink rule above the section; no photo (the hero directly above already has the large photo).
- Mobile: text first, stats as a 2×2 grid, link as a full-width secondary button.

### Acceptance
- [ ] Section appears immediately under the hero and links to `/hakkimizda/`.
- [ ] The old home About block is gone; no duplicate "12+ yıllık" wording.
- [ ] No layout shift; home JS grows ≤ 2 KB gzip.

---

## U-008 · Fotoğraflarda insan seçimi: yerel iş ortamı

**Status:** ⏳ Bekliyor
**Scope:** Every photo on the site that shows people (service pages, success stories, home, about, blog defaults). Overrides all earlier "diverse/international team" photo instructions.
**Allowed new dependency:** none.

### Files
- `src/assets/photos/**` (replacements), image references, `src/content/photo-credits.ts`
- `docs/image-audit.md` — add a section "U-008 insan içeren görseller"

### Rule
- Photos with people must look like **an authentic Turkish business setting**: professionals as one would typically meet them in Istanbul/Anatolian offices, factories, farms and ports; Turkish-looking workplaces, Mediterranean look and feel. The visitor should feel "this is a firm in Türkiye working with companies like mine".
- Search terms to prefer: "turkish business people", "istanbul office meeting", "turkish engineer factory", "turkish farmer tractor", "mediterranean business team", plus the subject terms from U-003.
- Go through every people photo and **replace any that does not match this local setting** (same slot, same aspect ratio, same subject from U-003 table A/B, pipeline and budgets unchanged). Prefer subject-first images (machines, hands at work, over-the-shoulder, backs, small figures in wide shots) where a close-up face is not needed.
- Keep U-003 rules: subject match, no reuse, no logos, no AI images.
- Log every replaced file in `docs/image-audit.md`.

### Acceptance
- [ ] All people photos reflect a local Turkish business context.
- [ ] Every replaced file is listed in the audit with its new credit.

---

## U-009 · Ana sayfa: sadece ana hizmetler carousel’i (yatay genişleyen kartlar)

**Status:** ⏳ Bekliyor
**Scope:** Home page services area only.
**Allowed new dependency:** none (Embla already exists; the expanding row is plain CSS/React).

### Files
- `src/components/home/PracticeAreaCarousel.tsx` — **new**
- `src/content/home-practice-areas.ts` — **new** (data only)
- Home page file — replace the current practice-areas section with the new component; **remove** from the home page any other service-list blocks, per-area photo panels and links to child services (e.g. the five tall accordion panels with photos and service lists, and the hero's mini-panel switcher if present). Hero, About teaser (U-007), success stories, YouTube, blog and CTA stay.

### Content
- Header (existing verbatim texts): label `ÖNE ÇIKAN HİZMETLER`; H2 "İşletmenizin başarısı için **stratejik çözümler**"; lead "Finans, dijital, yönetim ve teşvik alanlarında bütünleşik hizmet sunuyoruz."
- Cards = the 5 practice-area hubs (data file allows a 6th later):

```ts
export const homePracticeAreas = [
  { no: '01', title: 'Hibe ve Teşvik',        href: '/hizmetler/hibe-tesvik-danismanligi/',    icon: 'Landmark',   tone: ['#5C2FFC', '#270F33'] },
  { no: '02', title: 'İhracat ve Dış Ticaret', href: '/hizmetler/ihracat-danismanligi/',        icon: 'Ship',       tone: ['#3F2A9E', '#0E0A18'] },
  { no: '03', title: 'Pazarlama',             href: '/hizmetler/pazarlama-danismanligi/',      icon: 'Megaphone',  tone: ['#B0689F', '#3A1432'] },
  { no: '04', title: 'Yönetim',               href: '/hizmetler/yonetim-danismanligi/',        icon: 'Compass',    tone: ['#D7962F', '#4A2A08'] },
  { no: '05', title: 'İnsan Kaynakları',       href: '/hizmetler/insan-kaynaklari-danismanligi/', icon: 'Users',   tone: ['#7E5BFD', '#1E1446'] },
];
// description  = that hub's metaDescription from nextera-content.json (verbatim)
// keywords line = the first 3 child-service menu labels of that area, joined with " · " (uppercase via CSS)
// meta line     = "{n} hizmet" (count of services in the area) — computed
```

### Design — horizontal version of the reference (no vertical text)
- **Desktop (≥1024px): expanding card row.** One row, height 440px, gap 12px. The **active card** takes the remaining width (flex-grow) and shows: number `01` + icon (top), title (Sora 40px, white), keywords line (Inter 12px, caps, letter-spacing .12em, white 70 %), the verbatim description (3 lines max), a hairline, meta line "{n} hizmet", and a button "İncele ↗" (white glass chip) → hub page.
  **Inactive cards** are ~168px wide, **text stays horizontal**: number + icon at the top, title at the bottom in 2 lines (Inter 600, 16px, white), no description.
  Background of every card = its `tone` gradient (diagonal, top-left → bottom-right) + a subtle large outline icon watermark (8 % white) — **no photos**, no links to child services.
- Interaction: hover or focus on an inactive card expands it (width transition 500 ms, `cubic-bezier(.22,1,.36,1)`; content fades in 150 ms after); click anywhere on the active card goes to the hub. Auto-advance every 6 s while the section is in view and not hovered; pauses on hover/focus; off with reduced motion.
- Controls under the row: progress indicator "01 / 05" with a Horizon Line segment filling for the active card, prev/next round buttons.
- **Tablet (640–1023px):** same idea with 3 visible cards (active wide + 2 narrow), the rest reachable by arrows/swipe.
- **Mobile (<640px):** Embla horizontal carousel of full cards (88 % width, next card peeks), each card shows all content (title, keywords, description, "İncele"); dots + swipe.
- Accessibility: the row is a `role="tablist"`-free list of links; each card is one `<a>` with `aria-label="{title} hizmetlerini incele"`; arrow keys move focus/expand; visible focus ring. All 5 descriptions stay in the DOM (prerendered) for SEO.
- Radius 16px; shadow `0 30px 60px -30px rgba(14,10,24,.45)` on the active card only; white page background around it.

### Acceptance
- [ ] The home page has exactly one services block: this carousel.
- [ ] Each card links only to its hub page; no child-service links, no photos inside cards.
- [ ] Desktop expanding behaviour, tablet 3-up, mobile swipe all work; auto-advance pauses on hover/focus.
- [ ] No CLS; home JS grows ≤ 4 KB gzip.

---

## Template for next updates (copy below)

```md
## U-00X · <kısa başlık>
**Status:** ⏳ Bekliyor
**Scope:** <which page/feature only>
**Allowed new dependency:** none
### Files
- <exact paths>
### Change
- <bullet list, concrete values>
### Acceptance
- [ ] <checks>
```
