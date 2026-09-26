# NEXTERA DANIŞMANLIK — LOVABLE MASTER PROMPT (v3)

> Prompt language: English · Website language: **Turkish (tr-TR)** · Attached content file: **`nextera-content.json`**
> Build phase by phase (§16). After every phase: stop, summarise, list anything you could not do, wait for my "devam".

---

## 0. MISSION IN ONE PARAGRAPH

Build a brand-new corporate website for **Nextera Danışmanlık** (Turkish consulting firm: grants & incentives, export & foreign trade, marketing, management, HR). **The text content of the current website is kept word for word. The interface is completely new.** The new interface must carry the prestige of **PwC, EY, KPMG and Deloitte** — calm, editorial, grid-disciplined, confident — and it must feel alive: every service page gets **new interactive, user-focused parts** (panels, schemas, drafts, processes, carousels, creative cards) that show what working with Nextera looks like in practice. The `/hizmetler/bordro-yonetimi/` page with its payroll dashboard is the model: verbatim text + a realistic interactive product screen. Every one of the 42 service pages gets the same level of treatment, each in its own original way.

---

## 1. NON-NEGOTIABLE RULES

1. **Verbatim text.** Every string in `nextera-content.json` is rendered exactly as written — no rewriting, shortening, merging, translating, "improving" or punctuation fixes. Render from the JSON; never hardcode copy inside components.
2. **Arrangement is free, words are not.** You may re-order and re-compose the sections of a page, turn a list into an interactive component, put stats into a key-facts strip, split a section into chapters. You may not change or drop any word.
3. **Do NOT reuse the old website's interface.** Do not look at, fetch or imitate nexteradanismanlik.com. The banned "old-site DNA" is listed in §3.8. The only visual element carried over is the **gradient headline colouring** (§3.2) — it stays and is used generally.
4. **No new pages or text sections invented by you.** The only new section of the site is **`/basari-hikayeleri/`** (content from the JSON, §9). Do not create blogs, guides, insights, sectors, platform/showcase pages, team pages, testimonials, client logos, awards or SEO text blocks.
5. **New interactive parts carry microcopy only.** The interactive parts you add (§6) may contain: a short title (≤ 6 words), a one-line caption (≤ 16 words), UI labels, axis labels, button labels, status names and clearly fictional sample data. No new paragraphs of marketing or SEO prose. Wherever possible, the interactive parts **re-use the page's own verbatim text** (process steps, explore cards, scope items, stats, FAQ) instead of new text.
6. **Sample data is always labelled.** Every panel/draft/simulation carries the badge **"Temsili ekran"** or **"Taslak · Örnektir"** and uses fictional names ("Örnek A.Ş.", "Personel-017"). No real personal data, no invented statistics presented as Nextera facts, no invented legal rates or deadlines. Numbers shown as results come from that page's own `results` stats (e.g. "%50-70") or are obviously fictional demo figures inside a labelled panel.
7. **URLs are preserved exactly:** `/hizmetler/<slug>/` with trailing slash (see §4.2).
8. **Background predominantly white** (≥ 80 % of every page's scroll length). Subtle tonal transitions are welcome (§3.1).
9. **Mobile-first, SEO-first, prerendered, fast** (§11–§14). Real photos in WebP, compressed.
10. **No third-party logos** (no ministry, Google, Meta, Amazon, Trendyol… logos) inside panels or anywhere. Text labels only.

---

## 2. TECH STACK

- Vite + React 18 + TypeScript + React Router, Tailwind CSS, shadcn/ui (Radix) primitives, lucide-react icons, framer-motion, recharts (small custom SVG where lighter), **Embla Carousel** for all carousels, dnd-kit where drag is needed.
- **Static prerendering of every public route** (`vite-react-ssg` preferred). Every route must ship complete HTML (title, meta, H1, all verbatim text, JSON-LD) without JavaScript. If the environment blocks this, tell me before Phase 1 ends and propose the closest working alternative.
- Head: `react-helmet-async` (or the SSG head API) via one `<Seo>` component.
- Images: `vite-imagetools` (or equivalent) → AVIF/WebP + `srcset`. One `<ResponsiveImage>` component.
- Fonts self-hosted woff2 with latin-ext (ç ğ ı İ ö ş ü): **Sora** 600/700, **Inter** 400/500/600, **Space Grotesk** 500/600. `font-display: swap`; preload Sora 700 + Inter 400 only.
- Forms: react-hook-form + zod; submissions to a Supabase table `contact_requests` (Supabase is connected) or a clearly marked placeholder handler. KVKK consent required. Honeypot anti-spam.
- Each interactive part is its own lazy chunk, preloaded 400 px before entering the viewport. A static prerendered fallback (title, caption, key labels, short `aria` description) is always in the HTML.

---

## 3. DESIGN SYSTEM — "NEXTERA CLARITY"

A new visual language: Big Four editorial discipline (ink typography, strict grid, hairline structure, large photography) + Nextera's own warmth (gradient headline words, soft tonal transitions, crafted interactive components).

### 3.1 Colour & background

| Token | Hex | Use |
|---|---|---|
| `--ink` (Mürekkep) | `#0E0A18` | Headlines, primary buttons, footer |
| `--text` | `#3D3950` | Body text |
| `--muted` | `#6E6A80` | Captions, meta, labels |
| `--rule` | `#E6E3EE` | 1 px rules, borders |
| `--purple` (Nextera Mor) | `#5C2FFC` | Links/active states, key data, focus ring, primary buttons on hover |
| `--lilac` (Lila) | `#7E5BFD` | Secondary data, hover tints |
| `--deep` (Derin Mor) | `#270F33` | Panel sidebars, one optional deep band per page |
| `--white` | `#FFFFFF` | Base background |
| `--paper` | `#FBFAFD` | Tonal band 1 |
| `--mist` | `#F4F2FA` | Tonal band 2 (stages for panels, key-facts strip) |
| `--rose` | `#C995C9` | Gradient mid stop |
| `--amber` | `#EAB054` | Gradient end stop |
| status | `#12A87A` / `#E59A1B` / `#E0475B` | Status pills inside panels only |

- **Background system:** white base; up to 3 tonal bands (`--paper`/`--mist`) per page, each entering and leaving with a soft 120–160 px vertical fade so sections melt into each other — no hard edges. Optionally one deep band (`--deep` → `--ink`, very subtle diagonal) per page, typically the stage of the main panel or the CTA. Footer is ink. No blobs, no dotted grids, no noise, no glass.

### 3.2 Gradient headlines (keep and use generally)

- Signature gradient: `linear-gradient(90deg, #6A45FF 0%, #8E6CF7 30%, #C995C9 68%, #EAB054 100%)`.
- **Use it in most H1s and H2s:** 1–3 consecutive key words per headline are gradient-filled (`background-clip:text`), e.g. "Bordro Süreçlerinizi **Profesyonellere** Bırakın", "Dört Adımda **Outsource Bordro** Süreci". The words themselves are unchanged — only wrapped in `<GradientText>`.
- Choose highlight words per headline in `src/content/highlights.ts` (slug → section index → exact substring). Rule of thumb: the service keyword or the promise word. Never highlight the whole headline; never more than one gradient span per headline; H3s stay ink.
- On desktop the gradient drifts very slowly (background-position, 10 s, alternate); static with reduced motion.
- The same gradient also appears as the **Horizon Line**: 3 px bar at the top of the header, reading-progress line, active tab underline, 2 × 48 px line above chapter labels, hover border on creative cards, highlight series in charts.
- Contrast: gradient text only at ≥ 28 px and weight ≥ 600.

### 3.3 Typography

- **Sora** — H1 `clamp(2.6rem, 1.6rem + 4.2vw, 5rem)`, 700, −0.035em, lh 1.03; H2 `clamp(2rem, 1.4rem + 2.4vw, 3.1rem)`, 600, −0.03em, lh 1.08; H3 22–26 px, 600. Left-aligned by default.
- **Inter** — body 18 px desktop / 16.5 px mobile, lh 1.7, max 64ch; lead 21–23 px.
- **Space Grotesk** — every number: stats, KPIs, tables, chart labels, step numbers; `tabular-nums`.
- **Chapter labels** instead of pills: `02 — Çalışma Modelimiz` (number in Space Grotesk, text = the section's verbatim `eyebrow`), Inter 13 px 600, `--muted`, Horizon Line above.

### 3.4 Grid, shape, spacing

- Container 1360 px, 12 columns, 24 px gutter. Asymmetric compositions: labels cols 1–3, content cols 4–12; running text never wider than cols 4–11.
- Radii: photos and bands 0; buttons/inputs 6 px; creative cards 14 px; panel frames 12 px; inside panels 8 px.
- Structure with hairline rules and space, not boxed cards; cards are used deliberately (creative cards, panels, drafts).
- Section padding 104 px desktop / 64 px mobile.

### 3.5 Buttons & links

- Primary: ink background, white text, 6 px radius, 52 px high, arrow that shifts 4 px; hover → `--purple`.
- Secondary: text link with animated underline and arrow.
- No pill buttons, no gradient-filled buttons.

### 3.6 Motion

- Headline lines rise + fade (500 ms, stagger 80 ms); images clip-path reveal; numbers count up once; charts draw in; carousels glide with inertia; creative cards flip/expand with 3D perspective 900 px (subtle); scroll-linked progress in processes.
- Easing `cubic-bezier(.22,1,.36,1)`. Everything respects `prefers-reduced-motion`.

### 3.7 Photography & icons

- Real editorial photos, large, full-bleed or edge-bleeding, cinematic crops (21:9 / 16:9 / 4:5), no fade masks, radius 0. Short captions allowed.
- Icons only where meaningful (UI, status, file types), lucide 1.5 stroke. No decorative icon tiles.

### 3.8 Banned "old-site DNA" (must not appear)

- Pill-shaped eyebrow labels with a coloured dot ("• NEXTERA DANIŞMANLIK").
- Service cards whose photo fades into white/dark with an icon tile overlapping the fade and a "Detaylar →" link.
- The 2 + 3 grid of featured service cards.
- Two portrait photos side by side with a floating dark stat badge.
- Dotted-grid backgrounds, radial glows/blobs, glassmorphism, tilted screenshots.
- Every section as "centred label + centred H2 + subline + card grid".
- White rounded icon tiles as decoration.
(Gradient headline words are **allowed and wanted** — see §3.2.)

---

## 4. SITEMAP & NAVIGATION

### 4.1 Routes (complete list — do not add others)

| Route | Page |
|---|---|
| `/` | Ana Sayfa (§10.1) |
| `/hizmetler/` | Hizmetler index (§10.2) |
| `/hizmetler/<slug>/` | 42 service pages (§5–§8) |
| `/basari-hikayeleri/` | Başarı Hikayeleri index (§9) |
| `/basari-hikayeleri/<slug>/` | 19 success-story detail pages (§9) |
| `/hakkimizda/` | Hakkımızda (§10.3) |
| `/iletisim/` | İletişim (§10.4) — all "Ön görüşme ayarla" buttons go here |
| `/kvkk/`, `/cerez-politikasi/` | Legal placeholders ("Hukuki metin eklenecek") |
| 404, `sitemap.xml`, `robots.txt` | Generated |

### 4.2 The 42 service routes

Hubs are the practice-area landing pages (their `related` section lists all children).

| # | Label | Path | Practice area |
|---|---|---|---|
| 01 | Hibe ve Teşvik Danışmanlığı (hub) | `/hizmetler/hibe-tesvik-danismanligi/` | hibe-tesvik |
| 02 | KOSGEB Destekleri | `/hizmetler/kosgeb-hibe-destekleri/` | hibe-tesvik |
| 03 | TÜBİTAK Destekleri | `/hizmetler/tubitak-destekleri/` | hibe-tesvik |
| 04 | Ticaret Bakanlığı Destekleri | `/hizmetler/ticaret-bakanligi-destekleri/` | hibe-tesvik |
| 05 | İhracat Teşvikleri | `/hizmetler/ihracat-tesvikleri/` | hibe-tesvik |
| 06 | Yatırım Teşvik Belgesi Danışmanlığı | `/hizmetler/yatirim-tesvik-belgesi/` | hibe-tesvik |
| 07 | Ar-Ge ve Tasarım Merkezi Destekleri | `/hizmetler/arge-tasarim-merkezi/` | hibe-tesvik |
| 08 | Avrupa Birliği (AB) ve Uluslararası Hibeler | `/hizmetler/avrupa-birligi-hibe-fonlari/` | hibe-tesvik |
| 09 | Turquality Danışmanlığı | `/hizmetler/turquality-danismanligi/` | hibe-tesvik |
| 10 | Yurtiçi ve Yurtdışı Fuar Destekleri | `/hizmetler/yurt-disi-fuar-tesvikleri/` | hibe-tesvik |
| 11 | Kırsal Kalkınma (TKDK) Destekleri | `/hizmetler/kirsal-kalkinma-tkdk-destek/` | hibe-tesvik |
| 12 | Marka, Tasarım ve Patent Desteği | `/hizmetler/marka-patent-tasarim/` | hibe-tesvik |
| 13 | SGK Teşvik Danışmanlığı | `/hizmetler/sgk-tesvik-danismanligi/` | hibe-tesvik |
| 14 | Sağlık Turizmi Teşvikleri | `/hizmetler/saglik-turizmi-tesvikleri/` | hibe-tesvik |
| 40 | Güneş Enerjisi Teşvik & GES Belge | `/hizmetler/gunes-enerjisi-yatirim-tesvikleri/` | hibe-tesvik |
| 15 | İhracat Danışmanlığı (hub) | `/hizmetler/ihracat-danismanligi/` | ihracat |
| 16 | Yurtdışı Teşvikleri | `/hizmetler/yurt-disi-tesvikleri/` | ihracat |
| 17 | Pazara Giriş Belgesi Desteği | `/hizmetler/pazara-giris-belgesi/` | ihracat |
| 18 | Yurt Dışı Pazar Araştırması | `/hizmetler/yurt-disi-pazar-arastirmasi/` | ihracat |
| 19 | Yurt Dışı Pazar Araştırması Desteği | `/hizmetler/yurt-disi-pazar-arastirmasi-destegi/` | ihracat |
| 20 | İthalat Danışmanlığı | `/hizmetler/ithalat-danismanligi/` | ihracat |
| 21 | Gümrük Danışmanlığı | `/hizmetler/gumruk-danismanligi/` | ihracat |
| 22 | Pazarlama Danışmanlığı (hub) | `/hizmetler/pazarlama-danismanligi/` | pazarlama |
| 23 | Sosyal Medya Yönetimi Danışmanlığı | `/hizmetler/sosyal-medya-danismanligi/` | pazarlama |
| 24 | SEO Danışmanlığı | `/hizmetler/seo-danismanligi/` | pazarlama |
| 25 | Marka Danışmanlığı | `/hizmetler/marka-iletisim-danismanligi/` | pazarlama |
| 26 | Google Ads ve Meta Reklamları Yönetimi | `/hizmetler/google-ads-reklam-yonetimi/` | pazarlama |
| 27 | Dijital Pazarlama Danışmanlığı | `/hizmetler/dijital-pazarlama-danismanligi/` | pazarlama |
| 28 | E-Ticaret Danışmanlığı | `/hizmetler/e-ticaret-danismanligi/` | pazarlama |
| 41 | Marka Reklam Filmi | `/hizmetler/marka-reklam-filmi/` | pazarlama |
| 29 | Yönetim Danışmanlığı (hub) | `/hizmetler/yonetim-danismanligi/` | yonetim |
| 30 | Kurumsal Eğitim Danışmanlığı | `/hizmetler/kurumsal-egitim-danismanligi/` | yonetim |
| 31 | Kurumsallaşma Danışmanlığı | `/hizmetler/kurumsallasma-danismanligi/` | yonetim |
| 32 | Aile Şirketi Danışmanlığı | `/hizmetler/aile-sirketi-danismanligi/` | yonetim |
| 33 | Aile Anayasası | `/hizmetler/aile-anayasasi/` | yonetim |
| 34 | İnsan Kaynakları Danışmanlığı (hub) | `/hizmetler/insan-kaynaklari-danismanligi/` | insan-kaynaklari |
| 35 | Outsource Bordro Yönetimi | `/hizmetler/bordro-yonetimi/` | insan-kaynaklari |
| 36 | İşe Alım Danışmanlığı | `/hizmetler/ise-alim-danismanligi/` | insan-kaynaklari |
| 37 | Çalışan Memnuniyeti Anketi | `/hizmetler/calisan-memnuniyeti-anketi/` | insan-kaynaklari |
| 38 | Yabancı Çalışma İzni | `/hizmetler/yabanci-calisma-izni/` | insan-kaynaklari |
| 39 | OKR Danışmanlığı | `/hizmetler/okr-danismanligi/` | insan-kaynaklari |
| 42 | Kurumsal Psikolojik Danışmanlık | `/hizmetler/kurumsal-psikoloji-danismanligi/` | insan-kaynaklari |

Menu labels = `seoTitle` without " | Nextera Danışmanlık". Practice-area names: "Hibe ve Teşvik", "İhracat ve Dış Ticaret", "Pazarlama", "Yönetim", "İnsan Kaynakları".

### 4.3 Header, menu, footer

- **Header:** 3 px Horizon Line on top; white bar 76 px (hairline + light blur after scroll); wordmark left (text "Nextera" Sora 700 + "DANIŞMANLIK" Inter caps until the SVG logo is supplied); nav: `Hizmetler` · `Başarı Hikayeleri` · `Hakkımızda` · `İletişim`; right: primary button "Ön görüşme ayarla".
- **Mega menu:** full-width white sheet. Left column: 5 practice areas in large Sora (active one gets the Horizon Line). Middle: that area's services as hairline text rows (hub first). Right: a featured success story card (photo, client name, one emphasised outcome from its `Çıktılar & Etki`) linking to the story. Hover-intent + click, Esc closes, full keyboard support.
- **Mobile menu:** full-screen sheet, large links, nested levels slide sideways, sticky "Ön görüşme ayarla" at the bottom.
- **Breadcrumbs** on all inner pages. **Reading-progress Horizon Line** on long pages. **"Bu sayfada" TOC**: sticky left column on desktop service pages, collapsible bar on mobile.
- **Footer (ink):** statement line (the CTA H2 of the home page), 5 columns listing all 42 services (crawlable), company column (Başarı Hikayeleri, Hakkımızda, İletişim, KVKK, Çerez Politikası), placeholder contact data from `src/config/site.ts` (address/phone/email/social = obvious TODO values), copyright.

---

## 5. CONTENT FILE — `nextera-content.json`

Place it at `src/content/nextera-content.json`. Structure: `{ services: ServiceRecord[42], successStories: Story[19] }`.

```ts
type PillarId = 'hibe-tesvik' | 'ihracat' | 'pazarlama' | 'yonetim' | 'insan-kaynaklari';

interface ServiceRecord {
  no: number; slug: string; path: string; legacyUrl: string;
  seoTitle: string; metaDescription: string;   // use exactly for <title>/<meta>
  h1: string; heroEyebrow?: string; heroLead?: string;
  pillar: PillarId; isPillarHub: boolean;
  sections: Section[];                         // original order
  faq: { q: string; a: string }[];             // 6 per page
}
interface Section {
  kind: 'content' | 'process' | 'results' | 'explore' | 'scope' | 'related' | 'cta';
  eyebrow: string | null; h2: string | null; lead: string[]; blocks: Block[];
}
type Block =
  | { type: 'paragraph'; text: string } | { type: 'bullets'; items: string[] }
  | { type: 'subheading'; text: string }
  | { type: 'card'; title: string; text: string[]; items: string[] }
  | { type: 'stat'; value: string; label: string; description: string | null }
  | { type: 'button'; label: string; href: string };

interface Story {
  no: number; slug: string; path: string;       // /basari-hikayeleri/<slug>/
  relatedServiceSlug: string;
  sourceHeading: string;                        // internal label from Word — NEVER display
  projectInfo: { Clients: string; Category: string; Date: string; Location: string; Duration: string };
  sections: { h2: string; paragraphs: string[]; bullets: string[];
              flow: { text: string; afterBullet: number }[];      // "A → B → C" lines, render as a flow chip row after that bullet
              bulletEmphasis?: Record<string, string[]> }[];      // bold fragments per bullet index (from Word) → render <strong>
  display: { title: string; clientName: string; clientDetail: string | null;
             serviceLabel: string; servicePath: string; pillar: PillarId;
             dateTR: string; dateISO: string; durationTR: string };
}
```

Section kinds: `content` = intro (eyebrow, H2, lead, 3 bullets, bold sub-heading + paragraph, 2 capability cards) · `process` = 4 steps · `results` = 2 stats · `explore` = 6 cards with bullets (absent on 22 and 41) · `scope` = lead + 3 cards with bullets · `related` = only on the 5 hubs, no blocks → render the hub's child services here · `cta` = H2, lead, 2 buttons · `faq` = 6 Q&As.

Rendering rules: one `<h1>` = `h1`; section `h2` → `<h2>`; card titles → `<h3>`; `subheading` → `<p class="lead-strong">` (or `<h3>` when it introduces cards); FAQ questions `<h3>` inside accordion triggers; eyebrows are chapter labels (`<p>`). Stat `value` strings shown exactly ("%99+", "40-60%", "250K-1M€", "6-12 ay"); count-up animates only digits and ends on the exact string. `lang="tr"` so CSS uppercase handles "İ". Write `scripts/verify-content.ts` that fails the build if any JSON string is missing from its prerendered page.

---

## 6. THE INTERACTIVE LAYER — COMPONENT FAMILIES

Every service page = **verbatim content, re-presented interactively** + **new interactive parts**. Six families are used across the site. Build them as a reusable library in `src/interactive/` with page-specific data in `src/pages-data/<slug>.ts`.

### 6.1 PANEL — realistic product screen (1 per page, the hero of the new parts)
The Bordro-style dashboard idea applied to every service: a realistic app screen for that service, rendered in a frame. Frames: **App window** (white, 12 px radius, hairline, top bar with three muted dots and a path like "Nextera Portal / Bordro / Nisan"), **Dark sidebar app** (`--deep` sidebar + white workspace), **Device pair** (laptop + phone). Contents: KPI tiles, charts, tables, maps, kanbans, calculators. Always at least two real interactions (tabs, filters, sliders, toggles, row detail drawers, drag). Mobile gets its own composition (KPI carousel first, charts full width, tables → cards). Badge "Temsili ekran", caption "Örnek verilerle hazırlanmıştır."

### 6.2 ŞEMA — interactive diagram (1 per page)
Explains the service's system/logic visually: flow diagrams, hub-and-spoke, swimlanes, matrices, Venn, org charts, maps with routes, funnels, data-flow schematics. Hover/tap a node → a side note appears. Nodes may use verbatim items from the page (scope bullets, explore bullets) or short labels. Built in inline SVG with the Horizon gradient on the active path; mobile falls back to a vertical stacked version.

### 6.3 TASLAK — a draft document preview (1 per page)
A realistic paper-like draft of a key deliverable of that service (form, report cover + first page, plan, certificate matrix, declaration, contract clause set, brief, spreadsheet). Off-white paper on a `--mist` band, fine rule lines, a diagonal watermark "TASLAK · ÖRNEKTİR", page-flip or tab navigation between 2–3 pages, hover highlights with margin annotations (short labels). Fields filled with fictional sample data. No download of real files; a "Taslağı incele" interaction only.

### 6.4 PROCESS — the page's verbatim `process` section, interactive
The 4 verbatim steps become an interactive process (variant per recipe):
- **PR1 · Stepper + detail pane** — horizontal steps; selecting one shows its verbatim description plus a small illustrative mini-visual (icon sequence, checklist, mini chart).
- **PR2 · Scroll-scrubbed timeline** — sticky timeline; the active step lights up as you scroll; a progress Horizon Line fills.
- **PR3 · Cycle** — circular 4-quadrant SVG; click a quadrant, centre shows the step; auto-advances once on first view.
- **PR4 · Swimlane** — lanes "Siz" / "Nextera" / "Kurum" (or relevant parties as short labels), the 4 steps placed on the lanes with connecting arrows.
Mobile: all become a vertical, tap-to-expand sequence.

### 6.5 CAROUSEL — (1–2 per page)
Embla-based, swipeable, keyboard-operable, visible progress bar, peeking next slide, pause on hover, no autoplay faster than 7 s (autoplay off with reduced motion). Types:
- **Success-story carousel** — slides from `successStories` related to the page (§9.4). Preferred whenever stories exist.
- **Panel screen tour** — 3–5 screens of the page's panel (e.g. "Genel bakış", "Detay", "Mobil") with captions.
- **FAQ carousel** is NOT allowed (FAQ stays an accordion for SEO).

### 6.6 KREATİF KARTLAR — creative cards for the verbatim `explore` (6) and `scope` (3) cards
Variants per recipe:
- **K1 · Flip cards** — front: card title + number; back: its verbatim bullets. Tap/Enter flips.
- **K2 · Card deck** — stacked deck; swipe/arrow to bring the next card forward; counter "3 / 6".
- **K3 · Expanding bento** — asymmetric bento; clicking a tile expands it to show bullets while others shrink.
- **K4 · Hover-reveal rail** — horizontal rail of tall cards; hover/focus reveals bullets with a gradient-border glow.
Scope (3 cards) uses **S1** columns with top Horizon Lines, **S2** capability matrix (rows = bullets, columns = the 3 cards, check marks), or **S3** fanned layered cards. All card text remains in the DOM (collapsed content via CSS, not conditional rendering) for SEO.

### 6.7 Results (verbatim `results`)
**R1** key-facts strip under the hero + statement later · **R2** stats on a deep band with gradient numbers · **R3** stats each paired with a tiny illustrative chart labelled "Temsili görselleştirme" · **R4** large photo with the two stats as overlay cards.

---

## 7. SERVICE PAGE COMPOSITION

### 7.1 Base order
1. Breadcrumb
2. Hero (variant per recipe): `heroEyebrow` as chapter label, H1 with gradient words, `heroLead`, "Ön görüşme ayarla" (→ `/iletisim/?hizmet=<slug>`) + "Panele göz atın ↓".
3. Key-facts strip (the 2 verbatim stats) — unless the recipe's results variant is R2/R4, then stats stay in the results section.
4. "Bu sayfada" TOC (desktop sticky left column).
5. Verbatim intro (`content`) as an editorial chapter: H2 left, lead + 3 bullets right, sub-heading as a pull statement, 2 capability cards as two text columns with top rules.
6. New parts + verbatim sections in the order given by the recipe's **order code**:
   - **O1:** Panel → Process → Şema → Results → Creative cards (explore) → Taslak → Scope → Carousel
   - **O2:** Process → Panel → Results → Şema → Creative cards → Scope → Taslak → Carousel
   - **O3:** Şema → Process → Creative cards → Panel → Results → Taslak → Scope → Carousel
7. Hubs only: `related` → all child services as a numbered interactive index (hover shows a mini preview of each child's panel thumbnail).
8. Verbatim `cta` — large H2 with gradient words, lead, buttons; on white or the page's deep band.
9. FAQ accordion (verbatim, first open, FAQPage JSON-LD).
10. Footer.

### 7.2 Hero variants
- **H1 · Editorial split** — text cols 1–7, photo bleeding off the right edge (4:5).
- **H2 · Panel-led** — text on top, the page's panel below it flat and large on a `--mist` band.
- **H3 · Cinematic band** — text on white, full-bleed 21:9 photo band below with Horizon Line at its bottom edge.
- **H4 · Key-facts hero** — H1 left; the two stats stacked on the right with count-up.
- **H5 · Hub index** — H1 + lead left; right the numbered list of the area's services; photo band below.
- **H6 · Map line** — text left; right a fine-line map (Türkiye / Europe / world) with gradient route strokes.
- **H7 · Report cover** — composed like a premium publication cover: big H1, meta row, thick ink rule, 3:2 photo right.
- **H8 · Device duo** — text left; laptop + phone showing the panel on a `--paper` band.
- **H9 · Pure type** — oversized H1 across 11 columns, lead offset below, generous white space.

### 7.3 Recipe table (every page differs from its neighbours)

| # | Service | Hero | Process | Cards | Results | Scope | Order |
|---|---|---|---|---|---|---|---|
| 01 | Hibe ve Teşvik Danışmanlığı | H5 | PR1 | K2 | R2 | S1 | O1 |
| 02 | KOSGEB Destekleri | H1 | PR2 | K3 | R1 | S2 | O3 |
| 03 | TÜBİTAK Destekleri | H9 | PR3 | K4 | R4 | S3 | O2 |
| 04 | Ticaret Bakanlığı Destekleri | H4 | PR4 | K1 | R3 | S1 | O1 |
| 05 | İhracat Teşvikleri | H2 | PR1 | K2 | R2 | S2 | O3 |
| 06 | Yatırım Teşvik Belgesi Danışmanlığı | H6 | PR2 | K3 | R1 | S3 | O2 |
| 07 | Ar-Ge ve Tasarım Merkezi Destekleri | H3 | PR3 | K4 | R4 | S1 | O1 |
| 08 | Avrupa Birliği (AB) ve Uluslararası Hibeler | H6 | PR4 | K1 | R3 | S2 | O3 |
| 09 | Turquality Danışmanlığı | H2 | PR1 | K2 | R2 | S3 | O2 |
| 10 | Yurtiçi ve Yurtdışı Fuar Destekleri | H3 | PR2 | K3 | R1 | S1 | O1 |
| 11 | Kırsal Kalkınma (TKDK) Destekleri | H1 | PR3 | K4 | R4 | S2 | O3 |
| 12 | Marka, Tasarım ve Patent Desteği | H7 | PR4 | K1 | R3 | S3 | O2 |
| 13 | SGK Teşvik Danışmanlığı | H4 | PR1 | K2 | R2 | S1 | O1 |
| 14 | Sağlık Turizmi Teşvikleri | H3 | PR2 | K3 | R1 | S2 | O3 |
| 15 | İhracat Danışmanlığı | H6 | PR3 | K4 | R4 | S3 | O2 |
| 16 | Yurtdışı Teşvikleri | H1 | PR4 | K1 | R3 | S1 | O1 |
| 17 | Pazara Giriş Belgesi Desteği | H7 | PR1 | K2 | R2 | S2 | O3 |
| 18 | Yurt Dışı Pazar Araştırması | H5 | PR2 | K3 | R1 | S3 | O2 |
| 19 | Yurt Dışı Pazar Araştırması Desteği | H9 | PR3 | K4 | R4 | S1 | O1 |
| 20 | İthalat Danışmanlığı | H4 | PR4 | K1 | R3 | S2 | O3 |
| 21 | Gümrük Danışmanlığı | H2 | PR1 | K2 | R2 | S3 | O2 |
| 22 | Pazarlama Danışmanlığı | H5 | PR2 | — (no explore) | R1 | S1 | O1 |
| 23 | Sosyal Medya Yönetimi Danışmanlığı | H8 | PR3 | K4 | R4 | S2 | O3 |
| 24 | SEO Danışmanlığı | H2 | PR4 | K1 | R3 | S3 | O2 |
| 25 | Marka Danışmanlığı | H9 | PR1 | K2 | R2 | S1 | O1 |
| 26 | Google Ads ve Meta Reklamları Yönetimi | H4 | PR2 | K3 | R1 | S2 | O3 |
| 27 | Dijital Pazarlama Danışmanlığı | H3 | PR3 | K4 | R4 | S3 | O2 |
| 28 | E-Ticaret Danışmanlığı | H8 | PR4 | K1 | R3 | S1 | O1 |
| 29 | Yönetim Danışmanlığı | H9 | PR1 | K2 | R2 | S2 | O3 |
| 30 | Kurumsal Eğitim Danışmanlığı | H3 | PR2 | K3 | R1 | S3 | O2 |
| 31 | Kurumsallaşma Danışmanlığı | H7 | PR3 | K4 | R4 | S1 | O1 |
| 32 | Aile Şirketi Danışmanlığı | H1 | PR4 | K1 | R3 | S2 | O3 |
| 33 | Aile Anayasası | H7 | PR1 | K2 | R2 | S3 | O2 |
| 34 | İnsan Kaynakları Danışmanlığı | H5 | PR2 | K3 | R1 | S1 | O1 |
| 35 | Outsource Bordro Yönetimi | H2 | PR3 | K4 | R4 | S2 | O3 |
| 36 | İşe Alım Danışmanlığı | H1 | PR4 | K1 | R3 | S3 | O2 |
| 37 | Çalışan Memnuniyeti Anketi | H7 | PR1 | K2 | R2 | S1 | O1 |
| 38 | Yabancı Çalışma İzni | H6 | PR2 | K3 | R1 | S2 | O3 |
| 39 | OKR Danışmanlığı | H4 | PR3 | K4 | R4 | S3 | O2 |
| 40 | Güneş Enerjisi Teşvik & GES Belge | H3 | PR4 | K1 | R3 | S1 | O1 |
| 41 | Marka Reklam Filmi | H8 | PR1 | — (no explore) | R2 | S2 | O3 |
| 42 | Kurumsal Psikolojik Danışmanlık | H9 | PR2 | K3 | R1 | S3 | O2 |

Store in `src/content/layouts.ts`.

---

## 8. PER-PAGE BRIEFS — NEW INTERACTIVE PARTS

Format: **Panel** · **Şema** · **Taslak** · **Carousel**. (Process and creative cards come from the recipe table and use the page's own verbatim content.) All labels in Turkish, all data fictional and badged.

**01 · Hibe ve Teşvik Danışmanlığı (hub)** — **Panel "Destek Pusulası":** 4-step matcher (Sektör, Ölçek, Hedef, İl) → ranked list of matching Nextera service pages with match bars and each page's own support-range stat; donut of support types. **Şema:** hub-and-spoke of the hibe-tesvik services grouped by goal (Yatırım, Ar-Ge, İhracat, İstihdam, Kırsal, Enerji). **Taslak:** "Destek Uygunluk Ön Raporu" (cover + summary page). **Carousel:** all hibe-tesvik success stories.

**02 · KOSGEB Destekleri** — **Panel "KOSGEB Başvuru Masası"** (dark sidebar): kanban Uygunluk → Hazırlık → Değerlendirmede → Onaylandı → Ödeme/Rapor, draggable cards, document checklist ring. **Şema:** application flow from idea to payment with party lanes. **Taslak:** "Proje Başvuru Formu" (işletme bilgileri, proje özeti, bütçe tablosu). **Carousel:** stories related to KOSGEB.

**03 · TÜBİTAK Destekleri** — **Panel "Ar-Ge Proje Kanvası":** TRL 1–9 slider updating a project canvas (Amaç, Yenilik, İş Paketleri, Bütçe, Riskler) and an 18-month mini Gantt. **Şema:** work-package dependency graph. **Taslak:** "Proje Önerisi" first pages (özet, yenilik unsuru, iş paketi tablosu). **Carousel:** TÜBİTAK stories.

**04 · Ticaret Bakanlığı Destekleri** — **Panel "Destek Harcama Defteri":** ledger with status pills, filters, min–max reimbursement using the page's range, reimbursement timeline chart. **Şema:** expense → document → application → payment flow. **Taslak:** "Harcama Belgesi Dosyası" index + one filled form page. **Carousel:** related story.

**05 · İhracat Teşvikleri** — **Panel "Teşvik Simülatörü":** sliders for pazar araştırması, fuar, tanıtım, belgelendirme, yurt dışı birim → stacked bars spend vs. support range (from the page's stat) and net-cost waterfall. **Şema:** incentive map linking activities to related Nextera services. **Taslak:** "Yıllık İhracat Teşvik Planı" table. **Carousel:** panel screen tour.

**06 · Yatırım Teşvik Belgesi** — **Panel "Yatırım Avantaj Hesaplayıcı":** region selector on a line map of Türkiye, investment inputs, teşvik unsurları rows (from the page's cards) with "Uygulanabilirlik" indicators and a cost-advantage band tied to the page's stat. **Şema:** incentive elements tree. **Taslak:** "Teşvik Belgesi Başvuru Özeti". **Carousel:** investment-incentive stories.

**07 · Ar-Ge ve Tasarım Merkezi** — **Panel "Merkez Hazırlık Endeksi":** semicircle gauge driven by a 10-item self-check, tabs Ar-Ge / Tasarım Merkezi, top-3 actions. **Şema:** centre organisation (personel, projeler, altyapı, raporlama). **Taslak:** "Merkez Başvuru Dosyası" contents + one page. **Carousel:** R&D-centre story.

**08 · AB ve Uluslararası Hibeler** — **Panel "Konsorsiyum Planlayıcı":** Europe line map with partner nodes, WP Gantt (36 months), budget donut per partner in €. **Şema:** consortium roles matrix. **Taslak:** proposal skeleton (Excellence / Impact / Implementation pages). **Carousel:** Horizon Europe story.

**09 · Turquality** — **Panel "Marka Olgunluk Radarı":** 8-axis radar Bugün vs Hedef with sliders, multi-year roadmap. **Şema:** brand-to-market value chain. **Taslak:** "Markalaşma Yol Haritası" one-pager. **Carousel:** panel tour.

**10 · Fuar Destekleri** — **Panel "Fuar Operasyon Takvimi":** 12-month calendar, fair selection shows stand, expense lines, lead pipeline (Görüşme → Numune → Teklif → Sipariş). **Şema:** before–during–after fair flow. **Taslak:** "Fuar Katılım Planı". **Carousel:** Hibe ve Teşvik area stories.

**11 · Kırsal Kalkınma (TKDK)** — **Panel "Kırsal Yatırım Planlayıcı":** investment type selector → simple SVG site plan highlighting eligible components, budget table "Hibe kapsamında / Kapsam dışı", grant band from the page's stat. **Şema:** IPARD application path with checkpoints. **Taslak:** "İş Planı" pages (tesis, makine listesi, bütçe). **Carousel:** all five TKDK stories.

**12 · Marka, Tasarım ve Patent** — **Panel "Fikri Mülkiyet Portföyü":** register table (Marka/Patent/Faydalı Model/Tasarım), jurisdictions on a world line map, upcoming renewals. **Şema:** protection-type decision tree. **Taslak:** "Marka Başvuru Dosyası" page. **Carousel:** panel tour.

**13 · SGK Teşvik** — **Panel "Teşvik Tarama":** animated scan over an anonymised personnel list → flagged rows with generic incentive categories, monthly savings chart, retroactive-potential KPI using the page's stats. **Şema:** personnel data → eligibility check → bildirge → saving flow. **Taslak:** "Teşvik Tarama Raporu". **Carousel:** panel tour.

**14 · Sağlık Turizmi** — **Panel "Uluslararası Hasta Yolculuğu":** funnel Talep → Tedavi → Takip, world arcs to Türkiye, supported activities panel (from the page's cards). **Şema:** patient journey service blueprint. **Taslak:** "Tanıtım Faaliyet Planı". **Carousel:** panel tour.

**15 · İhracat Danışmanlığı (hub)** — **Panel "Export Control Tower"** (dark sidebar, flagship): KPIs (Aktif Sevkiyat, Toplam Ağırlık kg/ton, TEU, Zamanında Teslim, Aylık İhracat USD), world map with animated routes, transport-mode filter (Deniz/Kara/Hava/Demiryolu), shipments table (Ürün, GTİP sample, Kg, Ülke, Mod, Incoterm, ETA, Durum), product-by-kg bar chart, market donut, document status list, shipment timeline drawer. **Şema:** export value chain (Pazar → Alıcı → Teklif → Sipariş → Lojistik → Tahsilat). **Taslak:** "İhracat Yol Haritası" + proforma draft. **Carousel:** all export stories.

**16 · Yurtdışı Teşvikleri** — **Panel "Yurt Dışı Birim Paneli":** overseas unit cards (Ofis/Depo/Mağaza), cost vs support range, FX toggle. **Şema:** unit types vs supported expenses matrix. **Taslak:** "Birim Gider Raporu". **Carousel:** export stories.

**17 · Pazara Giriş Belgesi** — **Panel "Sertifika Matrisi":** product × market grid with document tags and statuses; cell detail with steps and reimbursement band. **Şema:** certification path (test → belge → başvuru → ödeme). **Taslak:** "Belge Gereklilik Matrisi" printout. **Carousel:** export stories.

**18 · Yurt Dışı Pazar Araştırması** — **Panel "Pazar Çekicilik Matrisi":** bubble chart of candidate countries with illustrative index scores ("temsili skor"), weight sliders, compare two countries. **Şema:** research method funnel. **Taslak:** "Pazar Araştırma Raporu" cover + contents. **Carousel:** export stories.

**19 · Pazar Araştırması Desteği** — **Panel "Pazar Gezisi Planlayıcı":** itinerary days, activity and expense lines, support-range gauge. **Şema:** trip → rapor → başvuru → ödeme. **Taslak:** "Seyahat Raporu" draft. **Carousel:** export stories.

**20 · İthalat Danışmanlığı** — **Panel "Landed Cost Hesaplayıcı":** FOB, navlun, sigorta, vergi oranları (user-entered) → waterfall and supplier comparison. **Şema:** import flow from supplier to warehouse. **Taslak:** "Tedarikçi Karşılaştırma Tablosu". **Carousel:** panel tour.

**21 · Gümrük Danışmanlığı** — **Panel "Beyanname Akışı"** (dark sidebar): declaration pipeline, line distribution donut ("temsili"), sample GTİP search, alert feed. **Şema:** customs clearance swimlane. **Taslak:** declaration draft page with annotated fields. **Carousel:** panel tour.

**22 · Pazarlama Danışmanlığı (hub)** — **Panel "Büyüme Hunisi Stüdyosu":** funnel stages with channel chips linking to child pages, budget split sliders (100 % constraint), KPI tiles referencing the page's stats. **Şema:** marketing ecosystem map of the child services. **Taslak:** "Pazarlama Yol Haritası" one-pager. **Carousel:** panel tour.

**23 · Sosyal Medya** — **Panel "İçerik Takvimi"** (device duo): month grid with post cards, format filter, best-time heatmap, phone preview (no platform logos). **Şema:** content production pipeline. **Taslak:** "Aylık İçerik Planı". **Carousel:** post-preview carousel inside the phone.

**24 · SEO Danışmanlığı** — **Panel "SEO Denetim Raporu":** LCP/INP/CLS gauges, health score, keyword table with position changes, traffic area chart, issue list; tabs Teknik / İçerik / Off-page. **Şema:** site architecture tree with internal links. **Taslak:** "SEO Denetim Raporu" printout. **Carousel:** panel tour.

**25 · Marka Danışmanlığı** — **Panel "Konumlandırma Haritası":** draggable 2×2 perceptual map, archetype chips, tone-of-voice sliders with live sample headline. **Şema:** brand architecture (değer → kimlik → iletişim). **Taslak:** "Marka Kitabı" first spreads. **Carousel:** mood carousel of brand touchpoint mock-ups (fictional brand).

**26 · Google Ads & Meta** — **Panel "Kampanya Konsolu"** (dark sidebar): KPI row, campaign table, pacing chart, budget re-allocation simulator. **Şema:** targeting → creative → landing → conversion loop. **Taslak:** "Kampanya Kurulum Planı". **Carousel:** ad-creative variants carousel (fictional).

**27 · Dijital Pazarlama** — **Panel "360° Kanal Orkestrası":** radial channel map, Sankey-style flow Kanal → Ziyaret → Lead → Satış, attribution model toggle. **Şema:** the 360° model itself. **Taslak:** "Dijital Strateji Planı". **Carousel:** panel tour.

**28 · E-Ticaret** — **Panel "Pazaryeri Komuta Merkezi"** (device duo): channel tabs as text labels (Amazon, Etsy, Hepsiburada, Trendyol — no logos) + "Kendi site", sales, orders, returns, listing health, stock alerts. **Şema:** order-to-delivery operations flow. **Taslak:** "Ürün Listeleme Şablonu". **Carousel:** product listing preview carousel (fictional products).

**29 · Yönetim Danışmanlığı (hub)** — **Panel "Strateji Haritası":** 4 perspectives with objective bubbles, causal arrows, KPI status lights, Bugün / 12 ay toggle. **Şema:** management-system house (vizyon → strateji → süreç → performans). **Taslak:** "Stratejik Plan" contents. **Carousel:** panel tour.

**30 · Kurumsal Eğitim** — **Panel "Yetkinlik Matrisi":** department × competency heatmap, learning path per cell, 4-level impact bar. **Şema:** training cycle. **Taslak:** "Yıllık Eğitim Planı". **Carousel:** program cards carousel (fictional modules).

**31 · Kurumsallaşma** — **Panel "Organizasyon Tasarımcısı":** collapsible org chart + editable RACI matrix + process maturity bars. **Şema:** governance structure. **Taslak:** "Görev Tanımı Formu". **Carousel:** panel tour.

**32 · Aile Şirketi** — **Panel "Nesil Geçiş Yol Haritası":** interactive three-circle model (Aile/Sahiplik/Yönetim — from the page's content), fictional genogram G1–G3, succession timeline. **Şema:** family governance bodies. **Taslak:** "Halefiyet Planı". **Carousel:** panel tour.

**33 · Aile Anayasası** — **Panel "Anayasa Stüdyosu":** living document with articles from the page's "Anayasa Bileşenleri" items; option toggles change sample clauses; council diagram. **Şema:** aile konseyi ↔ yönetim kurulu ↔ şirket. **Taslak:** the constitution draft itself with signature block (placeholder names, "örnek madde metni, hukuki tavsiye değildir"). **Carousel:** article carousel of the document pages.

**34 · İnsan Kaynakları (hub)** — **Panel "İK Olgunluk Testi":** 10 Likert questions → maturity level stepper, spider chart, recommended child services. **Şema:** employee lifecycle wheel (işe alım → onboarding → performans → gelişim → ayrılış) linked to child services. **Taslak:** "İK Politika Seti" contents. **Carousel:** HR success story.

**35 · Outsource Bordro Yönetimi** — **Panel "Bordro360"** (dark sidebar, the model page): menu (Ana Sayfa, Bordro, Özlük Bilgileri, İzin Yönetimi, Masraf Yönetimi, Performans, Raporlar, Ayarlar), greeting header, KPIs (Net Ücret highlighted in Nextera purple, Çalışan Sayısı, Devam Oranı, Açık Talep), Brütten Nete bar chart, Kesinti donut, Yıllık Net Ücret line, Kalan Yıllık İzin card, Son Bordrolar table (download buttons disabled with "Demo" tooltip); toggle Çalışan / İşveren view (company cost, bildirim calendar chips for Muhtasar / e-Bildirge, department cost split); Brüt–Net calculator tab with parameters in a data file and note "Parametreler yıllık mevzuata göre güncellenir; temsili hesaplamadır". **Şema:** payroll data flow (Puantaj → Bordro → SGK/Muhtasar → Banka talimatı → Raporlama). **Taslak:** "Ücret Pusulası" (fictional employee). **Carousel:** panel tour (Çalışan, İşveren, Mobil).

**36 · İşe Alım** — **Panel "Aday Pipeline":** ATS kanban, candidate scorecards, KPIs referencing the page's stats. **Şema:** competency-based selection funnel. **Taslak:** "Pozisyon Tanımı & Değerlendirme Formu". **Carousel:** İşe alım success story.

**37 · Çalışan Memnuniyeti Anketi** — **Panel "Nabız Raporu":** participation, eNPS gauge, department × dimension heatmap, Likert distributions, trend vs previous survey, chip cloud from open answers, action plan table; note "5 kişiden az yanıtlı kırılımlar gösterilmez". **Şema:** survey cycle (tasarım → uygulama → analiz → aksiyon). **Taslak:** survey questionnaire draft. **Carousel:** report pages carousel.

**38 · Yabancı Çalışma İzni** — **Panel "İzin Takip Merkezi":** case list with stage stepper, expiry chips 90/60/30, document checklist, nationality map. **Şema:** permit application swimlane. **Taslak:** document checklist form. **Carousel:** panel tour.

**39 · OKR Danışmanlığı** — **Panel "OKR Ağacı":** company → team → individual objectives, KR sliders recalculating parents, quarter selector, check-in timeline. **Şema:** OKR cycle. **Taslak:** "OKR Çalışma Sayfası". **Carousel:** example OKR cards carousel (fictional).

**40 · Güneş Enerjisi (GES)** — **Panel "GES Getiri Simülatörü":** province (illustrative irradiation index, "temsili"), area, kWp, consumption, incentive toggle → yearly production, cumulative cash-flow with break-even band tied to the page's "4-6 yıl", cost-advantage band from "%25-40". **Şema:** GES permit & incentive path. **Taslak:** "GES Fizibilite Özeti". **Carousel:** panel tour.

**41 · Marka Reklam Filmi** — **Panel "Storyboard Stüdyosu"** (device duo): 6-frame storyboard with scene notes, aspect-ratio switcher (16:9 / 9:16 / 1:1), production timeline within the page's "3-7 gün". **Şema:** brief → script → AI production → revision → delivery. **Taslak:** "Film Brifi" form. **Carousel:** storyboard frame carousel.

**42 · Kurumsal Psikolojik Danışmanlık** — **Panel "İyi Oluş Nabzı":** aggregate-only wellbeing index, burnout-risk bands at company level, programme participation, session calendar; privacy badge "Bireysel veri gösterilmez · En az 5 kişilik kırılım". Calm, soft palette. **Şema:** support model (bireysel, grup, yönetici). **Taslak:** "Program Planı". **Carousel:** panel tour.

---

## 9. BAŞARI HİKAYELERİ (the only new section)

Source: `successStories` (19 items, verbatim from the owner's Word document).

### 9.1 Rendering rules
- **Never display `sourceHeading`.** Story title on cards and H1 = `display.title` (client name — service label).
- Field labels in Turkish: Müşteri (`projectInfo.Clients`), Hizmet (`display.serviceLabel`, linked), Kategori (`projectInfo.Category`, shown as-is), Tarih (`display.dateTR`), Lokasyon (`projectInfo.Location`), Süre (`display.durationTR`).
- Sections in order with their verbatim H2s; bullets verbatim; `bulletEmphasis` fragments wrapped in `<strong>` (these are the highlight numbers); `flow` lines rendered as a chip row with arrows right after bullet `afterBullet`.
- "Çıktılar & Etki" is the star: big outcome cards; the first emphasised fragment of each bullet becomes the headline number of the card (Space Grotesk, gradient).

### 9.2 `/basari-hikayeleri/` index
H1 "Başarı Hikayeleri" with gradient on "Hikayeleri". Filter chips by practice area and by service; sort by date (newest first). Layout: a featured story at the top (large photo, title, 3 outcome numbers), then a grid/list of all stories with photo, client, service, date, and the first outcome. ItemList JSON-LD.

### 9.3 `/basari-hikayeleri/<slug>/` detail
Hero (report-cover style, H1 = `display.title`), project-info panel (sticky on desktop), chapters from the sections, outcome cards, a small "Proje akışı" visual built from the section order (Genel Bakış → Başlangıç → Araştırma → Uygulama → Çıktılar), link to the related service, "Diğer başarı hikayeleri" carousel, CTA. JSON-LD: `Article` (headline = title, datePublished = dateISO, author/publisher = Nextera Danışmanlık) + `BreadcrumbList`.

### 9.4 Carousels
- **Home page — "Başarı Hikayeleri" carousel (showpiece):** full-width Embla carousel. Each slide = split composition: left a large sector photo (clip-path reveal on slide change), right client name, service label (link), date · location, and **three outcome numbers** from `Çıktılar & Etki` counting up on slide enter, plus "Hikayenin tamamı →". Controls: numbered progress bar (01 / 19) with the Horizon gradient filling per slide, prev/next buttons, swipe, keyboard, filter chips above (Tümü, Hibe ve Teşvik, İhracat, İnsan Kaynakları). Autoplay 7 s, pauses on hover/focus, off with reduced motion. Mobile: photo on top, numbers as a 3-column row, full swipe.
- **Service pages:** carousel of stories whose `relatedServiceSlug` = page slug; hubs show all stories of their practice area; all other pages use exactly the carousel named in their brief in §8 (when a brief says "… stories" for a page without direct stories, title the carousel "{Practice area} alanındaki başarı hikayeleri").
- Story → service mapping: İşe Alım (1), İhracat Danışmanlığı (4), Ticaret Bakanlığı (1), Yatırım Teşvik Belgesi (2), KOSGEB (2), Ar-Ge Merkezi (1), AB Hibeleri (1), TÜBİTAK (2), TKDK (5).

### 9.5 Photos for stories
The Word file has no usable images. Use real photos matching each story's sector (software office, furniture/industrial export, machinery, food processing, textile, chemicals, automation, IoT lab, dairy, poultry farm, olive oil, feed mill…). No photos implying they show the actual client.

---

## 10. OTHER PAGES (verbatim texts, new interface)

### 10.1 Ana Sayfa
Texts come from the current home page (verbatim):
1. **Hero:** label "NEXTERA DANIŞMANLIK"; H1 "Şirketinizi geleceğe taşıyan stratejik danışmanlık" (gradient on "geleceğe taşıyan"); subline "Finansal, dijital ve yönetimsel çözümlerle sürdürülebilir büyümenizi destekliyoruz."; buttons "Ön görüşme ayarla", "Hizmetleri keşfet". New composition: pure-type hero with a cinematic full-bleed photo band below it, and a slim interactive strip showing three live mini-panels (Export Control Tower, Bordro360, Nabız Raporu) that the user can switch between.
2. **Practice areas:** label "ÖNE ÇIKAN HİZMETLER"; H2 "İşletmenizin başarısı için stratejik çözümler" (gradient on "stratejik çözümler"); subline "Finans, dijital, yönetim ve teşvik alanlarında bütünleşik hizmet sunuyoruz." New interface: an interactive accordion-panel — five tall vertical panels side by side (one per practice area), the hovered/selected one expands to show a photo, the hub's `metaDescription` (verbatim) and its service list; mobile = stacked accordion.
3. **Başarı Hikayeleri carousel** (§9.4).
4. **About:** label "HAKKIMIZDA"; H2 "Birlikte büyüyen işletmeler için buradayız" (gradient on "büyüyen işletmeler"); paragraph "Nextera olarak, iş dünyasının değişen dinamiklerine uyum sağlayan; hibe, teşvik, yönetim, dijital ve ihracat danışmanlık hizmetleriyle markaların sürdürülebilir başarısını inşa ediyoruz."; items "Strateji, deneyim ve güvenle" — "Karmaşık süreçleri sadeleştiriyor, büyümenizi sistematik bir modele dönüştürüyoruz." and "Her ölçekte işletme için esnek çözümler" — "Start-up'tan global markalara kadar her ölçekte işletmeye özel hizmet modelleri tasarlıyoruz."; figure "12+" — "Yıllık Sektör Deneyimi". New interface: editorial split with one cinematic photo and a large key-facts line; no two-photo + badge collage.
5. **CTA:** "Geleceğinizi şekillendiren danışmanlıkla tanışın" (gradient on "danışmanlıkla") + "Hibe, teşvik, ihracat ve dijital alanlarda stratejik çözümlerle işletmenizi bir adım öteye taşıyoruz." + buttons.
JSON-LD: Organization, WebSite, ProfessionalService.

### 10.2 Hizmetler index
H1 "Hizmetlerimiz" (gradient on "Hizmetlerimiz"). Search (Turkish-aware), practice-area tabs, all 42 services as hairline rows (label + meta description + arrow) grouped by area. ItemList JSON-LD.

### 10.3 Hakkımızda
Use the verbatim about texts above in a new editorial layout. If more about text is needed, leave a clearly marked placeholder block "Hakkımızda metni eklenecek" — do not write new copy.

### 10.4 İletişim
H1 "İletişim". Form: Ad Soyad, Şirket, E-posta, Telefon, İlgilendiğiniz hizmet (preselected from `?hizmet=`), Mesaj, KVKK onayı. Contact details from `site.ts` placeholders. ContactPage JSON-LD.

---

## 11. TECHNICAL SEO

- Exact legacy paths with trailing slash; non-slash requests redirect/canonicalise. `SITE_URL = https://nexteradanismanlik.com` in `site.ts`.
- `seoTitle` / `metaDescription` verbatim on services. Other titles: "Başarı Hikayeleri | Nextera Danışmanlık", "{display.title} | Nextera Danışmanlık", "Hizmetlerimiz | Nextera Danışmanlık", "Hakkımızda | Nextera Danışmanlık", "İletişim | Nextera Danışmanlık". Story meta description = the first sentence of its "Genel Bakış" paragraph (verbatim, trimmed at a word boundary to ≤ 160 chars).
- Canonical, `lang="tr"`, `og:locale=tr_TR`, OG image per practice area (white, ink type, gradient words).
- JSON-LD `@graph` per service page: Organization (by @id), WebPage, BreadcrumbList, Service, FAQPage (verbatim), HowTo from the process steps. No fake ratings/reviews.
- Heading outline as in §5; panel/draft UI text uses `<p>/<span>`, never headings.
- All verbatim text and collapsed content present in prerendered HTML (use `hidden="until-found"` / CSS, not conditional rendering).
- Internal links: breadcrumbs, mega menu, footer (all 42), hub ↔ children, story ↔ service, Destek Pusulası results → services. Every page within 2 clicks of home.
- `sitemap.xml` (all public routes + lastmod), `robots.txt`.
- Targets (mobile): LCP < 2.0 s, INP < 200 ms, CLS < 0.05; Lighthouse Performance ≥ 90, SEO 100, Accessibility ≥ 95.

---

## 12. IMAGES — REAL, WEBP, COMPRESSED

- Real photos from Unsplash/Pexels (free licence); download originals into `src/assets/photos/…` and convert at build; record credits in `photo-credits.ts`. If downloads are impossible, use the Unsplash CDN with `?auto=format&fm=webp&q=72&w=…` in `srcset`.
- AVIF + WebP; widths 480/768/1080/1440/1920; WebP q 70–76. Budgets: hero ≤ 180 KB desktop / ≤ 110 KB mobile; content images ≤ 70 KB; thumbnails ≤ 25 KB.
- Explicit width/height, blur-up placeholders, lazy below the fold, hero `fetchpriority="high"` + preload.
- Turkish, descriptive `alt` text. No AI images, no watermarks, no visible brand logos.
- Art direction: Hibe/Teşvik → factories, labs, machinery, farms, solar; İhracat → ports, ships, trucks, air cargo, warehouses, customs; Pazarlama → studios, content production, analytics, e-commerce; Yönetim → boardrooms, workshops, family businesses; İK → diverse teams, interviews, training, calm spaces. 3–5 photos per service page, none reused across service pages.

---

## 13. MOBILE-FIRST & ACCESSIBILITY

Design at 375 px first; test 360/390/414/768/1024/1280/1440/1920. Tap targets ≥ 44 px. Sticky mobile bottom bar "Ön görüşme ayarla" on service pages (hides on scroll down). Panels, schemas, drafts and carousels each have a designed mobile version. Keyboard support for menus, tabs, accordions, flip cards, carousels, sliders, drag (with button alternatives). Focus ring Nextera purple. Charts have hidden data tables or text summaries. Contrast AA.

## 14. PERFORMANCE

Initial JS per service page ≤ 170 KB gzip (excluding lazy interactive chunks, each ≤ 90 KB). CSS ≤ 40 KB. Maps are lightweight inline SVG. Animate only transform/opacity.

## 15. FILES & GOVERNANCE

`src/content/nextera-content.json` (verbatim), `layouts.ts`, `highlights.ts`, `pages-data/<slug>.ts` (sample data + microcopy for the new parts), `site.ts` (placeholders), `photo-credits.ts`, `scripts/verify-content.ts`, `/docs/url-map.md` (legacyUrl → route, should be identical).

---

## 16. BUILD PHASES (stop after each)

1. **Foundations:** tokens, fonts, GradientText, Horizon Line, background bands, header/mega menu/mobile menu/footer, all routes with placeholders, prerendering, and a style-guide route `/_stil/` (noindex) showing every hero variant, PR1–PR4, K1–K4, S1–S3, R1–R4, panel frames, draft paper, schema styles and the carousel. **Stop — I approve the look first.**
2. **Service page engine:** content loader, composition by recipe and order code, verbatim sections, process and creative-card components wired to verbatim content, results, FAQ, CTA, related, JSON-LD. All 42 pages render verbatim text; new parts shown as named placeholders. Run `verify-content.ts`.
3. **Başarı Hikayeleri:** index, 19 detail pages, home carousel, service-page carousels.
4. **Flagship interactive parts:** 35 Bordro360, 15 Export Control Tower, 37 Nabız Raporu, 01 Destek Pusulası, 40 GES Simülatörü — each with its Şema and Taslak.
5. **Remaining pages' Panel + Şema + Taslak + Carousel,** one practice area per step (Hibe ve Teşvik → İhracat → Pazarlama → Yönetim → İK).
6. **Photos** everywhere + OG images.
7. **Home, Hizmetler, Hakkımızda, İletişim, legal, 404.**
8. **QA:** verbatim check, old-site resemblance check (§17), mobile pass, accessibility, Lighthouse, sitemap. Final report per page: recipe, panel name, schema, draft, carousel type.

## 17. ACCEPTANCE

- [ ] All 42 service pages at their exact paths, every JSON string present (script passes).
- [ ] Each service page has: panel, şema, taslak, interactive process, creative cards (where explore exists), carousel — all original to that page, mobile-designed, badged as sample.
- [ ] No new pages besides `/basari-hikayeleri/` (+ its 19 details); no new prose sections anywhere.
- [ ] Gradient words used in most H1/H2s; white dominates; tonal transitions soft.
- [ ] None of the banned old-site patterns in §3.8.
- [ ] Home success-stories carousel works with swipe, keyboard, filters, progress, count-up.
- [ ] Prerendered HTML complete; JSON-LD valid; images WebP/AVIF within budget; Lighthouse targets met.

Before coding, reply with (1) your understanding in 8–10 bullets, (2) folder structure, (3) any environment limits (prerendering, image processing, downloads) and workarounds. Then build Phase 1.
