# GUNCELLEMELER1.md — Nextera site update log (part 2)

> **How to use this file (for Lovable):** When the owner writes "GUNCELLEMELER1.md içindeki U-1XX'i uygula", implement **only that block**. Do not re-apply the master prompt or other update files, do not touch files outside the block's "Files" list, do not refactor or restyle other parts of the site. Reuse existing components, tokens and utilities. When done, reply with max 5 lines (files changed + anything needing the owner's input) and mark the block `✅ Uygulandı (date)`.
>
> **Relationship to GUNCELLEMELER.md:** U-104 **replaces U-004 and U-005** of `GUNCELLEMELER.md`. If U-004/U-005 were already applied, migrate their table/components into U-104 instead of creating duplicates; if not applied, skip them and mark them `⛔ U-104 ile değiştirildi`.

## Credit-saving rules (apply to every block)

1. Single pass; ask nothing unless something is impossible.
2. Touch only listed files; new files exactly where specified.
3. No global refactors, no dependency upgrades; new libraries only if the block allows them.
4. Do not regenerate unrelated pages, content JSON or tests.
5. Short replies, no alternative proposals.
6. Missing values → clearly named placeholders, never invented.

**Recommended order:** U-101 → U-102 → U-103 → U-104.

---

## U-101 · Tüm carousel ve kaydırmalı alanlar: masaüstü + mobil kullanılabilirlik denetimi

**Status:** ⏳ Bekliyor
**Scope:** Every horizontally scrolling / sliding UI on the site. Behaviour and controls only — no content changes, no new sections.
**Allowed new dependency:** none (Embla already in use).

### Files
- `src/components/ui/Carousel.tsx` — **new shared wrapper** around Embla (or refactor the existing one if present)
- Every component that currently scrolls horizontally — switch to the shared wrapper
- `docs/carousel-audit.md` — **new** report

### Inventory first
List every horizontal scroller in `docs/carousel-audit.md` (at least: home practice-area carousel U-009, home success-stories carousel, YouTube carousel U-001, home blog row on mobile, service-page success-story / panel-tour carousels, story-detail "Diğer başarı hikayeleri", K2 card deck, K4 hover-reveal rail, mobile KPI carousels inside panels, Teşvik Robotu map strip / compare cards, any `overflow-x:auto` table wrapper). For each: page, component, device issues found, fix applied.

### One standard for all carousels
- **Controls always visible on desktop:** prev/next buttons (40–44px, clear arrow icons) placed consistently (top-right of the section header, or bottom-centre for full-width hero carousels). Never hover-only. Disabled (40 % opacity, not clickable) at start/end when not looping.
- **Position indicator:** "3 / 8" counter or dots (max 8 dots; above that use counter + Horizon Line progress bar). Current state announced to screen readers.
- **Mobile:** swipe with snap; cards 85–88 % width so the **next card visibly peeks**; no arrows required but allowed; indicator always visible below the track. First time a carousel enters view on a touch device, a subtle one-time nudge (track shifts 24px and back, 600 ms) — skip with reduced motion.
- **Gestures:** `touch-action: pan-y` on the track so vertical page scrolling is never blocked; drag threshold ≥ 8px so taps on links/buttons inside slides still work; no scroll-jacking; mouse drag enabled on desktop with `cursor: grab`.
- **Keyboard:** Left/Right arrows when the carousel has focus; Tab reaches every link inside visible slides; off-screen slides are `inert`/`aria-hidden` appropriately.
- **Autoplay:** only for the home practice-area carousel (U-009) and the home success-stories carousel; interval ≥ 6 s; pauses on hover, focus and touch; **stops permanently after the first manual interaction**; off with `prefers-reduced-motion`.
- **Hover-only features must work on touch:** K4 hover-reveal and similar become tap-to-reveal on touch devices; expanding cards (U-009) expand on tap, second tap opens the link.
- **Sizing:** equal slide heights per carousel (no jumping), images with fixed aspect ratio (no CLS), minimum tap target 44×44.
- **Nested scrolling:** no horizontal carousel inside another horizontal scroller; tables inside panels scroll inside their own container with a right-edge fade hint and a "kaydırın →" micro-label on mobile.
- **Loop:** off by default (clear start/end); on only where content is decorative.
- **Accessibility:** wrapper `role="region"` + `aria-roledescription="carousel"` + `aria-label`; slides `role="group"` + `aria-label="3 / 8"`; buttons with Turkish labels ("Önceki", "Sonraki").

### Test matrix (report results in the audit file)
360×800, 390×844, 768×1024, 1280×800, 1440×900 · Chrome, Safari iOS, Firefox · touch + mouse + keyboard.

### Acceptance
- [ ] All carousels use the shared wrapper and the standard above.
- [ ] On mobile every carousel shows a peeking next card and an indicator; vertical page scroll never gets stuck.
- [ ] On desktop every carousel has visible arrows and a position indicator.
- [ ] Audit file lists every scroller and its fix.

---

## U-102 · Yönetim paneli altyapısı + Formlar sayfası

**Status:** ⏳ Bekliyor
**Scope:** Admin area at `/yonetim/` and the contact-form backend.
**Allowed new dependency:** none.

### Files
- `src/admin/**` — **new** (layout, auth guard, pages)
- Supabase migration(s): `admins`, `contact_requests` changes
- `src/pages/Contact.tsx` (only the submit handler, to store the new fields)
- `robots.txt` / sitemap generator: exclude `/yonetim/`

### Auth & access
- Route `/yonetim/` (and all children): `noindex, nofollow`, excluded from sitemap and prerendering.
- Supabase email + password login (magic link optional). Access only for users present in table `admins(user_id uuid pk references auth.users, name text, role text default 'editor', created_at)`. First admin: the owner adds their user id via Supabase dashboard — document this in `docs/admin.md`.
- Roles: `owner` (everything), `editor` (Blog + Formlar). Enforced with RLS, not only in UI.
- Session timeout 12 h; "Çıkış yap" in the header.

### Admin layout
- Left sidebar (Derin Mor `#270F33`, white text): **Genel Bakış · Formlar · Blog · SEO** (Blog = U-104, SEO = U-103; show them disabled with "Yakında" until those blocks are applied). Top bar: page title, user name, logout. White workspace, Inter, compact tables; fully usable on mobile (sidebar becomes a bottom sheet menu).
- **Genel Bakış:** 4 KPI tiles — Yeni form (bugün / 7 gün), Yanıtlanmayan form, Yayındaki yazı, Taslak yazı — and the 5 latest form submissions.

### Formlar (form inbox)
- Table `contact_requests` (extend if it exists): `id`, `created_at`, `name`, `company`, `email`, `phone`, `service_slug`, `message`, `source` (`iletisim` | `tesvik-robotu` | other), `status` (**`yeni` | `yanitlanmadi` | `inceleniyor` | `yanitlandi` | `arsiv`**, default `yeni`), `assigned_to uuid null`, `internal_note text`, `answered_at timestamptz`, `updated_at`, `kvkk_consent boolean`, `consent_at timestamptz`.
  - When an admin opens a `yeni` request it automatically becomes `yanitlanmadi` (seen but not answered). Setting `yanitlandi` stamps `answered_at`.
- RLS: anonymous users can **insert only** (no select); admins can select/update; only `owner` can delete.
- List view: columns Tarih · Ad Soyad · Şirket · Hizmet · Kaynak · Durum · Atanan; status as coloured pills (Yeni = purple, Yanıtlanmadı = amber, İnceleniyor = lilac, Yanıtlandı = green, Arşiv = grey); unread rows bold; filters by status / service / source / date range; search (name, company, email); sort by date; pagination 25; counters per status as tabs ("Yeni 4 · Yanıtlanmadı 7 · Yanıtlandı 23 …").
- Detail drawer: all fields, message, Teşvik Robotu summary nicely formatted when `source='tesvik-robotu'`, status dropdown, assign to admin, internal note, buttons "E-posta ile yanıtla" (`mailto:` with subject "Nextera Danışmanlık – {hizmet}") and "Ara" (`tel:`), history line (created / answered).
- Bulk actions: mark as Yanıtlandı / Arşiv; **CSV export** of the current filter (UTF-8 with BOM for Excel, Turkish headers).
- Sidebar badge with the count of `yeni` + `yanitlanmadi`.
- KVKK: show consent timestamp; `owner` can permanently delete a request (confirmation dialog).

### Acceptance
- [ ] Only admins can open `/yonetim/`; RLS verified (anon cannot read requests).
- [ ] A new contact-form / Teşvik Robotu submission appears as "Yeni", becomes "Yanıtlanmadı" when opened, "Yanıtlandı" with timestamp when marked.
- [ ] Filters, search, bulk actions and CSV export work; mobile usable.

---

## U-103 · Yönetim paneli: SEO sekmesi

**Status:** ⏳ Bekliyor (U-102 sonrası)
**Scope:** New admin page `/yonetim/seo/` — read-only overview.
**Allowed new dependency:** none.

### Files
- `src/admin/pages/Seo.tsx` — **new**
- `src/admin/seo/collect.ts` — **new** (collects data from existing sources; no duplicated content)

### What it shows
Tabs:
1. **Hizmet Sayfaları (42)** — from `nextera-content.json`: No · Hizmet (menu label) · URL (link, opens in new tab) · **SEO Başlığı** · karakter sayısı · **Meta Açıklaması** · karakter sayısı · H1 · canonical.
2. **Diğer Sayfalar** — home, `/hizmetler/`, `/hakkimizda/`, `/basari-hikayeleri/` + 19 stories, `/yatirim-tesvik-robotu/`, `/iletisim/`, `/blog/` (values from the same `<Seo>` configuration the pages use).
3. **Blog Yazıları** — after U-104: title, SEO title (or fallback), meta (or fallback), status, canonical.

Features:
- Length indicators: SEO title ≤ 60 green, 61–70 amber, > 70 red; meta description 120–160 green, < 120 or 161–170 amber, > 170 red. Missing value = red "Eksik".
- Duplicate detection: highlight rows whose SEO title or meta description is identical to another page.
- Search + filter (practice area, status colour), sort by any column.
- Row click → side panel with a **Google result preview** (desktop and mobile width), the full texts with copy buttons.
- **CSV export** of the current tab.
- Read-only for service pages (their texts are verbatim content). A note at the top: "Hizmet sayfası SEO metinleri içerik dosyasından okunur; değişiklik için içerik güncellemesi yapılmalıdır."

### Acceptance
- [ ] All 42 service pages listed with correct SEO title and meta description and character counts.
- [ ] Colour indicators, duplicate warning, preview and CSV export work.

---

## U-104 · Blog yönetim sistemi (admin + /blog + ana sayfa + detay)

**Status:** ⏳ Bekliyor (U-102 sonrası)
**Scope:** Complete blog system. Replaces U-004 and U-005 of `GUNCELLEMELER.md`.
**Allowed new dependency:** **Tiptap** (`@tiptap/react`, `@tiptap/starter-kit`, `@tiptap/extension-link`, `@tiptap/extension-image`, `@tiptap/extension-placeholder`) for the rich-text editor, and **DOMPurify** for sanitising pasted/rendered HTML. Load them **only in the admin chunk**, never on public pages.

### A) Owner's requirements (verbatim — implement exactly as written)

```text
Please build a complete Blog Management System for the website. 1. ADMIN BLOG PANEL - Add a dedicated “Blog” section inside the admin panel. - Admin should be able to: - Create new blog posts - Edit existing posts - Delete posts - Save posts as Draft or Published - Select category - Set publication date - Add author name and role - Upload a featured image - Edit URL slug - Preview the article before publishing 2. EASY CONTENT EDITOR — NO HTML The blog editor must work like a normal WordPress / Notion / Google Docs editor. Do NOT require HTML editing. Users should be able to directly copy and paste content from Word, Google Docs, Notion or ChatGPT. Include formatting controls for: - Paragraph - H2 - H3 - Bold - Italic - Bullet list - Numbered list - Hyperlink - Remove hyperlink - Image insertion - Undo / Redo The article title must automatically be the only H1 on the page. Do not allow additional H1 headings inside the article body.
SEO SETTINGS Add a separate SEO section for every blog post. Fields: - SEO Title - Meta Description - Focus / Target Keywords - Canonical URL - URL Slug Recommended limits: - SEO Title: max 70 characters - Meta Description: max 160 characters If SEO Title or Meta Description is left empty, automatically use the blog title and summary. Canonical URL should automatically point to the blog post’s own URL unless a custom canonical is manually entered. 4. BLOG POST STRUCTURE Each blog post should support: - Title / H1 - Slug - Category - Summary / Excerpt - Featured Image - Author - Author Role - Publication Date - Article Content - SEO Settings - Draft / Published status 5. FEATURED IMAGE OPTIMIZATION When a featured image is uploaded: - Automatically convert it to WebP - Resize the long edge to approximately 1600 px - Compress it for fast loading - Remove EXIF metadata - Generate an SEO-friendly filename - Maintain consistent blog cover image proportions Target visual ratio: approximately 16:9.
BLOG LISTING PAGE Create a dedicated /blog page. It should include: - Featured image - Blog title - Short description - Category - Publication date - Author - Read More link Keep the design minimal, premium and consistent with the current NEXA website. 7. HOMEPAGE BLOG SECTION Add a stylish blog section to the homepage showing the latest 3 published articles. Each card should include: - Featured image - Category - Title - Short description - Read More Add a “View All Articles” CTA linking to /blog. This section should visually match the current website and should not feel like a separate template. 8. BLOG DETAIL PAGE Each article should have its own SEO-friendly page: /blog/article-slug Layout should include: - Featured image - H1 title - Category - Publication date - Author - Article content - H2 / H3 hierarchy - Internal and external hyperlinks - Related articles section - Back to Blog link 9. LINK SETTINGS For hyperlinks inside articles: - Allow custom anchor text - Allow internal or external URLs - External links can optionally open in a new tab - Automatically add appropriate rel attributes where necessary 10. SEO TECHNICAL REQUIREMENTS For every published blog post automatically generate: - Correct HTML title tag - Meta description - Canonical tag - Open Graph title - Open Graph description - Open Graph image - Twitter Card data
Blog posts must also be added automatically to the sitemap.xml. Draft posts must: - Not appear on the website - Not appear in sitemap - Not be indexable by search engines 11. STRUCTURED DATA Add Article / BlogPosting Schema automatically to each blog detail page. Include where available: - headline - description - image - author - datePublished - dateModified - publisher - mainEntityOfPage 12. IMPORTANT UX REQUIREMENT The admin blog system should be designed for a non-technical user. The workflow should simply be: Create Post → Enter Title → Paste Content → Format H2/H3 → Add Links → Upload Image → SEO Settings → Preview → Publish No HTML, markdown or code knowledge should be required.
The rich text editor must store and render semantic content correctly. H2 and H3 selections in the admin panel must render as actual <h2> and <h3> elements on the public blog page, not visually styled paragraphs.
```

### B) Nextera implementation notes (apply together with A; they do not change A)

- **"NEXA website" in A means the Nextera website** — use the existing Nextera v3 design system (white base, gradient headline words, Sora/Inter/Space Grotesk, Horizon Line, chapter labels).
- **Admin location:** "Blog" item in the U-102 admin sidebar (`/yonetim/blog/`). UI language Turkish (labels such as "Yeni yazı", "Taslak kaydet", "Yayınla", "Önizle", "SEO ayarları", "Öne çıkan görsel").
- **Data (Supabase):** table `blog_posts` — `id`, `title`, `slug unique`, `category`, `excerpt`, `content_json jsonb` (Tiptap document), `content_html text` (sanitised, generated on save), `cover_path`, `cover_alt`, `author_name`, `author_role`, `status` (`draft` | `published`), `published_at`, `seo_title`, `meta_description`, `focus_keywords text[]`, `canonical_url`, `created_at`, `updated_at` (→ `dateModified`). RLS: public select only `status='published' and published_at <= now()`; admins full access. Storage bucket `blog` (public read, admin write) for covers and in-article images.
- **Categories:** the five practice areas (Hibe ve Teşvik, İhracat ve Dış Ticaret, Pazarlama, Yönetim, İnsan Kaynakları) + "Genel"; stored as a small `blog_categories` table so the owner can add more later. Each category links a "İlgili hizmet" box on the article to the hub page.
- **Editor details:** toolbar with Turkish tooltips; paste handling cleans Word/Google Docs/Notion/ChatGPT markup (keep p, h2, h3, strong, em, ul, ol, li, a, img; drop inline styles, fonts, colours, spans); any pasted H1 becomes H2, H4–H6 become H3; the editor offers only Paragraf / H2 / H3. Link dialog: anchor text, URL, "Yeni sekmede aç" toggle; internal link detection (same domain or starts with `/`) → no `target`, no `nofollow`; external → `rel="noopener noreferrer"` (+ `target="_blank"` if chosen). In-article images: same WebP pipeline (long edge 1600px), alt text required, rendered with `loading="lazy"` and width/height.
- **Featured image pipeline (client-side, no server dependency):** canvas re-encode to WebP (quality ≈ 0.8) — this also strips EXIF; crop to 16:9 with a simple focal-point/crop picker; outputs 1600×900 and 800×450; filename = `{slug}-{yyyy-mm}.webp` (Turkish characters transliterated); alt text required before publishing.
- **Slug:** auto-generated from the title (Turkish-safe transliteration), editable, uniqueness check; changing the slug of a published post keeps a redirect from the old slug (`blog_redirects` table).
- **Preview:** "Önizle" opens `/yonetim/blog/onizleme/{id}` rendering the real public layout with a "Taslak önizleme" banner; `noindex`.
- **Public routes:** `/blog/` and `/blog/{slug}/` (trailing slash like the rest of the site). Listing: category tabs, 12 per page, newest first. Detail: breadcrumb, reading time, share links, "İlgili hizmet" box, related articles (same category, 3), "← Blog’a dön".
- **Homepage section (latest 3):** place it after the YouTube section and before the closing CTA; header label `BLOG`, H2 "Güncel **içgörüler**", CTA "Tüm yazıları gör" → `/blog/` (Turkish for "View All Articles"), "Devamını oku" for "Read More". 3 equal cards on desktop, 2 on tablet, swipe row (U-101 standard) on mobile. Hidden when there are no published posts.
- **Freshness / sitemap:** public pages are prerendered at build and refresh on the client; a slug that was not prerendered renders client-side (never 404 for a published post). `sitemap.xml` includes published posts with `lastmod = updated_at`; drafts excluded; draft/preview URLs send `noindex`. If the hosting supports a deploy hook, call it on publish/unpublish (Supabase webhook); otherwise document in `docs/blog.md` that republishing refreshes prerendered HTML.
- **Schema:** `BlogPosting` with `publisher` = the site's Organization (@id), `author` = Person (name + jobTitle from author role), `image` = 1600 cover, `mainEntityOfPage` = canonical; plus `BreadcrumbList`.
- **Performance:** public blog pages must not load Tiptap/DOMPurify; render stored `content_html` (already sanitised). Covers served with `srcset` (800/1600). Home JS grows ≤ 5 KB gzip.
- **SEO tab:** after this block, U-103's "Blog Yazıları" tab reads from `blog_posts`.

### Acceptance
- [ ] Non-technical workflow works end to end: Yeni yazı → başlık → yapıştır → H2/H3 → link → görsel → SEO → önizle → yayınla.
- [ ] Pasted Word/Google Docs content keeps headings/lists/links, loses messy styles; no H1 inside the body.
- [ ] Public HTML shows real `<h2>`/`<h3>`; title is the only `<h1>`.
- [ ] Cover is WebP, ~1600px long edge, 16:9, EXIF-free, SEO filename.
- [ ] Drafts invisible publicly, not in sitemap, `noindex`; published posts in sitemap with correct canonical/OG/Twitter tags and BlogPosting schema.
- [ ] Home shows the latest 3 posts; "Tüm yazıları gör" goes to `/blog/`.

---

## Template for next updates

```md
## U-1XX · <kısa başlık>
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
