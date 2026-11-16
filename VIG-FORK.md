# This fork

A fork of https://github.com/pbakaus/impeccable kept so that the plugin updates only when
Dean pulls an upstream change in, and so that its engine never talks to a server.

`main` mirrors upstream and is never edited. `vig` is `main` at a reviewed commit plus the
changes below, and is the branch machines install the plugin from.

## What differs from upstream (branch `vig`)

1. **The launcher sets every server address to a closed local port, on every run.**
   `skill/scripts/impeccable` and its copy `plugin/skills/impeccable/scripts/impeccable` export
   `IMPECCABLE_API_URL`, `IMPECCABLE_UPDATE_HOST` and `IMPECCABLE_ACCOUNT_URL` as
   `http://127.0.0.1:9`, and `DO_NOT_TRACK=1` and `IMPECCABLE_NO_TELEMETRY=1`. A request is
   refused on the machine and nothing leaves it. (An empty value would not do: the engine
   treats empty as unset and uses its built-in address.) Checked on 2026-11-15 with
   `strace -f -e trace=connect`: `impeccable context` made one connection attempt, to
   127.0.0.1 port 9, and none anywhere else.
2. **The launcher never downloads an engine.** It exits before upstream's download path.
3. **The Linux x64 engine is committed** at
   `plugin/skills/impeccable/scripts/bin/linux-x64/impeccable` (upstream ignores that folder).
   It is upstream's release asset `impeccable-linux-x64` for engine 0.1.24, sha256
   `d317bd1be061fdc902c4dede7fdf1efd1f7744882fb511a1f962fcdc4eed5382`, the digest
   `engine-checksums.txt` pins. Every machine Dean runs is Linux x64.
4. **The plugin declares no hooks.** `plugin/hooks/hooks.json` is empty, so nothing is scanned
   at session start, after an edit or at the end of a turn. The detector runs only inside
   design work, when the skill runs it (its context output asks for one run at the end where
   no hook is wired). Dean, 2026-11-16: "it should only run when I'm doing design work".
   Upstream's own tests of that file (`tests/hook-build.test.mjs`) are not expected to pass here.

Not changed: the Windows launcher, the copies of the launcher under the other harness
folders, and the engine's source. The engine still contains its network code; the launcher
is what keeps it from reaching anything.

## What still reaches the network

Nothing through the launcher. `impeccable login` cannot reach the account service. Asking for
a generated image with `OPENAI_API_KEY` set would still call OpenAI: that address is not one
the engine reads from the environment.

## Taking an upstream update

1. Bring `main` up to upstream, and read what changed since the reviewed commit, above all in
   `crates/` (the engine) and `skill/scripts/impeccable` (the launcher).
2. Merge `main` into `vig`. Keep the two launcher changes, and keep `plugin/hooks/hooks.json`
   empty.
3. If `ENGINE_VERSION` moved: download `impeccable-linux-x64` for the new version from
   upstream's release, check it against `engine-checksums.txt`, and commit it over the old one.
4. Run `sh plugin/skills/impeccable/scripts/impeccable engine-probe`: it must print the new
   version. Then repeat the `strace` check from item 1 above.

Reviewed commit: `2b86d9c` (plugin 4.9.1, engine 0.1.24), 2026-11-15.
