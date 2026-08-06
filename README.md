# gatewaze-template-speaker-cards

Speaker promo-card templates, brand colorways, and the event→brand mapping
for the `event-speakers` module's promo kits. Follows the same convention as
`gatewaze-template-email` / `gatewaze-template-blocks`: content that
designers iterate on lives in git, not in the platform codebase.

The event-speakers promo-kit worker fetches this repo (config
`SPEAKER_CARDS_TEMPLATE_REPO`, e.g.
`https://github.com/gatewaze/gatewaze-template-speaker-cards#main`),
caches it briefly, and falls back to the templates vendored with the module
whenever the repo is unset or unreachable — so a bad push here can never
break kit generation.

## Layout

```
templates/
  speaker-card-square.html      # 1080×1080 → exported 1200×1200 (feed post)
  speaker-card-story.html       # 1080×1920 (stories)
  speaker-card-landscape.html   # 1200×630 (link previews / X)
brands/
  <key>.json                    # colorway: accent, accent_bright, accent_dark, wave_from, lockup
  lockup-<key>.svg              # the vertical's logo lockup ({{brand.lockup_url}})
mapping.json                    # event → brand rules (first match wins) + default_brand
```

## Editing rules

- Templates are **self-contained HTML** — fonts and art embedded as data
  URIs, `{{scope.field}}` variables left in place (see the header comment in
  each template for the variable list). Open one in a browser to preview
  with sample data. The in-page scripts (colorway config, auto-fit,
  silhouette fallback) are part of the render contract — the worker waits on
  them, so keep them intact.
- The renderer HTML-escapes every substituted value; templates must keep
  treating variable values as text, never as markup.
- Brand hex values must be `#rgb`/`#rrggbb`(`aa`) — the worker validates and
  ignores anything else.
- `mapping.json` matching is deliberately substring/exact only (no regex).
  Prefer durable rules (`title_contains: "finance"`) over hardcoded
  `event_id`s so next year's forums inherit their branding automatically.

## Adding a forum vertical

1. Add `brands/<key>.json` with the accent set (from
   `AgentsForum_Logo_Reference.pdf`) and drop the vertical's
   `AgentsForum_Logo_<Vertical>_Color_Negative.svg` in as
   `brands/lockup-<key>.svg`.
2. Add a rule to `mapping.json`.
3. Push — the worker picks it up on the next kit generation (cache ≤10 min).
