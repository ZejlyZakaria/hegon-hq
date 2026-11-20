# List artwork and accolade surface - 2026-11-01

Owner approved an IMDb-inspired list collage and restrained gold award highlighting.

InList now presents one144px-high clickable tile per membership: up to three real stored list posters, cover cropping without stretching, dark lower mask and readable list name. Zero posters has a neutral icon fallback; one/two posters fill available cells without invented images. Removed the separate removal control and its unused callback; membership add/remove/create stays in Manage, and tile navigation/See all remain. Reused existing image-size helper and Next Image, no new provider reads. Manage hint now names membership management.

Accolades retains its plain shared heading typography and uses a dark gold-tinted surface and quiet gold border. Actual wins (canonical or positive OMDb win evidence) receive9percent tint; nomination-only evidence receives3percent. No award status, counts, catalogue or personal data rewritten; the existing award-details door remains.

Validation: TypeScript and affected lint pass; DS coverage regenerated and whitespace checked. Native browser fixture renders actual InList/Accolades, controls and CSS with deterministic0/1/2/3poster data at390/768/1024/1440, exercising image loading, list navigation and Manage add/remove, absence of nested tile actions, and different computed surfaces for wins versus nominations. Screenshots inspected. This is component verification, not a fresh production-data or authenticated full-page check. No schema/migration, production write, deployment or Git mutation.


## Owner correction: horizontal tiles and visible gold

Owner rejects full-width stacked list cards and the dark award tint. Superseding layout: each list occupies65percent of its rail, with horizontal navigation/swipe and three fixed portrait cells inside a2:1 tile. Missing artwork leaves neutral cells; object-contain preserves the complete supplied posters. Reuses card-aligned rail navigation, made generic over the HTML element type for the list's semantic ul. Manage retains membership operations. Award surface is now the solid existing gold token with black readable copy; nomination-only uses80percent gold, still clearly visible. Gold ribbon retains a black backing for contrast. The actual-component browser fixture now asserts65percent tile width, same-row siblings, previous/next actions,0/1/2/3images, membership actions, no overflow/errors and distinct win/nomination backgrounds at all four widths. Source data unchanged.
