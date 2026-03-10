# Brand Asset Manifest -- SGA Dental Partners

> Scraped from https://sgadental.com/ on 2026-03-07
> Tagline: "Clinical freedom surrounded by the best of everything"

---

## Logo

| File | Format | Size | Notes |
|------|--------|------|-------|
| `SGA-logo.png` | PNG | 28 KB | User-provided logo (from `sgadental/brand-assets/`) |
| `sga-dental-logo.svg` | SVG | 6.2 KB | Primary site logo (header, 252x75px) |
| `sga-trademark-logo.png` | PNG | 83 KB | Trademark wordmark (2048x558px, high-res) |
| `smiley-favicon.svg` | SVG | 1.3 KB | Favicon / app icon -- smiley tooth graphic |
| `green-bar-accent.svg` | SVG | 468 B | Decorative green bar used as section divider |
| `footer-graphic.svg` | SVG | 5.2 KB | Abstract graphic used in footer section |

---

## Brand Colors

### Primary Palette (from Elementor Global Tokens + Theme CSS)

| Role | Color Name | Hex | RGB | Source |
|------|-----------|-----|-----|--------|
| **Primary Blue** | SGA Blue | `#2D6FB7` | rgb(45, 111, 183) | Homepage hero backgrounds, headings, link color, cursor follower, buttons |
| **Accent Green/Lime** | SGA Lime | `#C4D347` | rgb(196, 211, 71) | Elementor custom color (`--e-global-color-2f40205`), form field borders (`#CADF04`) |
| **Accent Cyan** | SGA Cyan | `#009DD6` | rgb(0, 157, 214) | Elementor custom color (`--e-global-color-cb205a6`) |
| **Accent Green** | Elementor Accent | `#61CE70` | rgb(97, 206, 112) | Elementor global accent color (`--e-global-color-accent`) |
| **Accent Orange** | CTA Orange | `#F36F21` | rgb(243, 111, 33) | Search submit button, call-to-action elements |
| **Nav Hover** | Hover Red | `#DE4625` | rgb(222, 70, 37) | Navigation link hover/active/focus state |

### Neutral Palette

| Role | Color Name | Hex | Source |
|------|-----------|-----|--------|
| **Background (Primary)** | White | `#FFFFFF` | Page backgrounds, section backgrounds |
| **Background (Alt)** | Off-White | `#FAFAFA` | Search input, form backgrounds |
| **Text (Primary)** | Dark Gray | `#333333` | Body text, figcaption |
| **Text (Secondary)** | Medium Gray | `#4F4F4F` | Dark nav menu links |
| **Text (Tertiary)** | Light Gray | `#7A7A7A` | Elementor global text color (`--e-global-color-text`) |
| **Text (Muted)** | Gray | `#666666` | Form borders, menu indicators |
| **Borders** | Light Gray | `#CCCCCC` | Table borders, dividers |
| **Stripe** | Pale Gray | `#EEEEEE` | Submenu borders |
| **Button Dark** | Charcoal | `#32373C` | WordPress default button background |
| **Elementor Secondary** | Dark Slate | `#54595F` | Elementor global secondary (`--e-global-color-secondary`) |

### Elementor CSS Custom Properties

```css
--e-global-color-primary: #6EC1E4;
--e-global-color-secondary: #54595F;
--e-global-color-text: #7A7A7A;
--e-global-color-accent: #61CE70;
--e-global-color-2f40205: #C4D347;   /* Lime green */
--e-global-color-cb205a6: #009DD6;   /* Cyan blue */
```

### Usage Notes

- **`#2D6FB7`** is the dominant brand color -- appears in hero sections, headings, text overlays, and backgrounds on every page
- **`#C4D347` / `#CADF04`** (lime/yellow-green) is used as form accents and the decorative "green bar" divider
- **`#DE4625`** (red-orange) is reserved for hover states in navigation
- **`#F36F21`** (orange) is used for CTA submit buttons
- Semi-transparent blue `rgba(45, 111, 183, 0.2)` is used for pulse animations and button backgrounds

---

## Typography

### Adobe Typekit Fonts (loaded via `use.typekit.net/xid1tyx.css`)

| Role | Font Family | Weight(s) | Style | Source |
|------|-----------|-----------|-------|--------|
| **Primary Headings** | `lft-etica` | 900 (Black) | Normal | Homepage hero headings (60px), section titles |
| **Subheadings** | `lft-etica` | 300 (Light), 700 (Bold) | Normal | Section subheadings (27px-33px) |
| **Body Text** | `lft-etica` | 400 (Regular), 700 (Bold) | Normal | General body copy (20px-23px) |
| **Body (Light)** | LFT Etica Light | 300 | Normal | Theme CSS forms and light body text |
| **Display** | `etica-display` | 200, 900 | Normal, Italic | Available for display use |
| **Condensed** | `lft-etica-condensed` | 400, 700 | Normal, Italic | Available for compact layouts |
| **Compressed** | `lft-etica-compressed` | 400, 700 | Normal, Italic | Available for tight layouts |
| **Stencil** | `axia-stencil` | 300, 900 | Normal | Decorative/display use |
| **Base/Fallback** | `axia, Roboto, Arial, sans-serif` | 400, 700 | Normal | CSS body fallback stack |

### Google Fonts (loaded locally via Elementor)

| Role | Font Family | Weight | Source |
|------|-----------|--------|--------|
| **Elementor Primary** | Roboto | 600 (Semi-Bold) | `--e-global-typography-primary-font-family` |
| **Elementor Secondary** | Roboto Slab | 400 (Regular) | `--e-global-typography-secondary-font-family` |
| **Elementor Body** | Roboto | 400 (Regular) | `--e-global-typography-text-font-family` |
| **Elementor Accent** | Roboto | 500 (Medium) | `--e-global-typography-accent-font-family` |

### Other Fonts

| Role | Font Family | Source |
|------|-----------|--------|
| Navigation toggle | Fira Sans, serif | Theme Elementor CSS |
| Footer menus | Fira Sans, serif | Theme Elementor CSS |
| Menu toggle | Arial Black, sans-serif | Theme elements CSS |

### Typography Summary

The site uses **LFT Etica** (Adobe Typekit) as the primary brand typeface across all weights, with **Roboto** and **Roboto Slab** as Elementor defaults. **Fira Sans** appears in navigation/footer elements. The full Etica family (regular, display, condensed, compressed) plus **Axia** and **Axia Stencil** are loaded and available.

---

## Team Photos

All headshots downloaded from the team page at https://sgadental.com/team/

| Name | Title | Filename | Size |
|------|-------|----------|------|
| Jordan DiNola | Chief Executive Officer | `team/jordan_dinola.png` | 1.5 MB |
| Myles McAllister | Chief Operating Officer | `team/myles_mcallister.jpg` | 408 KB |
| Jim Paul | Chief Financial Officer | `team/jim_paul.png` | 1.0 MB |
| Dr. Kyle Hollis | Chief Clinical Officer | `team/dr_kyle_hollis.jpg` | 439 KB |
| Dr. Zackary L. Bentley | Chief Development Officer, Cofounder | `team/dr_zackary_bentley.jpg` | 552 KB |
| Dr. Lake Garner | Cofounder | `team/dr_lake_garner.jpg` | 693 KB |
| Dr. Jay Williams | Cofounder | `team/dr_jay_williams.jpg` | 412 KB |
| Dr. Chad Fussell | President of the Clinical Advisory Board | `team/dr_chad_fussell.jpg` | 246 KB |
| Steven Van Scoyoc, DDS, MS | Clinical Director of Perio | `team/steven_van_scoyoc.png` | 581 KB |
| Ron Kerensky | Chief Information Officer | `team/ron_kerensky.png` | 1.1 MB |
| Rebecca Roberson | Chief Human Resources Officer | `team/rebecca_roberson.jpg` | 317 KB |
| Pam Mullis | Senior Vice President of Finance | `team/pam_mullis.jpg` | 241 KB |
| Jonathan Mikailonis | VP of Mergers & Acquisitions | `team/jonathan_mikailonis.jpg` | 438 KB |
| Gail Clark | Director of Integrations | `team/gail_clark.jpg` | 417 KB |
| Beth Horowicz | General Counsel | `team/beth_horowicz.jpg` | 425 KB |

---

## Additional Images

### Brand Graphics

| Description | Filename | Size | Source URL |
|-------------|----------|------|------------|
| Team group photo (footer) | `team-footer-dentists.jpg` | 128 KB | `uploads/2021/12/team-footer-dentists.jpg` |
| Abstract footer graphic | `footer-graphic.svg` | 5.2 KB | `uploads/2022/01/Group-1.svg` |
| Green bar section divider | `green-bar-accent.svg` | 468 B | `uploads/2021/12/green-bar.svg` |

### Homepage Images

| Description | Filename | Size | Source URL |
|-------------|----------|------|------------|
| Practice/office photo 1 | `homepage/practice-1.png` | 888 KB | `uploads/2022/01/1.2.png` |
| Practice/office photo 2 | `homepage/practice-2.png` | 658 KB | `uploads/2022/01/2.png` |
| Practice/office photo 3 | `homepage/practice-3.png` | 333 KB | `uploads/2022/01/3.png` |
| "Let's Talk" CTA graphic | `homepage/lets-talk.png` | 22 KB | `uploads/2021/12/lets-talk-1.png` |
| "Schedule" CTA graphic | `homepage/schedule.png` | 25 KB | `uploads/2021/12/schedule.png` |
| "Freedom to Be Yourself" graphic | `homepage/freedom-to-be-yourself.png` | 270 KB | `uploads/2021/12/freedom-to-be-yourself.png` |

---

## Social Media Profiles

- Facebook: https://www.facebook.com/sgadentalpartners
- Instagram: https://www.instagram.com/sgadental
- LinkedIn: https://www.linkedin.com/company/sgadentalpartners

---

## Technical Notes

- **CMS:** WordPress with Elementor page builder (v3.33.6)
- **Theme:** Custom `sgadental` theme
- **Font loading:** Adobe Typekit (`use.typekit.net/xid1tyx.css`) + locally-hosted Google Fonts via Elementor
- **Container max-width:** 1140px
- **Responsive breakpoints:** 1024px (tablet), 767px (mobile)
- **Image CDN:** Direct WordPress uploads (no external CDN detected)

---

## AI-Generated Placeholders Needed

- Office/facility photos (interior, exterior) -- the 3 homepage practice images may partially cover this
- Clinical/procedure imagery (operatory, chair-side, hygiene)
- Patient smile transformations (before/after)
- Technology/equipment shots (CBCT, intraoral scanners, etc.)
- Location-specific exterior photos for individual practices
- Lifestyle/community imagery aligned with brand values

## Image Generation Approved

- Total: 103 images
- Budget: $2.81
- Premium: 25 images (heroes, primary creatives, thumbnails)
- Standard: 78 images (social, sections, headers)
