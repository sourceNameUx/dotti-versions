# dotti-versions

The released builds of the Dotti plugin, and the list the plugin reads to know
when a newer one exists.

Nothing here is source code. The addon lives in its own repository; this one only
carries the packaged zips and the manifest that points at them.

## How it is laid out

```
versiones/manifest.json          the list the plugin asks for
versiones/<version>/dotti-<version>.zip
```

`manifest.json`:

- `format`: the layout of this file. Currently `1`.
- `latest`: the version to offer. It has to appear in `versions`.
- `notes`: a line about the release, shown in the plugin when `versions[].notes`
  is missing.
- `versions`: the newest first. Each entry carries `version`, `file` (relative
  to this manifest, so `b52/dotti-b52.zip`), `sha256`, `size`, `required` and,
  optionally, `notes`.

The plugin compares `latest` against its own build number and only offers
something when it is really newer. Older entries stay in the file, which is what
makes going back possible and turns the manifest into a history.

## What the plugin does with it

Once a day, on startup, the plugin reads `manifest.json` from
`raw.githubusercontent.com` and, when there is something new, shows a card. It
does nothing until the user accepts. On accept it downloads the zip, checks its
`sha256` against the manifest, copies the whole addon into a backup and only then
unpacks over `res://addons/Dotti/`, and asks for a restart. A zip whose hash does
not match is thrown away and nothing is written.

An unreleased or unpublished version is simply not listed: there is no way for
the plugin to see something this repository does not carry.

## Publishing a version

From the addon repository, with this repository cloned somewhere:

```
godot --headless --path . --script res://scripts/empaquetar_addon.gd -- <path-to-this-repo> "What changed"
```

That writes `versiones/<DOTTI_BUILD>/dotti-<DOTTI_BUILD>.zip` and updates the
manifest with its hash and size. Then commit and push: the plugin picks it up on
its next check. Nothing is served from a branch other than `main`.
