# Manifest

A Ninite/Linite-style mass installer for Linux. Pick a distro, check the apps you want,
and Manifest writes one `install.sh` (or a single copy-pasteable command) that installs
everything through your native package manager, with Flatpak as a fallback for apps
that don't ship native packages.

Single file, no build step: `index.html` contains all the HTML/CSS/JS.

## Host it on GitHub Pages

1. Create a new repo (or use an existing one) and push `index.html` to it — root of the
   repo, or a `/docs` folder, your call.
2. On GitHub: **Settings → Pages**.
3. Under **Build and deployment → Source**, choose **Deploy from a branch**.
4. Pick the branch (usually `main`) and the folder (`/root` or `/docs`), then **Save**.
5. GitHub gives you a URL like `https://<username>.github.io/<repo>/` — live in a minute
   or two.

No CI, no dependencies, nothing to install locally to deploy it.

## Editing the package list

Everything lives in two arrays near the top of the `<script>` block:

- `ESSENTIALS` — the pre-checked "baseline" items (drivers, hw-probe, timeshift,
  build tools, CPU microcode, flatpak setup, plus desktop-environment extras like
  `kio-admin` for KDE).
- `APPS` — the optional catalog, grouped by category (`browsers`, `comms`, `media`,
  `creative`, `dev`, `gaming`, `productivity`, `utilities`).

Each entry looks like:

```js
{id:'vlc', n:'VLC', d:'Plays absolutely everything.', c:'media',
 pkg:{apt:'vlc', dnf:'vlc', pac:'vlc', zyp:'vlc'}}
```

- `pkg` — package name per manager. Omit a manager if there's no native package.
- `fp` — a Flatpak app ID, used automatically when no native package exists for the
  selected manager (or as the only option for apps like Discord/Spotify/Slack that
  don't ship native repo packages at all).
- `special` — for items that need a whole custom command instead of a simple
  `install <name>` (e.g. Fedora's dev-tools group, the Arch `yay` bootstrap, Ubuntu's
  `ubuntu-drivers autoinstall`). A `special` value starting with `#` is emitted as a
  comment only, never executed — used for packages that need manual/AUR install.
- `de` — restricts an `ESSENTIALS` item to one desktop environment (`gnome`, `kde`,
  `xfce`).
- `onlyManager` — restricts an `ESSENTIALS` item to a single package manager.

To add a new app: add an object to `APPS` with a unique `id`. It'll show up under its
category automatically, and disappears from view for any manager where it has neither
`pkg[manager]` nor `fp`.

## Known limitations (same as Linite/Ninite have)

- Package names are best-effort and can drift as distros rename or move packages
  between repos — worth spot-checking against current docs before relying on this
  for a fleet of machines.
- Manifest doesn't enable extra repos for you (RPM Fusion on Fedora, `multiverse` on
  Ubuntu, Packman on openSUSE, an AUR helper on Arch). Several apps (Steam, VLC's
  extra codecs, some gaming packages) need one of those enabled first — the generated
  script header calls this out, but doesn't do it automatically on purpose, since
  that usually means editing repo config, not just installing a package.
- The "Download install.sh" script runs `sudo` commands. Treat it like any script off
  the internet — read it before running it (`chmod +x install.sh && ./install.sh`).
