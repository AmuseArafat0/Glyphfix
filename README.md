# Glyphfix

Drop a screenshot of a link. Get back the version that actually opens.

You read an address off a picture, type it out, and it fails. One character was a twin of another: `l` for `I`, `0` for `O`, `rn` for `m`. Glyphfix reads the picture, builds every plausible reading closest-first, then visits each one until something comes back alive.

One file. No build step. No server.

## What happens when you drop an image

1. **Reads it.** Tesseract runs in your browser. The picture never leaves your device.
2. **Splits it.** Recognises Drive folders and files, Docs, YouTube, or falls back to the last part of any address.
3. **Checks the length.** Drive ids are 33 characters (28 on older items), YouTube ids are 11. This matters more than anything below.
4. **Builds candidates.** Ordered by how far each strays from what you typed, so the likeliest fix is tried first.
5. **Tests them live.** Through public read-only relays, four at a time, stopping the moment one opens.
6. **Hands it back.** The working link, the folder name, and it is already on your clipboard.

Everything after step 1 also works if you skip the image and type the link yourself.

## Length is the thing to watch

Swapping characters can never fix a wrong length, so Glyphfix handles that separately:

- **One too long** (a wrapped screenshot duplicated a character at the break): tries removing each character, then swaps within each of those. The right answer usually lands in the first thirty checks.
- **One too short** (a character was lost at the break): tries inserting each of the 64 possible characters at each position. About 2,100 candidates, so switch on the Drive API key first or it will crawl.
- **Off by more than one**: nothing automated will save you. It says so and stops wasting your time. Go back to the picture and re-read the line break.

## Making it fast

Out of the box it uses public relays (r.jina.ai, allorigins, codetabs) with automatic failover when one starts erroring. No setup, but they are rate limited, so a thousand candidates takes a while.

For Drive links, add a free Google API key and it goes roughly fifty times faster:

1. console.cloud.google.com, make a project
2. APIs & Services, enable **Google Drive API**
3. Credentials, create an **API key**
4. Restrict it to your Pages address

The key stays in the page and is sent only to Google. It finds anything shared with anyone-who-has-the-link. A folder restricted to named people looks missing to the key, which is why the relays stay on alongside it and catch the "request access" page instead.

## Publish it

```bash
git init
git add .
git commit -m "Glyphfix"
git branch -M main
git remote add origin https://github.com/YOUR-NAME/glyphfix.git
git push -u origin main
```

Then **Settings → Pages → Deploy from a branch → main / (root)**. Live at `https://YOUR-NAME.github.io/glyphfix/`.

Relays need a real origin, so use the Pages address rather than opening the file off your disk.

## When nothing opens

The verdict panel tells you which case you are in.

- **Unclear results.** A relay answered with something that was neither a 404 nor a recognisable page. Open those by hand, there are usually only a few.
- **Never reached.** The candidate limit ran out. Raise it under look-alike sets.
- **All ruled out.** Widen the sets. There is a button that steps up one level and reruns.

## Look-alike sets

`l I 1 i` · `0 O o` · `O Q D 0` · `5 S s` · `2 Z z` · `8 B` · `6 G b` · `9 g q` · `7 T` · `4 A` · `3 B E` · `U V u v` · `C G` · `E F` · `n h` · `c e` · `t f` · `Y V y v` · `K X x k` · `M N W` · `- _ ~ .` · same-shape case pairs · two-letter swaps `rn`↔`m`, `vv`↔`w`, `cl`↔`d`, `ii`↔`n`

Toggle whole sets, or tap any single character in the grid and pick its twins by hand. Candidates using characters that cannot legally appear in a Google or YouTube id are dropped before anything is tested.

## License

MIT.
