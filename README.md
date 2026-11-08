# blackwell-isa

Public research artifacts for the NVIDIA **Blackwell** SASS instruction set.

This repository publishes machine-readable ISA databases for Blackwell GPUs:
consumer SM120 (RTX 50-series, including RTX 5090 / GB202), datacenter SM103a
(B300 SXM6), and SM121a (DGX Spark / GB10, Grace-Blackwell unified). It is intended for researchers and
implementers building assemblers, disassemblers, compiler backends, binary analysis
tools, and instruction schedulers for Blackwell-class targets.

Artifacts:

- [`sm120.json`](sm120.json) — canonical machine-readable ISA database
- [`sm103a.json`](sm103a.json) — canonical SM103a ("B300", Blackwell datacenter) ISA database; same schema, validated bit-exact against an SM103a corpus (see "SM103a notes" below); refreshed from the production cubit table including the tcgen05 class set (`UTCHMMA`/`UTCQMMA`/`UTCIMMA`/`UTCOMMA`, `LDTM`/`STTM`, `UTCCP`, `UTCSHIFT`, `UTCBAR`, `UTCATOMSWS`, `UVIRTCOUNT`)
- [`sm100a.json`](sm100a.json) — SM100a ("B200", Blackwell datacenter) ISA database derived from the SM103a canonical revision (encoding layer byte-identical on all 119 shared tcgen05 probe pairs) plus the tcgen05 acceptance deltas; offline-validated by decoder/encoder replay on an archived 20.0M-instruction sm_100 corpus (99.9219% EXACT-minus-sched on 1,134 cubins); scheduling/pipeline sections inherited from SM103a, unvalidated for B200 timing
- [`sm121a.json`](sm121a.json) — canonical SM121a ("GB10", DGX Spark Grace-Blackwell) ISA database; same schema, delta-driven vs SM120 with EXACT gates per entry; 23 833 encoding keys at this revision. Counting note: the SM121a table keeps a **finer per-key granularity** — its `instructions` dict holds one key per encoding row, so raw key counts are not comparable with the canonical instruction-*form* counts of sm120.json/sm103a.json
- [`SM120_ISA_REFERENCE.html`](https://kacper-daftcode.github.io/blackwell-isa/SM120_ISA_REFERENCE.html) — generated, searchable HTML reference ([browse online](https://kacper-daftcode.github.io/blackwell-isa/SM120_ISA_REFERENCE.html))
- [`SM103A_ISA_REFERENCE.html`](https://kacper-daftcode.github.io/blackwell-isa/SM103A_ISA_REFERENCE.html) — generated, searchable HTML reference for SM103a ([browse online](https://kacper-daftcode.github.io/blackwell-isa/SM103A_ISA_REFERENCE.html))
- [`SM100A_ISA_REFERENCE.html`](https://kacper-daftcode.github.io/blackwell-isa/SM100A_ISA_REFERENCE.html) — generated, searchable HTML reference for SM100a ([browse online](https://kacper-daftcode.github.io/blackwell-isa/SM100A_ISA_REFERENCE.html))
- [`SM121A_ISA_REFERENCE.html`](https://kacper-daftcode.github.io/blackwell-isa/SM121A_ISA_REFERENCE.html) — generated, searchable HTML reference for SM121a ([browse online](https://kacper-daftcode.github.io/blackwell-isa/SM121A_ISA_REFERENCE.html))

## Artifact Scope

This is a **data release**, not a compiler or assembler. It contains:

- 128-bit SASS encoding templates and variable bit-field mappings
- operand extraction rules for registers, predicates, immediates, guards, and modifiers
- instruction form metadata (`base_op`, operand signature, modifier groups)
- scheduling metadata: pipeline class, latency, throughput, stall defaults, control-word class
- pipeline/resource configuration used by an SM120 scheduler
- a generated HTML reference for manual inspection

It does not contain the reverse-engineering pipeline, probe programs, proprietary cubin
corpora, driver dumps, firmware notes, or private working material. Those are
intentionally excluded from this public artifact.

## SM103a notes

SM103a (`sm103a.json`) targets the B300-class Blackwell parts (ELF `e_flags` SM field `0x67`,
distinct from SM120 `0x78` and SM100 `0x64`). Empirical hardware finding: `CS2R Rd, SRZ`
clears the even-aligned 64-bit register pair `Rd:Rd+1`, not only `Rd` (cuda-gdb-proven;
binary patchers must not substitute 32-bit zero-idioms blindly — consumers of `Rd^1`
observe the side effect).

Measured tcgen05 facts encoded in this release:

- For all 119 probe pairs shared with SM100a in the tcgen05 corpus, the 128-bit
  instruction words are byte-identical between sm_100a and sm_103a; the encoding layer
  transfers 1:1 to SM100a.
- ptxas 13.3 accepts `tcgen05.mma` `kind::i8` on sm_100a but rejects it on sm_103a.
- `tcgen05.ld.red` (`LDTM.STAT`) is accepted on sm_103a only.
- `tcgen05.wait::alloc/dealloc` and `tcgen05.commit complete_tx::bytes` are rejected by
  ptxas 13.3 on both targets; TMEM allocation runs through `UTCATOMSWS`/`UTCBAR`
  arbitration and `UVIRTCOUNT.DEALLOC.SMPOOL` pool release as encoded here.

Coverage note: SM103a encoding records stand on a 1.94M-sample nvdisasm-pair corpus plus
the tcgen05 PTX probe corpus. Per-instruction scheduling metadata is thinner than in
`sm120.json` (baseline defaults except where hardware-probed); consult `ctrl_classes`
for control-word behavior, which is documented per class.

## Validation Status

What can be reproduced today, with the command that produces the number (all on the host that
builds [`cubit`](https://github.com/kacper-daftcode/cubit); nvdisasm 13.1 is the decode oracle):

- **SM120 strict re-encode census — gated on every table revision pushed since 2026-11-02, last run 2026-11-08 at this revision (cubit `main`):
  1 811 600 / 1 811 600 instructions over 169 cubins** — every word decodes, prints, re-parses and
  re-encodes to the identical 128 bits (`cubit roundtrip-strict`). The corpus is the production
  kernel set of the vLLM-Moet DeepSeek-V4.1 / GLM-5.3 SM120 deployment: FlashInfer 0.7.2
  sparse-MLA AOT + JIT caches (544 064 + 544 200 words), FlashInfer MXFP8 GEMMs (16 016),
  FlashInfer CAKE DSV4.1 mixed-cache kernels (599 048), b12x MoE kernels (52 024), DeepGEMM
  SM120 FP4/FP8 kernels (56 248). Before this revision the same census stood at
  1 797 483 / 1 811 600 (99.2 %) with five forms missing (see CHANGELOG 2026-11-02).
- **Whole-cubin reassembly (gated on every push; last run 2026-11-08 at this revision):** `cubit disassemble
  --frozen` → `cubit asm -T` reproduces all 157 census cubins that contain code byte for byte
  (`cubit tools/asm_roundtrip.py`); the census above is identical per cubin at this revision.
- **Per-form measurements:** every row with a `_src` tag of the form `<form>-sm120-<date>` has an
  nvdisasm fixture and a cubit test that replays it (`_meta.provenance.sources` names them).

Historical figures (kept as history; the corpora and hosts behind them are not available for a
re-run from the build host — they are **not** re-verified at this revision):

- 2026-02/03: 47 244 instructions decoded across 178 cubins with 100 % decode coverage; 5 014 / 5 014
  roundtrip fuzz cases; 936 / 936 targeted ground-truth instructions round-tripped including the
  25-type `QMMA.SF` ptxas corpus and `LDG.E.LTC128B.128`; all 36 dense `QMMA.SF...E8` type pairs
  (25 emitted by ptxas, 11 with the undocumented `E3M4` executed on RTX PRO 6000); selected semantic
  bit fields validated by hardware patch tests on RTX 5090 and RTX PRO 6000.
- 2026-08-25 (O2 consolidation): SM120 roundtrip over a 240-file corpus (1 076 075 instructions) at
  99.86 %.
- SM103a (as of 2026-09-18): disassemble→assemble byte-exact on 323 / 323 cubins of the publish
  corpus (tcgen05 probe kernels, FA4 kernels, instruction samples). **Re-run 2026-11-03 on the B300
  host `cold-face-bounces-fin-02`** (cubit 0fdd91a, `sm103a.json` / `sm100a.json` at 77a3dfa): the
  publish corpus holds 328 cubins today — 177 / 177 SM103a and 151 / 151 SM100a cubins byte-identical
  through `cubit disassemble --frozen` → `asm -T` (104 + 28 words carried as `__raw__`); the strict
  semantic round trip (no raw words) stood at 85 646 / 85 792 words and **stands at 85 656 / 85 792
  since the LDTM / STTM lane rebuild (third pass at cubit `52767f9`, 2026-11-06: 77 548 / 77 656 SM103a,
  8 108 / 8 136 SM100a, 240 / 328 cubins strict-exact)**, the refused words being fail-closed refusals of
  silicon-unsafe forms (BUG-080 guarded ATOMS, BUG-004) plus 3 re-encode residues of text-inert bits
  (`UMOV URd, 0x0` with a reuse bit ×2, `I2F.S16` bit 74; cubit `docs/KNOWN_ISSUES.md` V1, V5).
- SM100a (as of 2026-08-26): decoder/encoder replay over an archived sm_100 corpus (1 134 cubins,
  20 024 512 instructions, cuBLAS/cuBLASLt/cuSOLVER/cuSPARSE/curand + probes): 99.92 % EXACT-minus-sched.
  **Re-run 2026-11-03 on `cold-face-bounces-fin-02` with the archive's own tool: 99.924 %**
  (20 009 329 / 20 024 512, gate 99.90 PASS).
- SM121a (as of 2026-09-18): differential legality scan of 260 352 probes vs SM120 with 0 differences;
  `nvdisasm -b SM120` vs `-b SM121a` over 648 992 raw instructions with 0 decode differences; census
  over a 92 098-word GB10 corpus with 0 failures. **Re-run 2026-11-04 on the GB10 host `gx4`**
  (NVIDIA GB10, the 2026-09 census script over the same corpus, cubit 6392786 / `sm121a.json` at
  6619782): 91 927 / 91 927 unique words decode (0 failures), text parity against the recorded
  nvdisasm texts 91 554 / 91 927 = 99.59 % — 373 differences in 17 keys. Batches 2 (`15caf70`:
  `F2F` type order 217, `LDL.64` 95, `MUFU`, IMAD alias) and the T-4 parity pass (`R2P` `PR`, `BAR`
  second immediate, `IMNMX`/`UIMNMX` `!PT`, `SEL -0x1`, `REDUX`, `DEPBAR SB0`, `ATOMS [R+UR]`, `F2IP`
  order, `IMMA .ROW/.COL`) closed all of them: **372 / 373 of the recorded differences now match**,
  the one residue being `LEPC` (a PC-relative target the isolated-word census renders at address 0 —
  byte-identical to the vendor in a full disassembly). Decode coverage stays 100 % (0 failures over
  91 927 words). See cubit `docs/KNOWN_ISSUES.md` V3 for the re-run at this revision.
  **Regression found and closed 2026-11-08 (session T-20, `75e9d20`):** the census re-run locally on the
  91 927 unique words (nvdisasm 13.0 texts; cubit's `audit_words`) at the 2026-11-06..08 tables showed
  **564 words without a decode** — every signed FSETP word of the corpus (`FSETP.GT.AND P0, PT, R9, -R4, PT`,
  `FSETP.GEU.AND P1, PT, |R5|, …`): the cubit decoder arm of 2026-11-06 (FSETP off the sign-tolerant match,
  every table) had left this table's harvest FSETP rows — no sign fields — without a route for them. Closed by
  measurement, not by a tolerance: FSETP / DSETP rebuilt on `-b SM121` (see "Known issues"); 91 927 / 91 927
  decode again, text parity 91 926 / 91 927 (the `LEPC` residue), and the strict re-encode of the corpus words is
  91 887 / 91 927 (the 40 words outside the family — `FFMA_R_R_R_II`, `UIADD3` `64`, `HMMA.1688` — are the
  next GB10 batches' items).

Encoding evidence is not semantic evidence: a row that round-trips tells you how nvdisasm names a
word, not what the hardware does with it or how long it takes.

## Data Model

At a high level:

```text
sm120.json
├── _meta
│   ├── architecture, codename, gpu, instruction_width
│   ├── stats
│   ├── pipe_classes
│   ├── ctrl_classes / ctrl_epochs
│   ├── provenance       # legend, sources (every _src tag), history, errata
│   ├── key_aliases      # retired key name -> current key (2026-11-04)
│   └── hardware_config
│
├── instructions
│   └── INSKEY
│       ├── base_op          # opcode family, e.g. IADD3, QMMA, LDG
│       ├── operand_sig      # operand signature, e.g. R_P_P_R_R_R
│       ├── mod_groups       # encoding variants
│       ├── scheduling       # pipeline / latency / throughput metadata
│       ├── ctrl_class       # scheduler control-word class
│       ├── mercury          # recovered NVIDIA pattern metadata, when known
│       └── _discovery       # provenance for recently added or unusual entries
│
├── sched_only               # scheduling entries without full encoding records
├── pipeline_config          # resource defaults, latency tables, per-opcode records
├── operand_roles            # operand read/write roles per base_op (liveness passes)
└── stallfix                 # measured stall floors for cubit's `stallfix` pass
                             #   sm103a.json: v2 (B300, 2026-08-22); sm120.json: v4 (RTX PRO 6000, 2026-11-03/05)
```

A typical encoding entry (one line in the file; shown indented here):

```json
{
  "and_base": "0x0000000000000000000000000000082e",
  "variable_mask": "0x1c00000000000000000000000000f000",
  "fields": [
    { "shift": 12, "bits": 4, "token_idx": 0, "extraction": "guard" },
    { "shift": 16, "bits": 8, "token_idx": 1, "extraction": "reg" }
  ],
  "count": null,
  "_src": "prmt-ur-sm120-2026-11-02"
}
```

`and_base` is the constant part of the 128-bit instruction word. `variable_mask`
identifies operand-dependent bits. `fields` tells an assembler/disassembler where to
read or write each operand field (`token_idx` is the operand position, 0 = the guard).

**File layout (since 2026-11-08).** The tables are compact JSON with **one encoding row per line**:
`instructions.<key>` opens with the key's own attributes (`base_op`, `operand_sig`, `scheduling`, `_src`,
`_note`, …) and ends with `mod_groups`, whose every row is one line; `_meta` and the small sections
(`sched_only`, `pipeline_config`, `operand_roles`, `stallfix`, `cost_model`) are indented. A diff of a
table therefore reads row by row. The layout is produced and checked by cubit's `tools/tablefmt.py`
(`--check` / `--rewrite`; `tools/sync_table.py --validate-only` refuses any other layout). Before
2026-11-08 the files were `json.dumps(indent=1)` with the measurement note repeated on every row of a
lattice: `sm121a.json` had reached 87 MiB (GitHub refuses files over 100 MiB); the rewrite took the
four tables to 28.0 / 23.7 / 23.6 / 32.8 MiB (sm120 / sm103a / sm100a / sm121a) without changing a row.

### Provenance

- `_src` on a row names the measurement that produced or last repaired it; `_meta.provenance.sources`
  lists every tag of the table with its kind (nvdisasm field probe, encoder fix wave, native capture),
  date, and — where they exist — the cubit tool and test that hold the evidence. `_note` is the
  measurement in prose. **Provenance is written once per level (2026-11-08):** `instructions.<key>._src`
  is set when every row of the key carries the same tag (the rows keep their own `_src`; a row without
  one inherits the key's); a lattice family's law is written once in `_meta.provenance.sources[<tag>].note`
  when every row of the tag shares it, on `instructions.<key>._note` when every row of the key shares
  it, and on the row only where the row's own note differs — read a row's note as *row `_note`, else
  key `_note`, else `sources[tag].note`* (cubit `tools/tablefmt.py`: `row_src()` / `row_note()`; the
  HTML references render the key's tag and note once above its rows). A field carries its own `_src`
  only when it was measured by another tag than its row. Rows **without** `_src` (7 of 38 838 SM120 variants at this revision — the documented owner items of cubit's KNOWN_ISSUES T1; 111 of 38 866 at `7f12b68`, 143 of 38 898 at `fd736a7`, 304 of 38 966 at `37d030f`, 18 841 of 26 598 at `c896e8f`; the T1 production line of 2026-11-06/07 measures them family by family: HSETP2, ISETP / UISETP, FSETP / DSETP, PLOP3 / P2R / R2P, UFSETP, the FP32 ALU and its uniform twin, the F2I / I2F / FRND / MUFU conversions, FSET / FSEL / FCHK, then the 256-bit global load and store and the global reduction's uniform-indexed address forms — LDG.256 `dARI` / `ARURI64` / `ARURI`, STG.256, REDG `ARURI` / `ARURI64`, then the whole long tail row by row (the tensor cores, then half-precision / integer ALU / uniform datapath / control / memory / atomics — batches 15–16, then T-18's re-pin of the 2026-08/09 bug-wave suites to the measured doctrine, then T-19's field-narrowing pass over the last 111 rows), 31 213 measured rows + 6 design rows marked `modelled`, 14 918 harvest keys retired as `_meta.key_aliases`) are
  the 2026-06..08 *harvest* — fields learned from compiler output and validated by the round-trip
  census, not by per-field probes. The **wave-11 lattice rows** (the lane merge `e9b0808` of 2026-11-05 — the modifier
  lattices of `ATOM_P_R_dARI_R` / `ATOMG_P_R_dARI_R`, 46 → 2 886 groups each) carry their tags since 2026-11-06
  (`wave11-atom-lattice-2026-11-05` / `wave11-integration-2026-11-05`, 5,736 SM120 rows); their measurement record is the
  lane's own report (see "Known issues"). The harvest rows' known defect classes and the audit of
  them are tracked in cubit's `docs/KNOWN_ISSUES.md`.
- `_meta.provenance.history` keeps the fix-wave notes (formerly loose `_meta.bugNNN` keys) verbatim;
  `_meta.provenance.errata` the former top-level `_errata_*` notes of `sm121a.json`.
- `count` is the number of harvest samples a row was learned from; `null` on measured rows.
- `_meta.key_aliases` (2026-11-04) maps retired key names to the key that carries the rows today —
  the 20 SM120 keys (10 in SM121a) whose mnemonic carried a glued predicate token
  (`QMMA.16816.F16.E4M3.E4M3_P0_R_R_R_R` → `QMMA.16816.F16.E4M3.E4M3_R_R_R_R`, `F2IP.U8.F32_P2_…`,
  `DMMA.8x8x4_P5_…`, `S2R_P0_R_L`, `MOV_P0_R_II`, …). nvdisasm names their words with the plain
  mnemonic; cubit resolves an alias for lookups and old text, never prints it. Validated by
  `sync_table.py` (every alias names an existing key, none shadows one).
- **Undocumented forms.** Rows whose words nvdisasm (13.1, 13.3) cannot name on SM120 but the silicon
  executes carry a `_src` whose `sources` entry says `nvdisasm_sm120: "no text"` and cites the
  measurement: `undocumented-e3m4-2026-03` (22 `QMMA.SF[.SP]` rows with type code 2 = E3M4; the
  11 dense combinations executed on RTX PRO 6000 Blackwell 2026-08-09, 11/11 legal, 3/3 numerical
  anchors exact), `undocumented-utc-sm120-2026-03` (`UTCHMMA/UTCIMMA/UTCIMMA.KIND2/UTCQMMA.1CTA`
  executed on an RTX 5090 2026-03-28 although ptxas refuses `tcgen05.mma` for sm_120; `UTCATOMSWS`),
  `bug234-2026-08-28` (FFMA2). They stay on purpose: cubit's dynamic audit classifies their words
  as `vendor_unknown`, not as defects.
- **Baked control bits.** 880 of the 1 145 SM120 variants that carry non-zero `and_base` bits in
  [127:105] (the control code: stall, yield, barriers, wait mask, reuse) inherited them from the
  compiler sample they were learned from. Decoders strip the control code before matching and
  assemblers overwrite it with the authored or scheduled control, so these bits are a provenance
  artefact, not part of the encoding identity; cubit's `tools/sync_table.py` keeps a ratchet on
  their number so that no new row bakes a control template unnoticed.
- The schema is validated by `cubit/tools/sync_table.py --validate-only` (structure, extraction
  vocabulary, provenance convention, control-bit ratchet) and `blackwell-isa-internal/tools/validate.py`.

## Minimal Usage Example

```python
import json

db = json.load(open("sm120.json"))
insn = db["instructions"]["IADD3_R_P_P_R_R_R"]
variant = insn["mod_groups"][""]

print(insn["base_op"])                 # IADD3
print(variant["and_base"])             # 128-bit instruction template
print(variant["fields"][0])            # one encoded field description
print(insn["scheduling"]["pipe_name"]) # INT_ARITH
```

For an end-to-end assembler/disassembler using this data, see
[`cubit`](https://github.com/kacper-daftcode/cubit).

## Selected Research Results

These are included to orient readers; the primary contribution is the data itself.

- **SM120 is not SM100.** SM120 uses register-accumulator QMMA/HMMA/IMMA/DMMA forms,
  while SM100 exposes the TMEM-based `tcgen05.mma` model. Treating consumer Blackwell
  as a small B200 gives wrong code-generation assumptions.

- **Block-scaled MMA is encoded in SASS.** `QMMA.SF`, `QMMA.SF.SP`, and `OMMA.SF`
  forms are present. Current `compute_120f` PTX exposes the documented
  `mxf8f6f4` type pairs; the database additionally records undocumented `E3M4`
  and hidden/sparse forms outside the normal compiler path.

- **There is an undocumented FP8-like type code.** Type code 2 in the block-scaled
  MMA type field corresponds to `E3M4`; it appears in dense and sparse instruction
  forms and is accepted by hardware.

- **Scheduling metadata matters.** Several instruction families are not assigned to
  the pipeline one would infer from the mnemonic alone. Schedulers should consume the
  `scheduling` and `ctrl_class` fields together with their provenance and confidence.
  Compiler pipeline IDs do not by themselves establish separate physical execution
  resources or concurrent instruction throughput.

- **Control-word handling is instruction-class dependent.** The upper control bits are
  not a uniform free field. `ctrl_class` and `ctrl_epochs` describe which bits are fixed
  discriminators and which bits a scheduler can safely modify.

- **The public memory hierarchy numbers are easy to misread.** The database records
  RTX 5090 L2 as 96 MB; 128 MB is the access-policy window, not the cache size.

## Known issues

Everything below is known and listed rather than hidden; the detailed list with evidence and what
each item needs is cubit's [`docs/KNOWN_ISSUES.md`](https://github.com/kacper-daftcode/cubit/blob/main/docs/KNOWN_ISSUES.md).

- **SM120 at `27b1272` (2026-11-08, sessions T-19 / T-20): 7 of 38 838 encoding variants carry no `_src`, and every one of the 7 is a documented owner item** — the alias-able groups of the four mixed keys `F2FP_R_R` (2), `FLO_R_UR`, `WARPSYNC` (2) and `WARPSYNC_R` (1), which a group-level alias would retire once the schema has one, and `WARPSYNC.COLLECTIVE_R_II`, whose 50-bit immediate needs an extraction the schema lacks (cubit's `docs/KNOWN_ISSUES.md` T1 and `T/NEEDS_OWNER.md` 55). Six design rows carry `_src: "modelled"` (the IADD3.X encode / decode pair, `HFMA2_R_R_R_II_FI`'s plain group, `UBLKPF.L2_AUR_UR`, the `encode_only` retention rows `REDUX_UR_R` `AND` and `UISETP_UP_UP_UR_UR_UP` ``; each note names the cubit test that pins it). The other three tables are still harvest (sm103a 15,864 / 22,445 without `_src`, sm100a 15,864 / 22,439, sm121a 25,696 / 26,421). The history of the SM120 line: 18,845 of 26,580 SM120 encoding variants carried no `_src` on 2026-11-02 (sm103a 15,864 / 22,420,
  sm100a 15,864 / 22,420, sm121a 25,427 / 26,595 — the FSETP / DSETP rows of sm121a and the REDG uniform-indexed forms of sm103a / sm100a measured on 2026-11-08, see below: sm103a / sm100a are 1 584 / 32 916 and 1 584 / 32 910 at `39d0372` after the LDG.256 / STG.256 batch of the same day, sm121a 10 000 / 36 884 and sm103a / sm100a 1 280 / 33 532 and 1 280 / 33 526 at `0525e95` after the ISETP / FSETP / PLOP3 batch): these are the harvest rows of 2026-06..08
  (fields learned from compiler output, validated by the round-trip census, not by per-field probes). Every
  row added since carries a tag — the 5 736 / 5 748 / 5 744 / 44 wave-11 rows were stamped on 2026-11-06
  (`wave11-atom-lattice-2026-11-05`: the BUG-509 ATOM/ATOMG [63:60] lattice with its silicon-confirmed
  claim_forbid; `wave11-integration-2026-11-05`: the lane-lineage rows of the 3-way merge; the stamp tool is
  coldface's `stamp_provenance.py`, run with the `rows` recount corrected). A static audit (cubit
  `tools/audit_table.py`, 2026-11-02, pre-wave-11 numbers) flags 1 223 fields that address an operand the key
  does not have, 994 register fields off the standard slots, 257 immediate-bearing keys without an
  immediate field and 14 634 pairs of rows whose constant bits do not separate them (the decoder's
  tiebreak decides). The census proves these rows on the words real kernels use, not on every value of
  every field. The dynamic nvdisasm cross-check (cubit `tools/audit_dynamic.py`, 88 388 probe words)
  **is at its end state: 0 rows with a finding (2026-11-06, replay at `39bafbb`; 1 row at the T-11 tables — the
  `IMMA.SF.16832.U8.U8` hypothesis, since measured on silicon and renamed `QMMA.SF.16832.F32.MXINT8.MXINT8.E8`, see
  below)** — from 1 977 on 2026-11-02, 418 on the T-7 tables, 230 (T-8), 215 (T-9), 14 (T-10), 1 (T-11). The 13 closed on
  2026-11-06 (T-11, each re-measured with nvdisasm 13.1 on the four targets, cubit `tests/sm120_audit_t11.rs`): the
  seven RZ-base 64-bit descriptor words print the vendor's `[???255.64+UR..]` glyph as the REDG / ATOMG `.EL` twins do
  (BUG-461 doctrine: text-faithful decode, the encoder refuses the glyph), ULEA's `-URa` bit and its elided `UPT`
  operand, ULOP3's source-predicate fields on the dest-less row (the elided `UPT` destination), `BMOV.64 ATEXIT_PC`,
  `DEPBAR` SB6 / SB7 claim-forbidden, the SAT-less `FADD.SAT_R_R_R` row retired, `USETMAXREG` off cubit's
  sign-tolerant fallback (what closes the last one: cubit `docs/KNOWN_ISSUES.md` T1). The audit tool names six measured non-defect classes (a reuse bit on a register-less slot that nvdisasm
  rejects, text-inert bits the measured rows fail closed on, the vendor's own `INVALIDn`, the NaN print policy, a
  text both oracles print that cubit refuses to assemble under a silicon-measured law — BUG-037's MMA operand
  alignment); 118 words of the undocumented
  forms above are `vendor_unknown`, not defects; the 2026-11-04 oracle records had 386 legal words marked as
  rejected through a scratch-file race of the tool — re-measured on
  2026-11-06); the families are listed in cubit `docs/KNOWN_ISSUES.md` T1. Rebuilt from measurement
  since: the **`IDP` family on the four tables** (2026-11-06, session T-11, the review of the B300 lane's D2 rows for
  NEEDS 36: 12 forms × 2 operand shapes — `IDP.{4A,2A.LO,2A.HI}.<A>.<B> Rd, Ra, Rb | URb, [-]Rc`, [77:76] the mode,
  73 / 74 the signedness of the two types, 75 `-Rc`; the mixed types as typed keys (D2), the symmetric forms as
  mod-group rows; 24 rows per table — the uniform rows had no `-Rc` field, the 2A half of the family no rows, the
  sm121a register rows no `-Rc` and `Rc = RZ` baked), the **`BRX` / `BRXU` / `RET` family** (2026-11-06, session T-10: 8 keys × 46 rows — register, a signed
  56-bit dword offset, predicate operand, `.INC` / `.DEC`, BRXU's `.U` / `.DIV` / `.CONV`, RET's `.ABS` / `.NODEC`
  and UR form; 13 harvest keys retired), the **dense `IMMA` family** (T-10: 32 typed keys `IMMA.<16816|16832>.<A>.<B>
  [.SAT]_R_R_R_R[_UP]` × 1 measured row — types, shape, `.SAT`, the ROW / COL layouts baked and fail-closed, the
  `[!]UPn` fifth operand — for 14 dense harvest keys whose register fields sat two bits off), **15 harvest-row field
  repairs + 105 junk keys / 5 groups retired** (T-10, `audit-fix-2026-11-06-t10`: every repair asserted against
  nvdisasm before the write, every retirement checked against the 249 929 distinct words of the production census —
  ATOMG CAS Pd, STS.64 opcode bit 9, the MOV / WARPSYNC / VOTE junk fields, UFMUL, UTMAREDG, USETMAXREG, LDCU, BAR.RED
  twins, SYNCS RED.A1T0, F2I.NTZ UR; three batch-1 retirements refuted by cubit's pins were undone with the
  measurement: `LD_R_ARURI [E]`, `IADD3_R_P_P_R_II_R [X]`, the LDGSTS `[128,E,LTC128B]` rows),
  the **`BRA` family on sm103a / sm100a** (T-10: the SM120 measurement applies 1:1, the 0x547 uniform-predicate
  form included — 5 harvest keys / 9 rows → the same 6 keys / 66 rows per table),
  since: the **SM120 `BRA` family** (2026-11-06, session T-9: 6 keys × 66 rows — `BRA[.INC|.DEC][.U|.DIV|.CONV]
  [.ANY][.NOT_TID0] [Pn,] [[~]URn | [!]UPn,] target`, the signed 56-bit dword target, the `~UR` and `[!]UPn`
  glyphs, the 0x547 uniform-predicate form; 15 harvest keys retired), the **`LEA` family on the three tables** (2026-11-06: 24 keys × 36 rows, `LEA[.HI][.X][.SX32]`
  × register / imm32 / UR forms, the `~R` glyph of the `.X` forms, nvcc's text-inert values baked —
  56 / 18 / 18 harvest keys retired), the SM120 `ATOMS_R_ARURI_R` lattice (2026-11-06: 9 operations × 4
  types, the harvest groups that read the UR byte as a register retired), the SM120 `HADD2` register form
  (2026-11-06, 12 groups), the plain `.EL` ATOMG twins (U32 and 64-bit base), the `UTMALDG` plain form
  (five dimensions), the sm103a/sm100a `LEA_R_P_R_UR_R_II`, `LDG_R_dARI` SM-scope offset, `LDGSTS` imm
  sign and `IADD.XOR` / `LEA.HI.X` rows of coldface's B300 review, the `IADD3.X` carry-in negation on
  the three tables. The GB10 host census (2026-11-04) found `LDL.64 Rd, [Rn+URm+imm]`
  without an SM120 row (95 cuBLASLt words), a wrong `MUFU` UR-form function field, the F2F type-pair
  rows and the IMAD alias groups — fixed in `15caf70`; the remaining SM121a text-parity families
  (`R2P`, `BAR`, `IMNMX`, `SEL`, IMMA `.ROW/.COL`, …, 61 words) are listed in cubit's V3.
- **The other three tables at `3fb3b82` (2026-11-08, sessions T-20 / T-21 / T-22): sm121a 2 654 of 50 444 rows without `_src`, sm103a 893 of 35 876, sm100a 893 of 35 870** (25 685 / 15 858 / 15 858 at the start of T-20; 25 427 / 9 714 / 9 714 after its first batches) — the inventories (`cubit tools/audit_table.py --summary`): on sm121a the lattices LDG 7 216 / REDG 6 407 / ATOMG 3 523 / ATOM 3 465 / STG 1 222 (85 %), then ISETP 635, UISETP 512, QMMA 269, F2I 266, … and 254 `?` partial keys; on sm103a / sm100a LDG 7 030 / REDG 6 173 / STG 1 215 (91 %), then F2I 306, ISETP 129, …. **First batch, sm121a: FSETP / DSETP measured on `-b SM121`** (cubit `tools/probes/re_sm120_fsetp_dsetp.py --arch SM121`: the sm120 law of 2026-11-06 holds word for word — 1 022 form + 12 glyph + 223 lattice words and all 2 688 FSETP / DSETP words of the GB10 census corpus; 432 measured rows on the six plain keys, `FSETP_P_P_R_UR_P` created, 312 groups added, 105 dotted harvest keys + 7 junk keys retired as `_meta.key_aliases`; 25 685 → 25 427). **First batch, sm103a / sm100a: the REDG uniform-indexed forms measured on `-b SM103` / `SM100`** (G's batch-14 probe: 9 334 form + 58 lattice words identical to SM120 on both targets; `REDG_ARURI_R` / `REDG_ARURI64_R` created with 9 216 measured rows per table, the 6 144 dotted harvest keys of each retired as aliases; 15 858 → 9 714 on both; the B300 corpus strict round trip unchanged at 85 704 / 85 792). **Batch 4, sm103a / sm100a: LDG.256 (dARI / ARURI64 / ARURI) and STG.256 measured on `-b SM103` / `SM100`** (G's batch 10–13 probes with `--arch SM103` / `SM100`: every form word prints the SM120 text on both targets; `LDG_R_R_{dARI,ARURI64,ARURI}[_II][_P]` and `STG_{dARI,ARURI,ARURI64}_R_R[_II]` created with 13 824 + 1 728 measured rows per table, the 7 782 + 1 164 dotted 2026-09 keys of each retired as aliases; 9 714 → 1 584 on both; the B300 corpus unchanged). **Batch 5, sm121a: the same five lattices measured on `-b SM121`** with the GB10 census words as witnesses (every form word prints the SM120 text; the 99 LDG.256 + 68 STG.256 corpus words obey the law; the 14 286 dotted 2026-08/09 keys of the three families retired as aliases; 25 427 → 11 160; the GB10 census 91 927 / 91 927 unchanged). **Batch 6, the three tables: ISETP / UISETP measured on `-b SM121` (the 64-bit compare included; 3 155 GB10 witnesses) and on `-b SM103` / `SM100` (no 64-bit compare there — bit 80 text-inert and fail-closed, NEEDS 46; the four phantom `.S64` / `.U64` groups retired), FSETP / DSETP on sm103a / sm100a (NEEDS 58: bit 75 = the vendor's `.XOR`), PLOP3 / P2R / R2P on the three (the sm103a / sm100a PLOP3 harvest row whose "pred" field sat inside the opcode nibble retired); sm121a 11 160 → 10 000, sm103a / sm100a 1 584 → 1 280.** **Batch 7 (T-21 / T-22): the float conversions (F2I / I2F / FRND / MUFU) and the FP32 ALU (FADD / FMUL / FFMA / FMNMX) measured on `-b SM121` / `SM103` / `SM100`** (every word of G's two probes obeys the sm120 law on the three targets; the 12 + 14 plain keys carry the measured lattices, the dotted / mis-tokenised keys are aliases; the B300 corpus gains its `I2F.U16` / `FMUL.RZ` holes, 85 707 / 85 792, the GB10 strict residue drops 40 → 17), **plus NEEDS 59: the sm103a / sm100a VIMNMX3 rows measured (the harvest's predicate field sat inside the opcode nibble and claimed FMNMX words) and sm121a's junk key `MUFU_R_L` retired; sm121a 10 000 → 9 575, sm103a / sm100a 1 280 → 898.** **Batch 8 (T-22), sm121a: ATOM / ATOMG in the plain uniform-indexed form measured on `-b SM121`** (cubit's `re_sm120_atom_aruri.py`: 18 398 form + 55 lattice words; `ATOM_P_R_ARURI_R` / `ATOMG_P_R_ARURI_R` with 8 640 + 9 600 measured groups — the `.128` type, the no-`.E` plane and ATOMG's SAFEADD the harvest never saw —, the 6 912 dotted keys retired as aliases; the GB10 census unchanged; **sm121a 9 575 → 2 663**). `sm121a.json` is 87 MiB after this batch — within 13 MiB of GitHub's 100 MiB file limit; the next lattice needs the owner's schema decision (cubit `T/NEEDS_OWNER.md` 61). **Batch 9 (T-22), the three tables: the UIADD3 `.64` groups measured** (sm120's rows re-measured on `-b SM103` / `SM100` / `SM121` with the corpus words as witnesses; the B300 corpus' last `UIADD3.64` holes closed — 85 721 / 85 792 words, 296 / 328 cubins strict-exact —, the GB10 census at 91 926 / 91 927 re-encode; the `.X` rows measured with sm120's field lists once the plain rows' dead bits exposed the harvest rows' overprint — bug459a; sm103a / sm100a 898 → 893, sm121a 2 663 → 2 654). The production line of sm120 (one probe per family, the table's own strict corpus as the gate — the GB10 census for sm121a, the B300 publish corpus for sm103a / sm100a) continues table by table; the sm120 lattice probes apply to sm103a / sm100a directly (G recorded 0 text differences on `-b SM103` / `SM100` for STG.256 / REDG and the `INVALID3` L2 plane for LDG.256).
- `sm121a.json` still carries 15 keys with a glued predicate token whose plain rows do not provably
  cover them (`HMMA.1688.F16_P0_…`, `HMMA.SP.16816.*_P0_…`, `LEA_P0_R_R_R_II`, `MOV_P0_R_II`,
  `PLOP3.LUT_P1_…`, `STS.64_P0_AR_R`, …); they need a GB10-side probe batch.
- `base_op` values include modifiers (`LDG.E.EL.ELL2.256.STRONG.GPU`): `base_op` is the dotted
  mnemonic in front of the first `_` of the key, `operand_sig` the rest (every entry carries both
  since 2026-11-03).
- Keys with `?` are partial: the encoding is valid, the full operand syntax is not determined.
- Scheduling metadata outside the `hw_measured` rows is synthetic or ptxas-weight derived.

## Limitations

- `sm120.json` targets SM120 / consumer Blackwell; `sm103a.json` targets SM103a (B300);
  `sm121a.json` targets SM121A (GB10, DGX Spark). Together they are not a complete
  SM100/B200 ISA, though the measured tcgen05 encoding layer is byte-identical between
  SM100a and SM103a.
- The tables use different key-granularity conventions: canonical forms with nested
  variants (`sm103a.json`) vs enumerated variant keys (`sm120.json`, `sm121a.json`).
  Raw key counts are not comparable across tables; the mnemonic-family coverage is
  essentially the same (~150 families each).
- SM121A entries are validated on GB10 (Grace-Blackwell, unified memory) only.
- Some keys are marked partial (`?`): the encoding is valid, but the full operand
  syntax is not yet determined.
- Some entries are scheduling-only: they describe pipeline behavior without a complete
  encoding record.
- Some instruction names use provisional labels (`INVALID*`, recovered modifier names,
  or internal pattern names) where NVIDIA has no public terminology.
- Metadata under `_discovery` records provenance and confidence for unusual entries; users
  should inspect it before relying on those forms in production code.
- The former `IMMA.SF.16832.U8.U8` entry (opcode 0x47e; a 2026-03-26 hypothesis without a numerical
  oracle) is **`QMMA.SF.16832.F32.MXINT8.MXINT8.E8_R_R_R_R_R_R_UR`, measured on sm_120 silicon** (2026-11-06:
  executes, writes fp32, honours the E8M0 block scales, matches neither documented QMMA.SF input type;
  2026-11-07: both input element types identified as OCP MXINT8 — two's-complement int8 with six fractional
  bits under the E8M0 block scale, byte-value sweeps of A and B reproduced exactly on all 6 × 256 cases) —
  an undocumented block-scaled MMA of the QMMA.SF family selected by opcode bit 2, the MX integer kind PTX
  does not expose. nvdisasm 13.1 / 13.3 have no text for it. `_discovery` records the measurements and the withdrawn claims; the old key
  and the interim `UNIDENTIFIED` key are `_meta.key_aliases` entries. Scheduling fields are copied from the E4M3.E4M3 donor row and marked so —
  not a measured latency.

## Related Work

- [`cubit`](https://github.com/kacper-daftcode/cubit) — companion SM120 / SM103a / SM121A SASS
  assembler/disassembler using this database (per-arch tables: `sm120.json`, `sm103a.json`,
  `sm121a.json`).

## Citation

If this artifact is useful in a paper or project, please cite the repository and the
commit hash of `sm120.json` used in your work.

## License

MIT — see [`LICENSE`](LICENSE).

Reverse engineering for interoperability is protected under EU Directive 2009/24/EC
Article 6 and US DMCA §1201(f).
