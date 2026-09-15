# Choice Architecture and Conventions

Loaded for option-heavy screens, navigation and menu design, and surfaces that span desktop, mobile, or RTL locales.

## Hick's law

The time to decide grows with the number and complexity of choices — shoppers offered 24 jams buy less than shoppers offered 6. Google's single search box against Yahoo's portal of competing links; In-N-Out's four-item menu against the Cheesecake Factory's 250 — the smaller menu isn't less capable, it's faster to act on.

Reducing decision load rarely means removing capability:

- **A few options plus a filter.** Show a handful of curated choices and let search or filters reach the rest, instead of listing everything.
- **Curation over catalog.** Recommend a shortlist the way Netflix leads with a few rows — around 80% of what people watch there comes from recommendations, not from browsing thousands of titles.
- **Progressive disclosure.** Advanced and rare options live behind "More" or "Advanced"; the first screen carries only the decisions most users actually make.
- **Preselect the right-for-most default.** A good default turns a decision into a confirmation.
- **Long forms are the same law applied to fields** — the split rule lives in [FORMS.md](FORMS.md).

Bar: every screen's visible option count is deliberate — each option either serves most users or has moved behind disclosure, search, or a default.

## Jakob's law

People spend most of their time in other people's apps, so they arrive expecting yours to work like the web they already know. Meeting that expectation is a feature: the borrowed convention is interaction design the user has already learned. Spend deviation only where the deviation is the product — the one novel interaction that defines you — and be conventional everywhere else so the novelty gets the user's full attention.

Bar: every departure from a common pattern is a deliberate, stated choice, not a side effect.

## Platform and locale conventions

The conventions Jakob's law borrows differ by platform and locale:

| Context | Conventions to honor |
|---|---|
| Desktop web | Cart and account top-right; primary nav top or left; hover states are legitimate affordances. |
| Mobile | Primary actions at the bottom, inside thumb reach — the top corners are the hardest one-handed targets; no hover assumptions; touch targets sized for fingers. |
| RTL locales | Layout mirrors (the cart moves to the top-left); directional iconography mirrors (back arrows, progress); use logical CSS properties (start/end) so mirroring is automatic. |

Design for the platforms and locales your users actually inhabit, not the one on your desk.

Bar: each screen's layout matches the norms of the platform and locale it ships to, or the deviation is documented as deliberate.
