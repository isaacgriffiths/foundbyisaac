# Found by Isaac

[foundbyisaac.com](https://foundbyisaac.com) is the links page for my secondhand-selling side business, built as a car boot sale you walk through. The pitches along the field are the links (eBay, Vinted, YouTube, TikTok, Instagram, Facebook). Keep walking and it turns into a full flipping game: haggle for stock, list it online, hire staff, buy a van, unlock new venues and prestige for permanent perks.

It replaces the usual Linktree-style list with something people actually stay on.

## What is in the game

- **Five venues** to trade at: The Sunday Field, Seaside Sunday, Night Flea Market, Indoor Antiques Fair and Estate Clearance, each with its own scenery and gate fee.
- **Haggling** with sellers who have a floor price, limited patience and flaws you can point out.
- **Selling online** with listing limits, packing mini-games and broadband upgrades that speed it all up.
- **A warehouse** you can enter, with walk-up stations for the van, laptop, packing bench, shop, index and wardrobe.
- **Upgrades and gear** (bags, torches, loupes, flat caps, steel-toe trainers) and **staff** on timers (Barry the Packer, Shazza on Listings).
- **Prestige**: cash out for contacts and start again with permanent perks and better margins.
- **Mini-games** on a shared timed harness, used for packing parcels and listing stock.
- Saves to `localStorage` under `fbi-flip-v1`, so progress survives a refresh.

Swipe to walk, tap the grass to walk there, tap a pitch or station to open it. It is designed for a phone held upright and also takes the arrow keys on desktop.

## How it is built

One file. `index.html` holds the markup, the CSS and roughly ten thousand lines of vanilla JavaScript. There is no build step, no framework and no dependency beyond two Google Fonts. The scene is 2D side-on: vertical position in world units encodes depth, the `#world` layer scales up to 2.5x on wide screens, and UI inside it counter-scales so text stays readable.

It is hosted on GitHub Pages from `main` with the `CNAME` in this repo pointing the custom domain at it. Push to `main` and the site is live once the Pages build finishes.

`crossfeed/` holds the privacy policy, terms and data-deletion pages for Crossfeed, a separate social media analytics app, served under the same domain.

`MANSION-HANDOFF.md` is a design brief for a planned late-game area that has not been built yet. It documents the depth convention, the save format, the perk system and the mini-game harness in more detail than this file.

## Making it your own

Everything that says "Isaac" is in `index.html`; there is no config file.

- **Links.** Search for `id="linksPanel"`. The list there is the whole point of the page; swap in your own shops and socials. The pitches inside the game use the same URLs, so search for `ebay.co.uk` to find every place a link is used.
- **Name and blurb.** Search for `Found by Isaac`, and update the description and Open Graph meta tags at the top of the file.
- **Avatar.** Replace `avatar.jpg`. It is the favicon, the share image and the player.
- **Domain.** Put your own domain in `CNAME`, or delete the file to serve from `<user>.github.io`.
- **Save key.** Optional, but changing `SAVE_KEY` avoids inheriting a save from another copy of the game on the same origin.
- `crossfeed/` and `MANSION-HANDOFF.md` are specific to my setup and can be deleted.

## Working on it

Serve the folder and open it in a browser:

```bash
python -m http.server 3001 --bind 127.0.0.1
```

Conventions that matter when editing:

- Syntax-check the inline script after every edit. The whole game is one `<script>`, so one stray bracket takes the site down.
- New save fields go in both `save()` and the loader, or they silently never persist.
- New entries in `items` and `PACK_POOL` go at the end of the array. Saved inventories store array indexes.
- No em dashes anywhere in the file. Comments use `-`, user-facing text uses `·`.
