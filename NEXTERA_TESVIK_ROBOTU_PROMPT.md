# NEXTERA — "TEŞVİK ROBOTU" FEATURE PROMPT (Phase add-on)

> Prompt language: English · UI language: Turkish · Attached data file: **`tesvik-robotu-data.json`**
> This is an add-on to the existing Nextera project (master prompt v3). All design rules of v3 apply: white-dominant background, soft tonal bands, gradient headline words, Horizon Line, Sora / Inter / Space Grotesk, no old-site patterns, mobile-first, prerendered, fast.
> Work in the steps of §12 and stop after each.

---

## 0. WHAT WE ARE BUILDING

A standalone, highly engaging tool page: **"Nextera Teşvik Robotu"** at **`/yatirim-tesvik-robotu/`**.
A visitor picks a province (any of Türkiye's 81), a sector (NACE activity from the Decree's annex), and a few investment details; the robot instantly tells them **which incentive system applies, which support elements they get, for how long and roughly how much** — based strictly on **Presidential Decree No. 9903, "Yatırımlarda Devlet Yardımları Hakkında Karar" (Resmî Gazete 30 May 2025)** and its annexes (EK-1 … EK-5).

It must be: complete (all 81 provinces, all EK-3 sectors, all Madde 9 priority topics, EK-4 machinery list, EK-5 districts), correct (every result cites the article it comes from), fast (the site must not get slower), and fun to use (live map, instant recalculation, comparison across provinces).

---

## 1. DATA FILE — `tesvik-robotu-data.json`

Place the delivered file at `src/features/tesvik/data/tesvik-robotu-data.json`. At build time, split it with a small script into three chunks so the page only loads what it needs:

- `core.json` → `meta`, `iller` (81 provinces), `kurallar` (all rules/parameters), `ek1TeknolojiSiniflari`, `madde9OncelikliKonular` (~25 KB raw)
- `sektorler.json` → `ek3Sektorler` (EK-3: sections + 85 activities with verbatim conditions)
- `makineler.json` → `ek4GumrukMuafiyetiDisiMakineler` (EK-4: 233 GTİP lines)

Contents (use verbatim, never re-type):

| Key | What it is |
|---|---|
| `iller[]` | `plaka`, `ad`, `slug`, `bolge` (1–6, from EK-2), `lat`/`lng` (approx. province-centre coordinates, only for the dot map), `ek5Ilceler[]` (EK-5 districts giving "alt bölge" advantage), `cazibeMerkeziIli` (Geçici Madde 4 province flag — marked "doğrulanmalı") |
| `ek1TeknolojiSiniflari[]` | NACE codes classed "yuksek" / "ortaYuksek" (EK-1) |
| `ek3Sektorler[]` | `tip: 'bolum'` rows = EK-3 section headers (A, B, C, D, E, H, I, K, N, O, Q, R; section B carries its own conditions); `tip: 'faaliyet'` rows = eligible activities with `kod`, `kodlar[]`, `tanim`, `sartlar` (verbatim conditions text), `bolum`, `imalat`, `teknolojiSinifi`, `enerjiTuru`, `sadece6Bolge`, `istanbulHaric` |
| `madde9OncelikliKonular[]` | The 27 priority-investment topics of Madde 9/1 (a–y), verbatim, with helper flags (`otomatik`, `faizYok`, `asgariSYT`, `istanbulHaric`, `ek3SartiYok`, `asgariSYTYok`) |
| `ek4GumrukMuafiyetiDisiMakineler[]` | 233 GTİP numbers + names that do NOT get customs-duty exemption |
| `kurallar` | Every numeric rule of the Decree: minimum fixed-investment amounts, the five systems with their support elements, tax-reduction contribution rates, interest-support shares/caps/limits, machine support, SGK durations, alt-bölge rules, Geçici 3/4, nakil, restrictions with article numbers, support-element descriptions, admin parameters |

**Admin parameters** live in `kurallar.parametreler` and `kurallar.tlGuncelleme`. Values that are `null` (TCMB one-week repo rate, gross minimum wage, SGK employer rate) must be editable by the owner in one config file `src/features/tesvik/config.ts` and by the user inside the tool ("Kendi değerimi gireceğim"). If a needed parameter is empty, the robot shows the rule (%, years, caps) but not a TL amount — never invent a value.

**TL amounts:** the Decree's TL figures are 2025 values and are revalued yearly by tebliğ (Madde 5/10). Show a small note "Karardaki 2025 tutarları esas alınmıştır; yıllık güncellenen tutarlar için uzmanımıza danışın." and apply `tlGuncelleme.katsayi` (default 1.0) to all TL thresholds/caps.

**Known gaps (show as notes, do not guess):** the district list of Geçici Madde 3 (earthquake districts, 2018/11201 ek-2) is not in the file → the user answers a yes/no question instead. The Geçici Madde 4 province list is included but flagged for verification.

---

## 2. THE RULE ENGINE (pure TypeScript, unit-tested)

Create `src/features/tesvik/engine/` with pure functions — no React, no I/O. The engine must run all 81 provinces for one scenario in < 10 ms (used by the comparison map).

### 2.1 Input

```ts
interface Scenario {
  ilSlug: string;
  ilceInEk5: boolean;             // chosen district is in EK-5 list of that province
  osbVeyaEndustriBolgesi: boolean;
  sektor: { type: 'ek3'; id: string } | { type: 'madde9'; bent: string } | { type: 'tykh'; program: 'teknolojiHamlesi'|'yerelKalkinmaHamlesi'|'stratejikHamle' };
  teknolojiSinifi?: 'yuksek'|'ortaYuksek'|null; // auto from EK-1, user can confirm
  oncelikliUrunListesinde?: boolean;            // user checkbox (Teknoloji Hamlesi priority product list)
  yatirimCinsi: 'kompleYeni'|'tevsi'|'modernizasyon'|'urunCesitlendirme'|'entegrasyon'|'nakil';
  muteharrik: boolean;            // mobile-character investment (excluded from 9/1-ç)
  gecici3DepremIlcesi: boolean;   // user yes/no (only offered if application date ≤ 31.12.2026)
  syt: { arazi: number; bina: number; makineYerli: number; makineIthal: number; makine2MUstu: number; maddiOlmayan: number; diger: number };
  kullanilmisMakine: boolean;
  istihdam: number;               // additional employment
  kobi: boolean;
  firmaTescilSon1Yil: boolean;
  yeniFirmaSecenegiKullan: boolean; // Madde 20/4 option (forgo tax reduction for higher interest/machine support)
  kredi?: { tutar: number; vadeYil: number; doviz: boolean };
  ortalamaGumrukVergisiOrani?: number;  // user-entered %, needed for customs saving estimate
  yillikVergiMatrahi?: number;          // expected yearly taxable profit from the investment
  basvuruTarihi: string;                // default today
  parametreler: { repo?: number; kurumlarVergisi?: number; kdv?: number; asgariUcretBrut?: number; sgkIsverenOrani?: number };
}
```

### 2.2 Algorithm (follow exactly, cite articles in output)

1. **Region:** `bolge = iller[il].bolge`.
   **6th-region equivalence:** if (`gecici3DepremIlcesi` and date ≤ 2026-12-31) or (`cazibeMerkeziIli` && `osbVeyaEndustriBolgesi` && sector is manufacturing NACE 10–32 or 38.2 && date ≤ 2026-12-31) → `destekBolgesi = 6` with reason "Geçici Madde 3/4". Otherwise `destekBolgesi = bolge`.
2. **Eligibility of topic:** EK-3 activity required unless TYKH, Dijital/Yeşil (Madde 9/1-a), or Madde 9/1-v (Madde 5/1). Show the activity's verbatim `sartlar` text with a "Bu şartları sağlıyorum" confirmation; if the user does not confirm → result "Şartlar sağlanmadıkça desteklenmez". Apply structured flags: `sadece6Bolge` (82.2 call centres), `istanbulHaric` (section B mining), tanning-only-in-OSB note (15).
3. **Minimum fixed investment (Madde 5/2):** 12 M TL in regions 1–2, 6 M TL elsewhere (× katsayı), unless the chosen topic says otherwise (`asgariSYT`, `asgariSYTYok`, Stratejik 100 M/200 M, Yeşil/Dijital direct 50 M, nakil none). Below minimum → clear "not eligible" state with the gap amount.
4. **System selection:**
   - `tykh` → that program (show "Komite değerlendirmesi ile proje bazında karar verilir"; for Stratejik show the 5 pre-evaluation criteria as a checklist, ≥ 3 required, and SYT ≥ 100 M high-tech / 200 M other).
   - `madde9` → **Öncelikli**.
   - `ek3` → **Hedef**, upgraded to **Öncelikli** automatically when: `destekBolgesi === 6` and not `muteharrik` (9/1-ç); or high-tech and (SYT ≥ 500 M or on priority list) (9/1-b); or mid-high-tech, not İstanbul, and (SYT ≥ 1 B or on priority list) (9/1-c). Show YKT both gross and net of interest/machine support payments (5/7). Always show *why* ("Öncelikli: 6. bölge yatırımı — Madde 9/1-ç").
   - `nakil` from region 1 to 4/5/6, manufacturing, ≥ 50 employees → only regional supports (Madde 18/5).
5. **Support elements** per system from `kurallar.sistemler[x].unsurlar`, then remove per restrictions:
   - Hedef: no tax reduction in İstanbul or for energy kinds; interest support only regions 4–6 and never for energy kinds (10/2, 10/3).
   - Öncelikli (9/1-e solar/wind self-consumption): no interest support (9/2). 6th-region energy exceptions (9/3).
   - No tax reduction → no investment-site allocation; electricity generation → no site allocation (21/3).
   - Used machinery → no tax reduction, machine support, interest support (23/5).
   - TYKH: machine support XOR interest support (user toggles which; default = the larger) (15/9, 16/3).
   - `yeniFirmaSecenegiKullan` (only if `firmaTescilSon1Yil` and `kompleYeni`) → remove tax reduction, add +5 pts / +60 M TL (TYKH), +2 pts / +6 M (Öncelikli), +2 pts / +2.4 M (Hedef) to the interest/machine support's SYT ratio and cap (20/4).
6. **Amounts (only when inputs exist):**
   - `SYT = sum(syt)`; warn if `maddiOlmayan > 25 % SYT` (5/8, not for Dijital); warn ekosistem plan 2 % if not KOBİ or Yerel Kalkınma (5/9).
   - **Tax reduction:** `YKT = (SYT − arazi − (interest/machine support paid)) × yko` (arazi & non-depreciable excluded, 20/3; 5/7). Rate reduced by 60 %: effective rate = KV × 0.4. If `yillikVergiMatrahi` given → yearly tax saving = matrah × KV × 0.6, years to exhaust YKT; also show that up to 50 % of YKT may be used against other earnings (20/2).
   - **Interest support:** eligible credit = min(credit, 70 % SYT); points = min(repo × repoPayi, azamiPuan) (FX credit: 2 pts TYKH / 1 pt sectoral, ≤ 50 % of FX rate); yearly support ≈ outstanding principal × points (equal-principal amortisation over min(term, 5) years); total capped by min(sytOrani × SYT, azamiTutar × katsayı) (Madde 15).
   - **Machine support (TYKH):** 25 % × `makine2MUstu`, capped by min(15 % SYT, cap) (16).
   - **VAT exemption:** (makineYerli + makineIthal) × KDV rate.
   - **Customs exemption:** makineIthal × user-entered average duty rate (else show "oran girin"); remind EK-4 exclusions and link to the GTİP checker.
   - **SGK employer share (18, 22):** base region `destekBolgesi`; alt-bölge: OSB or EK-5 district → 1 region lower; both → 2 lower (cap at 6); if already 6 and advantage applies → +2 years. Duration from table (1: –, 2: 1 y, 3: 2 y, 4: 4 y, 5: 8 y, 6: 12 y); TYKH: 12 y in 6th region, 8 y elsewhere. Coverage: 100 % in 6th-region terms, 50 % otherwise, of the employer share corresponding to minimum wage, for additional employees. TL amount only if minimum wage and employer rate parameters exist.
   - **SGK employee share (19):** only if `destekBolgesi === 6`: 10 years.
   - **Total support estimate** = sum of computed items, shown as a range where inputs are partial, and as "% of SYT".
   - Global cap reminder: total of KDV, GV, tax reduction, interest and machine support cannot exceed realised SYT (5/14).
7. **Output object:** system, reasons, eligibility status, list of support elements (each: status active/removed/conditional, value, duration, article, one-line explanation from `destekUnsurlari`), warnings, required next steps (E-TUYS application, e-fatura requirement from 1.1.2026 — 5/11, 3-year investment period — 29/2), and the verbatim sector conditions.

### 2.3 Tests (Vitest) — must pass

| # | Scenario | Expected |
|---|---|---|
| T1 | Konya, EK-3 `28`, kompleYeni, SYT 150 M, no OSB | Hedef; YKO 20 % → YKT 30 M; no interest (region 2); GV + KDV + site allocation; SGK employer 1 y, 50 % |
| T2 | T1 + OSB + district Çumra (EK-5) | SGK terms of region 4 → 4 y, 50 % |
| T3 | Van, EK-3 `14`, SYT 20 M (no land), not mobile | Öncelikli (9/1-ç); YKO 30 % → YKT 6 M gross, 5.4 M if the full 2 M interest support is paid (5/7); interest limit min(10 % × 20 M, 24 M) = 2 M; SGK employer 12 y, 100 %; SGK employee 10 y |
| T4 | T3 + OSB | SGK employer 14 y |
| T5 | İstanbul, EK-3 `20`, SYT 50 M | Hedef; no tax reduction; no interest; no site allocation; SGK employer none (region 1) |
| T6 | Gaziantep, EK-3 `25`, SYT 5 M | Not eligible: below 6 M minimum (gap 1 M) |
| T7 | Sivas, Madde 9/1-e (GES self-consumption), SYT 40 M | Öncelikli; no interest (9/2); no site allocation (electricity, 21/3); YKO 30 % |
| T8 | Kocaeli, EK-3 `26`, high-tech, SYT 600 M (no land) | Öncelikli (9/1-b); YKT 180 M gross, 172.8 M after deducting the full 24 M interest support (5/7); interest cap min(60 M, 24 M) = 24 M |
| T9 | Repo 40 %, Öncelikli | points = min(10, 12.5) = 10 |
| T10 | Kilis (region 5) + EK-5 district Elbeyli + OSB | 2 regions lower, capped at 6 → region-6 terms: 12 y, 100 % (the +2 y rule of 22/2 applies only when the investment is actually in region 6; show an "uzmanla doğrulayın" note for this capped case) |
| T11 | İstanbul, EK-3 `20`, mid-high-tech, SYT 1.2 B | stays Hedef (9/1-c excludes İstanbul) |
| T12 | Any, used machinery | tax reduction, machine and interest support removed with 23/5 note |

---

## 3. PAGE ANATOMY — `/yatirim-tesvik-robotu/`

### 3.1 Above the fold
- Breadcrumb: Ana Sayfa / Teşvik Robotu.
- H1: **"Yatırım Teşvik Robotu"** (gradient on "Teşvik Robotu").
- One-line lead (new microcopy): "81 il ve sektör için 9903 sayılı Karar’a göre alabileceğiniz teşvikleri saniyeler içinde görün."
- The robot starts immediately (no splash screen): a large **province search field** with autocomplete (Turkish-aware: "izmir" finds "İzmir", "elazig" finds "Elâzığ") + the **dot map** next to it.
- Source strip under it: "Kaynak: 9903 sayılı Cumhurbaşkanı Kararı · RG 30.05.2025 · Karar ekleri EK-1…EK-5" + "Bilgilendirme amaçlıdır" link to the disclaimer.

### 3.2 The dot map (signature visual, zero geo files)
- An SVG "dot map" of Türkiye built from the 81 `lat/lng` points (simple equirectangular projection, lat scaled by cos 39°). Each province = a circle (radius 7–9 px desktop), coloured by region (6-step ramp from `#E9E4FF` (1) to `#270F33` (6), Nextera purples), plate number on hover, name tooltip.
- Legend "1.–6. Bölge" with counts (8 · 13 · 17 · 11 · 15 · 17).
- Click a dot = select province. Selected dot gets a gradient ring + pulse (once).
- **Heat mode:** after a scenario is set, a toggle "Bu yatırım her ilde ne kadar destek alır?" recolours all 81 dots by the engine's total-support % of SYT (gradient scale), and a ranked side list (top 10 / bottom 10) appears. This is the "wow" moment — it must be instant.
- Mobile: map becomes a horizontally scrollable strip or collapses behind a "Haritada gör" button; search is primary.
- No map libraries, no GeoJSON/TopoJSON. Total map code < 8 KB.

### 3.3 The wizard (left/centre) + live result (right, sticky)
Desktop: two columns — wizard steps on the left, **live result panel** on the right updating on every change. Mobile: steps full width, a sticky bottom bar shows "Tahmini destek: %xx · Sistem: Öncelikli" and opens the full result as a bottom sheet.

Steps (each collapsible, with progress Horizon Line):
1. **Yer** — province (search/map), district: chips of that province's EK-5 districts + "Diğer ilçe"; OSB/endüstri bölgesi yes/no; Geçici 3 question (only if date ≤ 31.12.2026); region badge with explanation ("Konya · 2. Bölge").
2. **Konu** — three tabs: "Sektör (EK-3)" searchable list grouped by EK-3 section with codes; "Öncelikli yatırım konuları (Madde 9)" list of the 27 topics; "Türkiye Yüzyılı Kalkınma Hamlesi" (3 programs with short explanation + committee note). Picking an EK-3 activity shows its verbatim conditions in a card with the confirmation checkbox. Technology class auto-detected from EK-1 with an override.
3. **Yatırım** — type (komple yeni, tevsi, modernizasyon, ürün çeşitlendirme, entegrasyon, nakil), SYT breakdown with live total and a stacked bar, used-machinery toggle, KOBİ, company age, employment count. Currency inputs formatted `tr-TR`, big number shortcuts (e.g. "50 M").
4. **Finansman & vergi (opsiyonel)** — loan amount/term/FX, average customs rate, expected yearly taxable profit, parameters (repo, minimum wage…) prefilled from config or blank with "Değer girin" hints.

### 3.4 Result panel (the payoff)
- Headline card: system name (e.g. **"Öncelikli Yatırımlar Teşvik Sistemi"**) + reason chip with article + estimated total support (TL and % of SYT, count-up) + eligibility status.
- **Support-element cards** (creative cards style from v3): one per element — Gümrük vergisi muafiyeti, KDV istisnası, Vergi indirimi, Faiz/kâr payı, Makine desteği, Yatırım yeri tahsisi, SGK işveren hissesi, SGK işçi hissesi. Each: status (Aktif / Uygulanmaz / Koşullu), key number (Space Grotesk), duration, article chip (tap → shows the article's rule sentence from `destekUnsurlari`/`kisitlar`). Removed elements are shown greyed with the reason — users love understanding *why*.
- **Timeline visual:** a horizontal years axis (0–14 y) with bars for SGK employer share, SGK employee share, interest support (≤ 5 y), tax-reduction years (if matrah given).
- **Waterfall chart:** SYT → minus each support → "net yatırım maliyetiniz".
- **Warnings list:** below-minimum, intangible > 25 %, ekosistem plan, e-invoice from 1.1.2026, application deadline 31.12.2030, EK-4 machinery reminder, TL amounts year.
- **Actions:** "Sonucu paylaş" (URL with query params, e.g. `?il=konya&sektor=28&syt=150000000…`), "Yazdır / PDF olarak kaydet" (print stylesheet only — no PDF library), "Karşılaştır" (adds scenario to comparison), and primary CTA **"Sonucu uzmanımızla doğrulayın"** → `/iletisim/?hizmet=yatirim-tesvik-belgesi&ozet=<encoded summary>` (the contact form pre-fills the message with the summary).

### 3.5 Extra tools (tabs under the robot, lazy-loaded)
- **İl karşılaştırma:** pick up to 3 provinces for the same scenario → side-by-side cards + bar chart.
- **81 İl Tablosu:** sortable table: plaka, il, bölge, SGK işveren süresi, SGK oranı, EK-5 ilçe sayısı, (for the current scenario) system and total support %. Filter by region. This table is also prerendered in static form (without scenario columns) for SEO.
- **GTİP sorgulama (EK-4):** search by GTİP or name across the 233 lines → "Bu makine gümrük vergisi muafiyetinden yararlanamaz (EK-4, sıra X)" / "EK-4 listesinde bulunamadı". Load `makineler.json` only when this tab opens.
- **Sektör rehberi (EK-3):** browsable accordion of all EK-3 sections/activities with verbatim conditions + a "Robotta dene" button per activity.

---

## 4. STATIC, CRAWLABLE CONTENT (prerendered, below the tool)

Everything here is derived from the Decree (facts, not marketing). Keep it compact, well structured, and lazy-free (plain HTML):
1. H2 "Teşvik sistemi nasıl işler?" — a schema (v3 Şema style): Türkiye Yüzyılı Kalkınma Hamlesi (3 programs) · Sektörel Teşvik Sistemi (Öncelikli, Hedef) · Bölgesel teşvikler — with their support elements (Madde 4).
2. H2 "Destek unsurları karşılaştırması" — table: rows = support elements, columns = Teknoloji/Yerel, Stratejik, Öncelikli, Hedef — values from `kurallar` (YKO 50/40/30/20 %, interest shares & caps, machine support, etc.).
3. H2 "81 ilin teşvik bölgeleri" — region table (all 81, grouped 1–6) + SGK duration table.
4. H2 "Alt bölge desteği alan ilçeler (EK-5)" — collapsible per province (55 provinces, 289 districts).
5. H2 "Sık sorulan sorular" — 8 short Q&As written strictly from the Decree (e.g. "Asgari yatırım tutarı nedir?", "Hangi bölgede SGK desteği kaç yıl?", "Öncelikli yatırım nedir?", "Faiz desteği ne kadar?", "Vergi indirimi nasıl hesaplanır?", "İstanbul’daki yatırımlar hangi desteklerden yararlanamaz?", "Gümrük muafiyeti hangi makinelerde uygulanmaz?", "Başvurular ne zamana kadar yapılabilir?"). Each answer cites the article. Store in `src/features/tesvik/content/faq.ts` with `generated: true` for owner review. FAQPage JSON-LD.
6. Disclaimer block: "Teşvik Robotu bilgilendirme amaçlıdır; hukuki veya mali danışmanlık niteliği taşımaz. Nihai değerlendirme Sanayi ve Teknoloji Bakanlığı’nın E-TUYS üzerinden yapacağı inceleme ile belirlenir. Tebliğler ve yıllık güncellenen tutarlar sonucu değiştirebilir."

---

## 5. PERFORMANCE — THE SITE MUST NOT GET SLOWER

- The tool is **route-level code-split**; nothing from it loads on other pages (the header link and teasers are plain links).
- Initial route payload: prerendered HTML + CSS + robot shell JS ≤ 60 KB gzip; `core.json` (~6–8 KB gzip) loaded in parallel with `fetchpriority="high"`; `sektorler.json` on first interaction with step 2 (prefetch on idle); `makineler.json` only when the GTİP tab opens.
- Engine runs in the main thread (it is tiny); heat mode computes 81 scenarios with `requestIdleCallback` fallback to a single pass (< 10 ms). If you measure > 16 ms, move it to a Web Worker.
- Charts: lightweight inline SVG (no recharts on this route unless already in the shared chunk).
- Inputs debounced 120 ms; results memoised by scenario hash.
- Lighthouse mobile targets for this route: Performance ≥ 90, INP < 200 ms, CLS < 0.05, SEO 100.
- Everything works offline after first load (static JSON cached by the browser; no API calls).

---

## 6. SEO & SHARING

- Title: "Yatırım Teşvik Robotu | 81 İl ve Sektöre Göre Teşvik Hesaplama | Nextera Danışmanlık"
- Meta description: "9903 sayılı Karar’a göre 81 il ve sektör için yatırım teşviklerini hesaplayın: bölge, destek unsurları, vergi indirimi, faiz desteği ve SGK prim desteği süreleri."
- Canonical `/yatirim-tesvik-robotu/` for all query-param variants (`?il=…` URLs are for sharing, not indexing).
- JSON-LD: `WebApplication` (applicationCategory "BusinessApplication", offers price 0, inLanguage tr) + `FAQPage` + `BreadcrumbList`.
- OG image: white card with the dot map in region colours and the H1.
- Add the route to `sitemap.xml`.

---

## 7. INTEGRATION WITH THE REST OF THE SITE

- Header: add **"Teşvik Robotu"** as a nav item (with a tiny gradient dot "Yeni").
- Mega menu (Hibe ve Teşvik column): a featured tile "Teşvik Robotu — 81 il, tüm sektörler".
- Service pages 01 (Hibe ve Teşvik hub), 06 (Yatırım Teşvik Belgesi), 07 (Ar-Ge ve Tasarım Merkezi), 11 (TKDK), 13 (SGK Teşvik), 40 (GES): a slim CTA band after the Panel — "Yatırımınızın teşvikini hesaplayın → Teşvik Robotu", deep-linking with sensible defaults (e.g. page 40 → `?konu=madde9-e`).
- Page 06's Panel ("Yatırım Avantaj Hesaplayıcı") should reuse the same engine and data (import the engine; no duplicate logic).
- Home page: a compact teaser band after the success-stories carousel — mini dot map + province search that jumps to the robot with `?il=`.

---

## 8. ANALYTICS & LEADS

- Use the existing no-op `analytics.ts`: `trackTool('tesvik_robotu', event, payload)` for: province selected, sector selected, result viewed, heat mode used, compare used, share, print, CTA clicked. No personal data in payloads.
- Contact form: when `ozet` is present, prefill "Mesaj" with the scenario summary and set hidden field `source=tesvik-robotu`; store in Supabase `contact_requests.source`.

---

## 9. ACCESSIBILITY

- Map dots are buttons with `aria-label="Konya, 2. Bölge"`; full keyboard navigation (arrow keys move between nearest dots; Enter selects); the search field is the primary accessible path.
- Result updates announced via a polite live region ("Sonuç güncellendi: Öncelikli Yatırımlar, tahmini destek %34").
- All charts have text equivalents; colour is never the only signal (status also as text/icon).

---

## 10. CONTENT RULES

- Legal texts (EK-3 conditions, Madde 9 topics, EK-4 names, district names) are shown **verbatim** from the JSON.
- Microcopy may be written (labels, hints, button texts). Longer explanations only in the FAQ (flagged for review) and must cite articles.
- Never state a TL amount that relies on a missing parameter.
- Every number on the result screen has an article reference on hover/tap.

---

## 11. FILE STRUCTURE

```
src/features/tesvik/
  data/tesvik-robotu-data.json        (source, not shipped directly)
  data/build-split.ts                 (→ public/data/tesvik/core.json, sektorler.json, makineler.json)
  engine/{types.ts, region.ts, system.ts, supports.ts, amounts.ts, index.ts}
  engine/__tests__/engine.test.ts     (T1–T12)
  config.ts                           (admin parameters, TL katsayı)
  components/{ProvinceSearch, DotMap, Wizard/*, ResultPanel, SupportCard, Timeline, Waterfall, CompareView, ProvinceTable, GtipSearch, SectorGuide, StickyMobileBar}
  content/faq.ts
  pages/TesvikRobotuPage.tsx
```

---

## 12. STEPS (stop after each and report)

1. **Data + engine:** split script, types, engine, tests T1–T12 passing. Show the test output.
2. **Page shell + static content:** route, prerendered sections of §4, SEO tags, JSON-LD, disclaimer.
3. **Robot UI:** province search, dot map, wizard, live result panel, mobile sticky bar/bottom sheet.
4. **Extras:** heat mode, comparison, 81-province table, GTİP search, sector guide, share/print.
5. **Integration:** header/mega menu, service-page CTAs, page-06 panel reuse, home teaser, contact prefill.
6. **QA:** Lighthouse for the route, bundle report (show chunk sizes), accessibility pass, verify that other routes' bundle sizes did not change.

## 13. ACCEPTANCE

- [ ] All 81 provinces selectable; region counts 8/13/17/11/15/17; all 289 EK-5 districts available under their provinces.
- [ ] All 85 EK-3 activities, all 27 Madde 9 topics, 3 TYKH programs, 233 EK-4 lines searchable.
- [ ] T1–T12 pass; every result element cites its article.
- [ ] Heat mode recolours 81 provinces instantly.
- [ ] No other route got heavier (bundle diff shown).
- [ ] Mobile experience complete (search-first, sticky result bar, bottom sheet).
- [ ] No invented values; missing parameters handled gracefully; disclaimer visible.
