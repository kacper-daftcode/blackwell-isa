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

- **SM120 strict re-encode census — gated on every table revision pushed since 2026-11-02, last run 2026-11-06 at this revision (cubit `main`):
  1 811 600 / 1 811 600 instructions over 169 cubins** — every word decodes, prints, re-parses and
  re-encodes to the identical 128 bits (`cubit roundtrip-strict`). The corpus is the production
  kernel set of the vLLM-Moet DeepSeek-V4.1 / GLM-5.3 SM120 deployment: FlashInfer 0.7.2
  sparse-MLA AOT + JIT caches (544 064 + 544 200 words), FlashInfer MXFP8 GEMMs (16 016),
  FlashInfer CAKE DSV4.1 mixed-cache kernels (599 048), b12x MoE kernels (52 024), DeepGEMM
  SM120 FP4/FP8 kernels (56 248). Before this revision the same census stood at
  1 797 483 / 1 811 600 (99.2 %) with five forms missing (see CHANGELOG 2026-11-02).
- **Whole-cubin reassembly (gated on every push; last run 2026-11-06 at this revision):** `cubit disassemble
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

A typical encoding entry:

```json
{
  "and_base": "0x0000000000000000000000000000082e",
  "variable_mask": "0x1c00000000000000000000000000f000",
  "fields": [
    { "shift": 12, "bits": 4, "token_idx": 0, "extraction": "guard" },
    { "shift": 16, "bits": 8, "token_idx": 1, "extraction": "reg" }
  ],
  "count": null,
  "_src": "prmt-ur-sm120-2026-11-02",
  "_note": "nvdisasm 13.1 sweeps: guard[15:12], Rd[23:16], ... (what was measured, what stays outside the row)"
}
```

`and_base` is the constant part of the 128-bit instruction word. `variable_mask`
identifies operand-dependent bits. `fields` tells an assembler/disassembler where to
read or write each operand field (`token_idx` is the operand position, 0 = the guard).

### Provenance

- `_src` on a row names the measurement that produced or last repaired it; `_meta.provenance.sources`
  lists every tag of the table with its kind (nvdisasm field probe, encoder fix wave, native capture),
  date, and — where they exist — the cubit tool and test that hold the evidence. `_note` is the
  measurement in prose. Rows **without** `_src` (18 841 of 26 598 SM120 variants at this revision) are
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

- 18,845 of 26,580 SM120 encoding variants carry no `_src` (sm103a 15,864 / 22,420,
  sm100a 15,864 / 22,420, sm121a 25,696 / 26,406): these are the harvest rows of 2026-06..08
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
  **is at its end state (2026-11-06, session T-11): 1 row — the documented `IMMA.SF.16832.U8.U8` hypothesis**
  (every probe word nvdisasm-rejected, kept by the owner's rule for undocumented forms until a numerical oracle on
  silicon) — from 1 977 on 2026-11-02, 418 on the T-7 tables, 230 (T-8), 215 (T-9), 14 (T-10). The 13 closed on
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
- The `IMMA.SF.16832.U8.U8` entry is an **unvalidated historical hypothesis**.
  Its former hardware-validation, pipeline, latency and dual-issue claims were not
  supported by a numerical oracle or retained overlap measurement. The historical
  opcode sweep classified nonzero output as success and assigned the mnemonic by
  analogy. NVIDIA `nvdisasm` 13.1 does not identify the canonical candidate or the
  tested QMMA-derived candidates. That does not prove the opcode is unimplemented;
  its identity and operands still require independent reverse engineering. Encoding
  fields remain available as a historical record, but must not be used to infer
  scheduling safety or a performance advantage.

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
