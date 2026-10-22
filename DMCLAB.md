# DMC lab fork of aind-ephys-preprocessing

Lab-maintained preprocessing step for the AIND ephys pipeline on Arrhenius.
The capsule layout (`code/run`, `code/run_capsule.py`, `code/params.json`) is
unchanged so the stock pipeline can run it through its repo/commit override:

```
preprocessing_repo   = 'https://github.com/xiao-1011/aind-ephys-preprocessing'
preprocessing_commit = '<full commit hash>'
```

Always pin the full commit hash in the pipeline config. Tags are for people.

## Branches

| Branch | Content |
|---|---|
| `main` | Mirror of `AllenNeuralDynamics/aind-ephys-preprocessing` `main`. Never commit here. |
| `dmclab` | Our changes, as a short series of commits on top of an upstream release commit. |

No other branches are kept. Upstream development branches live upstream.

## Production tags

| Tag | Commit | Upstream base | Used by |
|---|---|---|---|
| `dmclab-1.4.0-v2` | `194532b` | `4e86c7f` (pipeline 1.4.0) | DREDGE-100 v1 (`batch_runs/20261006_180017`) and v2 (`batch_runs/20261012_125513`) |

Tags are never moved or deleted: archived runs refer to these commits.

## Changes relative to upstream

1. `14acd91` Write the DREDge motion-corrected traces to the preprocessed binary
   when motion is applied. Upstream dumped `binary_{name}.json` before applying
   motion, so the sorter read uncorrected traces.
2. `194532b` Require applied motion: fail instead of silently sorting
   uncorrected data (`raise_error` when applying, error if apply without compute
   or if no motion is estimated), and re-dump the preprocessed recording JSON so
   postprocessing/visualization use the same corrected binary as the sorter.
3. Optional seeded noise estimate for motion. `compute_motion()` detects peaks
   with `get_noise_levels()` on unseeded random chunks, so motion (and the
   sort) changed from run to run. With
   `motion_correction.noise_levels.random_slices_kwargs` set, the estimate is
   computed with that seed and cached before `compute_motion()`. Without the
   key, behaviour is unchanged. DREDGE-100 v3 uses
   `{"num_chunks_per_segment": 400, "chunk_duration": "500ms", "seed": 0}`
   (noise-chunk pilot, `maintenance/noise_chunks_3sessions_20261020_135231`).

Not yet submitted upstream. Upstream `319079d` (pipeline 1.4.1) does not contain
an equivalent fix.

## Updating

- New lab change: commit on `dmclab`, test on a pilot, then tag
  `dmclab-<pipeline version>-v<N>` and pin its hash in `nextflow_arrhenius.config`.
- New upstream release: sync `main`, create the next series with
  `git rebase --onto <upstream release commit> <old base> dmclab`, review, test, tag.
  Old tags keep the previous series reachable.
