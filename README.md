# WH Rules data

Each bundle is a complete copy of the declarative rules snapshot for one rules family.
The standard bundle covers matched play by default. WCW 2026 contains the changes
for that event, already applied. Use one bundle for all data needed by a roster or game.

Start with [catalog.json](catalog.json). To follow the current standard bundle, read
`families.standard.currentBySchema["1"]` and find that tag in the family's `releases`.
For a tournament, start with its own family. For reproducible results, store the exact
tag and its `rulesetRef` with the roster or game. Use the release's manifest path for
the checked-out files or download its ZIP at `archive.url` and verify the published
SHA-256 before use. Local exports keep their ZIP under `archive.path`.
Dates in `effectiveFrom` and `effectiveTo` describe the source rules. Unknown dates
are null. The release's `publishedAt` records when this export became available.

A tag such as `wcw-2026-r1-s1-e1` identifies rules revision 1, schema 1, export revision 1.
A new accepted rules snapshot increments the family's rules revision. A packaging
correction increments its export revision. An incompatible JSON contract increments the
schema version. The catalog revision tracks changes to the catalog and its pointers.

## Files

The [manifest schema](schemas/manifest.json) describes each snapshot's file inventory.
The [catalog schema](schemas/catalog.json) describes the index. File keys use the logical
paths from WH Rules, with `src/` removed. In the manifest, `files[key].path` is relative
to the bundle directory. `sha256` checks the exact distributed bytes. `sourceSha256`
checks the corresponding canonical JSON or the original bytes of an image.

| Old path under src/data | Path in a bundle |
| --- | --- |
| `basicRules.js`, `advancedRules.js`, `reference.js` | `data/basicRules.json`, `data/advancedRules.json`, `data/reference.json` |
| `factions/<slug>.js`, `factions/ru/<slug>.js` | `data/factions/<slug>.json`, `data/factions/ru/<slug>.json` |
| `datasheets/<slug>.js`, `datasheets/ru/<slug>.js` | `data/datasheets/<slug>.json`, `data/datasheets/ru/<slug>.json` |
| `roster/<slug>.js`, `ledBy/<slug>.js` | `data/roster/<slug>.json`, `data/ledBy/<slug>.json` |
| `mfm/<file>.js` | `data/mfm/<file>.json` |
| `missions.js`, `missionsRu.js` | `data/missions.json`, `data/missionsRu.json` |

JSON objects derived from JS modules contain their original declarative exports.
Read a default export at `payload.default`, and named exports by name.
JSON source files retain their original shapes. Faction extensions, borrowing,
wargear references and RU overlays retain their source relationships within the bundle.
Read the English base and the matching RU overlay where present. The snapshot does
not generate translations for fields that had none. Use `datasheetIds.json`,
`sourceIds.json` and `datasheetRenames.json` for source identity and history. Positional
wargear indices belong to their specific snapshot. Proxy designations are in the
snapshot's Legends data where present. Within this schema, game content and the
records available in a module can change between rules revisions.

`snapshots/` contains source roots and provenance. Rules diagrams and other snapshot
images use their original paths under `images/`. The manifest lists the two app QR
images omitted from the export. The bundle includes all declarative snapshot data.
Consumers implement their own rules processing, including any behaviors represented
by code in the WH Rules engine. Raw GW downloads and the application are outside the
data contract.

## Reading the current or a pinned bundle

```python
import hashlib
import json
from pathlib import Path

root = Path(".")
catalog = json.loads((root / "catalog.json").read_bytes())
family = catalog["families"]["standard"]
tag = family["currentBySchema"]["1"]
# To reproduce an earlier roster or game, load its saved tag here.
release = next(entry for entry in family["releases"] if entry["tag"] == tag)
manifest_path = root / release["manifest"]["path"]
manifest_bytes = manifest_path.read_bytes()
assert hashlib.sha256(manifest_bytes).hexdigest() == release["manifest"]["sha256"]
manifest = json.loads(manifest_bytes)
record = manifest["files"]["data/datasheets/space-marines.js"]
payload = (manifest_path.parent / record["path"]).read_bytes()
assert hashlib.sha256(payload).hexdigest() == record["sha256"]
datasheets = json.loads(payload)["default"]
```
