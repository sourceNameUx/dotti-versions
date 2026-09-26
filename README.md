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
  to this manifest, so `1.0/dotti-1.0.zip`), `sha256`, `size`, `required` and,
  optionally, `notes`.

## One served version at a time

`latest` is not a suggestion. The gateway reads this file every minute and
refuses every model request whose `x-dotti-build` header is not `latest`, so a
build that is not the one published here does not work at all: the plugin shows
the update card and the chat stops until it is installed.

That is why every entry is written with `required: true`, and why the plugin does
not offer to postpone it. A version left in this file as `required: false` would
be one that cuts the chat without having said so first.

It also means the order below is not optional: the build has to be here, working,
before the gateway starts refusing the ones below it.

## Numbering

`MAYOR.MENOR`, two numbers and a dot: `1.0`, `1.1`, `2.0`. Never letters again.
The letter scheme (`b52`, `b55`) is finished and is only understood so as not to
break anyone still carrying it.

One consequence worth knowing before blaming the updater: a `b` version is never
newer than a dotted one, however large its number. A build stuck on `b55` will
say it is up to date and will never see `1.0`, because the comparison runs inside
the copy that is already installed and cannot be changed from here. Anyone on a
`b` build reinstalls by hand once.

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

The packager refuses to write anything if `secrets-src/` is missing or if one of
the sealed texts would travel in the clear, so a mistake here stops the release
instead of leaking it.
