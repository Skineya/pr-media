# pr-media

Public asset bucket for screenshots, screen recordings, and other media embedded in
pull requests across the Skineya repos.

> **This repository is public.** GitHub's image proxy fetches embedded images
> anonymously, so media referenced from a PR only renders if it lives in a public
> repo. Never upload anything that can't be world-readable: no real user data, no
> production credentials, no unreleased marketing assets you don't want indexed.

## Layout

```
<repo>/<pr-number>/<name>.png
```

e.g. `ios/412/checkin-empty-state.png`, `android/87/paywall-before.png`

Recordings go in the same folder as `.mp4` (GitHub plays them inline) or `.gif`.

## Adding media

```sh
git clone git@github.com:Skineya/pr-media.git
mkdir -p ios/412
cp ~/Desktop/shot.png ios/412/checkin-empty-state.png
git add . && git commit -m "ios#412 check-in empty state" && git push
```

## Referencing from a PR

Use the `raw.githubusercontent.com` URL pinned to `main`:

```md
![Check-in empty state](https://raw.githubusercontent.com/Skineya/pr-media/main/ios/412/checkin-empty-state.png)
```

To size a screenshot (phone shots are tall — scale them down):

```html
<img src="https://raw.githubusercontent.com/Skineya/pr-media/main/ios/412/checkin-empty-state.png" width="300">
```

Side-by-side before/after:

```md
| Before | After |
|---|---|
| <img src="https://raw.githubusercontent.com/Skineya/pr-media/main/ios/412/before.png" width="280"> | <img src="https://raw.githubusercontent.com/Skineya/pr-media/main/ios/412/after.png" width="280"> |
```

## Housekeeping

Files are never rewritten in place — a PR merged months ago should still render.
If a screenshot is wrong, add a new file rather than force-pushing over the old one.
