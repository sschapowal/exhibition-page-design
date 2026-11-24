# Anchor Link Bar — Behavior Spec

Sticky in-page navigation for exhibition detail pages, and how it works with the global nav.

- **Reference prototype:** `r3-a.html` (markup + JS in the first `<script>` block), styles in `styles/main.css` (`.inpage-nav*`, `.site-nav--scrolled`) and `styles/tokens.css`.
- **Figma:** AnchorLinkBar — desktop `2386:38735`, mobile `2386:39096` (file `kfWr23H6UOMF0XCG83EO7G`).
- **Breakpoint:** mobile `< 768px`, desktop `≥ 768px`.

---

## 1. Links

| Label | Target section |
|---|---|
| About the exhibition | Overview (quote, body, sidebar) |
| Artists | Learn about the artist |
| Events | Events |
| Shop | Shop |
| Audio | Audio guide |
| Videos and podcasts | Videos and podcasts |
| Articles | Articles |
| Publications | Publications |
| Images | Exhibition images |
| Exhibition support | Credits / special thanks |

- **Page sections must appear in the same order as the links.** The highlight follows page position, so a mismatched order makes the highlighted label disagree with the section you're reading.
- A section that has no link (e.g. Research resources, between Images and Exhibition support) counts as part of the section above it.
- Links are real `<a href="#section-id">` anchors inside `<nav aria-label="Exhibition sections">`, so they still work without JS.

---

## 2. Visual spec

| | Mobile | Desktop |
|---|---|---|
| Height (not stuck) | 44px + 1px border | 62px + 1px border |
| Height (stuck) | 44px + 1px border | **54px** + 1px border |
| Padding top / bottom | 8px / 24px | 24px / 24px (**16px** / 24px when stuck) |
| Inline padding (both sides) | 24px | `max(48px, (100% − 1184px) / 2)` — i.e. aligned to the 1184px content column |
| Label type | MoMA Sans Bold 16px, letter-spacing −0.36px | MoMA Sans Bold 18px, letter-spacing −0.36px |
| Gap between links | 24px | 48px |
| Arrow size / gap to label | 12px / 2px | 14px / 2px |
| Down-arrow baseline nudge | +1px down (down arrows only) | none |
| Label line-height | 12px (= arrow height) | 14px (= arrow height) |
| Bottom border | 1px `#EEEEEE`, max 1184px wide, aligned to content column; does **not** scroll with the links | same |
| Background | white | white |

**Colors**
- Active label: `#000000`, no visible arrow.
- Inactive label: `#767676` with arrow (`#767676`).
- Hover: label and arrow turn black.

**Arrow asset:** `img/shared/icons/InPageNav_Arrow.svg` (down arrow).

**Space below the bar:** the first content below the bar ("About the exhibition") starts **48px** below the bar's bottom at all breakpoints.

---

## 3. Arrow states

Each link's arrow shows where its section is relative to the one you're in:

| State | Arrow |
|---|---|
| Active (current section) | Hidden with `opacity: 0`. **It still takes up its space**, so labels never shift sideways when the highlight changes. |
| Section is below | Points down. |
| Section is above | Points up (rotated 180°). |

**Transitions**
- **Direction switches instantly (no spin).** When a link stops being active, its arrow fades in already pointing the right way: opacity 0 → 1 over 0.2s ease.
- Keep rotation in its own property (`rotate`), separate from the hover movement (`translate`), and don't animate `rotate`.
- **Hover:** the arrow nudges 4px in the direction it points (down arrows move down, up arrows move up) and turns black, over 0.15s ease.

---

## 4. Sticky behavior

- The bar is `position: sticky` and sticks to the **visible bottom of the global nav**:
  - Mobile: 76px.
  - Desktop: 158px with the nav expanded, 74px collapsed. See §7.
- When the global nav collapses or expands, the bar's `top` animates with it: **0.3s ease-in-out**, the same timing as the nav, so the two move together.
- **"Stuck" detection:** the bar counts as stuck once its in-flow position reaches the nav's bottom. To read that position, the prototype puts a zero-height element directly above the bar. Don't read the bar's own position: it changes during the slide animation and makes the stuck state flicker.
- **Desktop stuck padding:** while stuck, top padding drops from 24px to 16px. To keep the content below from jumping, add the 8px difference as `margin-bottom` on the bar while stuck. Animate both (0.3s ease-in-out).
- **No gap between nav and bar.** No sliver of page content may show between the global nav and the stuck bar.
  - **Exact nav height:** measure it with fractional precision (`getBoundingClientRect().height`, not `offsetHeight`). Re-measure **after web fonts load** and on resize; the fallback font can change the nav height by a pixel, which caused a 1px gap. Use the same measurement for the page's top offset and the bar's sticky `top`.
  - **White strip:** the bar paints an 8px white strip upward behind the nav, so sub-pixel rounding (browser zoom, fractional pixel densities) or the two animations drifting apart can't expose content. The prototype does this with `box-shadow: 0 -8px 0 white`. The nav is stacked above the bar, so the strip is hidden behind it, and above the unstuck bar there's only white space.

---

## 5. Active section (highlight)

- **The trigger line** is the bar's bottom + **48px**, with 1px of tolerance.
- **Each section's title** is its first line of text: its first `h1`, `h2`, `h3` or `p`. That's usually the heading. For Overview it's the quote; for Audio it's the "Audio guide" label.
- **Active section** = the lowest section on the page whose title's **cap height top** has reached the trigger line. If none has, the first section is active.
- **Measuring the cap top:**
  - The cap top is the top of the capital letters, without the line-height's half-leading or the space above the capitals.
  - The prototype measures it by comparing the text's position with and without `text-box: trim-both cap alphabetic`. The difference is that title's offset.
  - Re-measure after fonts load and on resize.
- **Use page position, not link order**, to decide which sections are above or below the current one.
- Re-evaluate on every scroll event.
- **Use the final positions,** not mid-animation ones: the nav and bar's settled height after any running transition.

---

## 6. Clicking a link

1. Smooth-scroll so the target section's **title cap top lands exactly 48px below the bar's bottom**.
2. **Calculate the target using the layout the page will have when the scroll ends**, not the layout at click time:
   - The bar will be stuck. On desktop that's 54px tall, not 62px.
   - The global nav ends up **collapsed** if it's collapsed now, or if the scroll goes down far enough to collapse it (destination > 40px and more than 6px below the current position).
   - Otherwise the nav stays **expanded**.
3. **The click scroll must never reopen the global nav**, even when scrolling up (e.g. clicking "Artists" from "Shop"):
   - While a click-triggered scroll runs, the nav may collapse but not expand.
   - The lock ends when scrolling stops: on the `scrollend` event, or 200ms after the last scroll event as a fallback.
   - Direction tracking then restarts from the landing position, so the distance the click scrolled doesn't count as the user scrolling up.
4. Update the URL hash with `history.replaceState` (no extra history entry, no native jump).
5. The highlight then follows from §5, because the title is now at the trigger line.

**Expected results** (verified in the prototype for all 10 links at 1440px and 390px, both from the top of the page and clicking forward/back in sequence):
- The title cap top is exactly 48px below the bar.
- The clicked link is highlighted.
- The global nav state follows the rules above.

---

## 7. Global nav interaction (desktop collapse, MoMA.org-style)

The global nav is a single `position: fixed` element. It **slides**; it never swaps to a separate condensed version.

| | Behavior |
|---|---|
| At the top (scroll ≤ 40px) | Always expanded (158px). |
| User scrolls **down** (> 40px, after ≥ 6px of downward travel) | Collapses: the whole nav moves up (`translateY`) by the distance from its top to the links row minus its top padding (**84px**), leaving a 74px strip with the links row. Donate + Tickets slide down to center on the links row; Membership fades out. |
| User scrolls **up** (≥ 6px of upward travel) | Expands again, from any scroll position. |
| Small reversals (< 6px, e.g. trackpad bounce) | Ignored. |
| Anchor-link scroll | Can collapse, never expands (see §6.3). |
| Animation | `transform 0.3s ease-in-out` (nav and Donate/Tickets); Membership `opacity`/`visibility` 0.3s ease-in-out. The anchor bar's `top` uses the same timing. |

- Compute the collapse distance and the Donate/Tickets offset from the actual layout, not hard-coded values. Re-measure on resize and after fonts load, with transitions temporarily off so you read final positions.
- **Mobile:** the nav is a fixed 76px bar (logo, Tickets, hamburger) and doesn't collapse. The anchor bar sticks under it.

---

## 8. Horizontal scrolling of the link row

- The links row scrolls sideways when it's wider than the screen, at **all breakpoints**. Hide the scrollbar.
- Equal inline padding on both ends keeps the first and last links aligned to the content column when the row is scrolled all the way.
- **Bringing the active link into view:**
  - **When:** only when the active link changes.
  - **If it's already fully visible:** don't move the row.
  - **If it's hidden off the right edge:** smooth-scroll the row just enough that the **label's right edge** sits on the right content edge.
  - **If it's hidden off the left edge:** smooth-scroll just enough that the label's left edge sits on the left content edge.
  - **What to measure:** the label text only, not its invisible arrow, so the label lines up exactly with the edge.
  - **Near the end of the list:** the row stops at its natural end, with no extra space added after the last link.
- The user can still swipe or scroll the row by hand. It's only adjusted again when the highlight changes.

---

## 9. Edge cases and notes

- **Text trimming:** the prototype uses CSS `text-box: trim-both cap alphabetic` in places to match Figma's cap-height spacing. Chrome and Safari support it; Firefox ignores it, so spacing there is a few pixels larger.
- **Without JS:** links still jump to their sections. `scroll-margin-top: var(--scroll-offset)` on the sections keeps them clear of the sticky bars.
- **Body offset:** the page's top padding equals the exact, fractional full nav height (the nav is fixed).
- **Resize / font load:** re-measure the nav height, nav collapse distance, Donate/Tickets offset, title cap offsets and bar heights.
