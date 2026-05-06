# evennia-world-builder-test-yaml

Tiny synthetic content repo used by [`evennia-world-builder`](https://github.com/FullCircleMUD/evennia-world-builder) for live integration smokes against `wb_build`, `wb-validate`, and the `GitHubReader` / `LocalReader` plumbing. Held outside the library's own test suite (which uses pure in-memory fixtures) so the discovery + loading + validation pipeline gets exercised end-to-end against a real backing store.

This repo is **fixture data**, not example usage. The shape and contents are deliberately minimal — just enough to walk the manifest tree across both folder-and-file kinds and across multiple zones.

## Layout

```
definitions.yaml         # consumer's level vocabulary + gating assertion
index.yaml               # root-level entries
millholm/                # zone (folder kind)
  index.yaml             # millholm's children
  inn.yaml               # room (file kind)
  bakery.yaml            # room (file kind)
aethenveil.yaml          # zone (file kind, no nested rooms)
hello.yaml               # legacy spike-1 fixture; not part of the indexed tree
```

The manifest declares `levels: [zone, room]` — two hierarchical levels. The zones intentionally split across kinds: `millholm` is a folder containing rooms; `aethenveil` is a single file at zone level (no rooms underneath). Together they cover the path-inference branches in the Loader.

## Conventions exercised

- **Index-driven discovery.** `index.yaml` files are the source of truth for what's part of the tree. Orphan files (`hello.yaml`) are deliberately ignored by walks.
- **Path inference.** `kind: folder` named `millholm` ⇒ `millholm/index.yaml`. `kind: file` named `aethenveil` ⇒ `aethenveil.yaml`.
- **Deployment identity.** Every leaf file declares `deployment_id` (a non-negative integer, unique within its file). Cross-references between entities — when they're added — will use `(deployment_file, deployment_id)` per the library's identity contract.
- **Validation gating assertion.** `definitions.yaml` carries `repo-ci-pre-validation: false`. The library defaults this to `false` so consumers explicitly opt in to "trust the gate" only after standing up CI on their content repo.

## Using it

Against the live private repo (requires PAT):

```bash
wb-validate --reader=github --repo=FullCircleMUD/evennia-world-builder-test-yaml --ref=main --pat="$WB_PAT"
```

Against a local clone (no auth):

```bash
git clone https://github.com/FullCircleMUD/evennia-world-builder-test-yaml.git
wb-validate --reader=local --root=./evennia-world-builder-test-yaml
```

Either should produce a clean run with three loaded entities (the inn, the bakery, the aethenveil sanctum) and a passing validator.

## Editing

Treat changes the same as any content repo: open a PR, let CI's `wb-validate` job gate the merge, only land green commits on `main`. Branch protection on `main` is the reason `wb_build` consumers can later flip `repo-ci-pre-validation: true` and skip pre-validation at deploy time without losing safety. See [DESIGN/validation-gating.md](https://github.com/FullCircleMUD/evennia-world-builder/blob/main/DESIGN/validation-gating.md) in the library repo for the full model.
