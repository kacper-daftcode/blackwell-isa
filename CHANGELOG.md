# blackwell-isa Changelog

## SM120/SM121a batch 2: the GB10-host census follow-up (November 4, 2026, `15caf70`)

The 2026-09 SM121a census script re-run on `gx4` (NVIDIA GB10) exposed four defects, fixed with
nvdisasm 13.1 probes (`blackwell-isa-internal/tools/t3_batch2_20261104.py`; probes, fixtures and
tests in cubit):

- `sm120.json` `LDL_R_ARURI` — `LDL[.size][.LU] Rd, [Ra+URb+imm]` had no row (95 cuBLASLt sm_120
  words): 14 groups, size enum [75:73] (7 = INVALID7), bit 84 (`.EF` when clear), 85 = `.LU`, 86 =
  `.NA`, 91 = the uniform frame, 76 = the descriptor form (`tools/probes/re_sm120_ldl_aruri.py`).
- `sm120.json` `MUFU_R_UR` rebuilt — 26 groups (10 functions [77:74] × the half types the function
  takes, abs 62 / neg 63 on the UR operand); the four harvest rows had baked a sample's UR and
  function with a few-bit mask (`MUFU.EX2 R4, UR7` decoded as `LG2`; `re_sm120_mufu_ur.py`).
- `F2F.F64.F32_R_R` / `F2F.F32.F64_R_R` on both tables — Rd[23:16], Ra[39:32], abs, neg, guard; the
  sm121a rows had Rd at shift 17 / Ra at 33, so 217 census words fell into the `F2F_R_R` 'F32,F64'
  harvest group and printed the wrong type order; that group is removed on both tables (on SM120 it
  assembled `F2F.F32.F64 R8, |R15|` with the abs in the text-inert bit 73).
- `sm120.json` IMAD — the seven alias-named harvest groups (`MOV`, `MOV,U32`, `IADD`, `IADD,U32`,
  `SHL,U32`; they put the Ra token on the Rb field) are removed and every four-operand form has a
  `U32` twin: nvdisasm's `.MOV` / `.IADD` / `.SHL` are a print of the operands (law measured on 144
  words, `re_sm120_imad_alias.py`), applied by cubit's printer and encoder.

SM120 16 106 → 16 107 forms / 20 863 → 20 894 variants; SM121a 26 381 → 26 380 variants.

## SM120: era-key aliases, undocumented-form provenance, measured `stallfix` section (November 4, 2026, `b063d43`)

Session T-3 of the cubit quality work (`blackwell-isa-internal/tools/t3_20261104.py`; evidence in cubit
`tests/fixtures/sm120_era_keys_20261104_nvdisasm.jsonl`, `tests/sm120_era_keys_20261104.rs`,
`tests/stallfix_sm120.rs`, `tools/stallfloor/`):

- **Era keys retired into `_meta.key_aliases`.** The 20 `sm120.json` keys whose mnemonic carried a
  glued predicate token (`DMMA.8x8x4_P0/P2/P5_…`, `ELECT_P0_P_UR_P`, `F2FP.F16.F32.PACK_AB_P2/P5_…`,
  `F2IP.U8.F32[.RELU]_P0/P2/P5_…`, `HMMA.1688.F16_P2/P5_…`, `IADD_P0_R_R_II`, `MOV_P0_R_II`,
  `PLOP3.LUT_P1_…`, `QMMA.16816.F16.E4M3.E4M3_P0/P2/P5_…`, `S2R_P0_R_L`, `SHF.L.U32_P0_…`) and the 10 of
  them in `sm121a.json` whose plain rows provably cover them. nvdisasm 13.1 names every one of their
  canonical words with the plain mnemonic (probed one word per call). Where the era row covered words
  the plain row did not, the plain row was widened by probe: `DMMA.8x8x4_R_R_R_R` gained its guard
  field, `F2FP.F16.F32.PACK_AB_R_R_R` the guard field and the text-inert window [71:64] (the lattice
  around it is recorded in the row's `_note`: 73/74/82/83 rejected, 79–81 `???n`, 75–78 RELU/BF16/
  SATFINITE/MERGE_C). `MOV_P0_R_II` and `PLOP3.LUT_P1_…` carried a 1-bit "guard" at bit 7 — they had
  claimed UMOV / UPLOP3 words. 16 126 → 16 106 forms, 20 883 → 20 863 variants (SM120); 23 833 → 23 823
  / 26 391 → 26 381 (SM121a). cubit's loader resolves an alias as an encoder-visible, decoder-invisible
  copy of the plain entry (lookups and old text keep working, the decoder never prints a glued name).
- **Undocumented forms stay and say so** (owner's decision, 2026-11-03). New `_meta.provenance.sources`
  entries with `nvdisasm_sm120: "no text"`: `undocumented-e3m4-2026-03` (22 `QMMA.SF[.SP]` rows with
  the E3M4 type code; 11 dense combinations executed on RTX PRO 6000 Blackwell 2026-08-09),
  `undocumented-utc-sm120-2026-03` (`UTCHMMA/UTCIMMA/UTCIMMA.KIND2/UTCQMMA.1CTA` executed on an
  RTX 5090 2026-03-28, `UTCATOMSWS`), and the note on `bug234-2026-08-28` (FFMA2). cubit's dynamic
  audit classifies their words as `vendor_unknown`, never as defects.
- **`stallfix` section for SM120** (rules v3): 23 stall floors measured 2026-11-03 on 4x NVIDIA RTX PRO
  6000 Server Edition (cubit harness `tools/stallfloor/`, 370 cubit-assembled kernels, 32 CTAs ×
  1 024 threads × 20 repetitions; raw-result SHA-256s in the section's `provenance`): `floor_global`
  4, `dmix` 5, `pcarry` 5/4 (new keys), guard d0 impossible up to the cap (`guard_d0_forbid`), guard
  d1 11 / d2 9, `xread` 6/4/2, `urpath` d1 3, `uguard` 10/8/6; `r2ur` unmeasured (the harness model
  was wrong; the section carries the cap for it until the corrected kernel runs); MUFU needs the
  scoreboard at every distance; S00 is a hardware interlock on sm_120, never a floor.
- README: Data Model (`key_aliases`, `provenance`, `operand_roles`, `stallfix`), the SM121a validation
  status re-run on the GB10 host `gx4` (decode 100 %, text parity 99.59 %, 17 keys named), known
  issues (463 audited rows, the GB10-census findings, the 15 remaining SM121a era keys).

## SM120 dynamic-audit batch 1: LDGSTS base field, LDC reuse, 22 foreign-opcode rows (November 3, 2026)

cubit's dynamic audit (`tools/audit_dynamic.py`: the canonical word of every row the static audit
flagged plus one-field-at-a-time variants, each rendered by nvdisasm 13.1 one word per call and
decoded / re-encoded by cubit; 88 388 words over 5 962 rows) named three defects of `sm120.json`
that are fixed here (`tools/audit_fix_20261103.py` in blackwell-isa-internal; evidence in cubit
`tests/fixtures/sm120_audit_20261103_nvdisasm.jsonl`, test `tests/sm120_audit_20261103.rs`):

- `LDGSTS_*` (70 rows): the shared-address base register field was 9 bits wide at shift 16 and
  swallowed bit 24, the low bit of the global base register. `[R5+0x10]` decoded as `[R261+0x10]`
  (906 probe words). The field is 8 bits wide now.
- `LDC_*` / `LDCU_*` (13 rows): a `reuse` field at bit 122. nvdisasm prints the same text with
  the bit set or clear (a constant-bank load has no reusable operand), so the bit is text-inert
  and leaves the rows.
- 22 harvest rows removed whose probe words nvdisasm names as a different instruction or rejects:
  `FSETP_P_P_L_FI_P` and `FSETP_P_P_L_L_P` (10 groups; their words are `P2R.B3`),
  `F2FP.F16.F32.PACK_AB_P_R_R_R` (`HMMA.1688.F16`), `HMMA.1688.F32_P0_R_R_R_R` and the three
  `HMMA.SP.*_P0_*` rows plus `HMMA.1684/1688/INVALID3.F16_P0_R_R_R_R` and
  `HMMA.SP.1688.F16/F32_P0_*` (`F2FP.F16.F32.PACK_AB`, the plain `HMMA.1688.F16`, or illegal),
  `LOP3.LUT_P_R_R_R_R_II_P` (`SEL`), `QMMA.16816.F16.E4M3.E4M3_P_R_R_R_R` (`MOVM.16.MT88`).
  16 138 -> 16 126 forms, 20 905 -> 20 883 variants.

The production census of the vLLM-Moet SM120 kernels decodes identically per cubin before and
after (166 cubins, 1 811 600 / 1 811 600 words): no corpus word was routed through a removed row.
Not changed, for the owner to decide (cubit `T/NEEDS_OWNER.md`): the 19 keys whose mnemonic
carries a glued predicate token (`DMMA.8x8x4_P0_...`, `F2FP.F16.F32.PACK_AB_P2_...`,
`F2IP.U8.F32_P0/P2/P5_...`, `QMMA.16816.F16.E4M3.E4M3_P0/P2/P5_...`, `S2R_P0_R_L`, ...): nvdisasm
names their words with the plain mnemonic, but two of the families are pinned by cubit tests as
"era keys". Remaining audit findings after this batch: 483 rows (from 1 977), listed by family in
cubit `docs/KNOWN_ISSUES.md` T1.

## Hygiene pass: control-code bits out of the templates, base_op / operand_sig everywhere (November 3, 2026)

No encoding or decoding changes. Every encoding variant of the four tables carried, in
`and_base` bits [127:105], the control word (stall count, yield, barriers, wait mask, reuse) of
the compiler sample it was learned from - 880 variants in `sm120.json`, 1 670 in `sm103a.json`,
1 674 in `sm100a.json`, 546 in `sm121a.json`. Decoders strip [127:96] of both the word and the
template before matching and assemblers compose the whole upper 32 bits from the control code,
so these bits were never part of an encoding identity; they only made the templates look as if
a stall count belonged to an instruction form. They are cleared now (`tools/hygiene_pass.py` in
blackwell-isa-internal). Bits [104:96] are untouched. The 43 + 17 + 17 + 52 entries without
`base_op` / `operand_sig` carry them now in the convention of the rest of the table (`base_op` =
the dotted mnemonic in front of the first `_` of the key, `operand_sig` = the rest). Proof of
neutrality with cubit at 0fdd91a: the production census strict round trip is identical per cubin
(166 cubins, 1 811 600 / 1 811 600 words before and after), 157 / 157 cubins byte-identical through
`disassemble --frozen -> asm -T`, cubit's suite green. Recorded in
`_meta.provenance.history["hygiene-2026-11-03"]`.

## SM120: the last five census forms, provenance metadata, regenerated references (November 2, 2026)

`sm120.json` gains the five instruction forms that kept the production census of the
vLLM-Moet SM120 kernels below 100 % strict re-encode (1 797 483 / 1 811 600 before, all
169 cubins / 1 811 600 words after). Each form was measured with nvdisasm 13.1 one word per
call (cubit `tools/probes/re_sm120_*.py`, fixtures `tests/fixtures/sm120_*_nvdisasm.jsonl`,
tests `tests/sm120_*.rs`); every row carries exactly its measured fields, the lattice
around it stays unclaimed (fail-closed):

- `HMUL2_R_R_FI_FI` (opcode 0x832, new key, 12 groups): the packed-immediate multiply
  `HMUL2[.BF16_V2][.FMZ][.SAT][.FTZ] Rd, Ra, imm_hi, imm_lo` with `-Ra`, `|Ra|`, `.H0_H0` /
  `.H1_H1`; 12 664 words of FlashInfer's CAKE DSV4.1 mixed-cache kernels (`3f80 3f80` = 1.0
  bf16, `7780 7780`). Bit 85 selects bf16, bits 76/77/80 FMZ/SAT/FTZ; bit 78 (INVALID3), bit
  91 (illegal) and the text-inert bits 64-71, 79, 81-84, 86-90, 92-95 are outside the rows.
- `LDSM_R_ARI` (0x83b, +30 groups): `LDSM.U4x16P64TO8` / `LDSM.U6x16P32TO8` (FP4 / FP6
  matrix loads with unpack to 8-bit lanes, PTX `ldmatrix .b8x16.b4x16_p64` / `.b6x16_p32`)
  for all five shapes and the x1/x2/x4 counts in the `[R+imm]` form; 576 words of DeepGEMM's
  grouped FP8xFP4 MoE GEMMs. Datatype [77:75] = 5 / 6 next to 0 (`16`) and 7 (`8`); codes 1-4
  are unnamed by nvdisasm.
- `PRMT_R_R_UR_R` (0xc16, new key, 7 groups): the byte permute with a uniform-register
  selector, modes [74:72] 0..6 (7 = INVALID7); 544 CAKE words.
- `ARRIVES_ARURI` (0x9b0, new key, 2 groups): `ARRIVES.LDGSTSBAR.64[.ARVCNT] [R+UR+imm]`
  (PTX `cp.async.mbarrier.arrive[.noinc].shared.b64`); 297 CAKE words. First ARRIVES row of
  the table; kind bit 72, size [75:73] = 5 (.64), ARVCNT bit 76 measured, `???n` codes outside.
- `UTMASTG_AURI_AURI` (0x3b5, +4 groups): the 1D, 2D, 4D and 5D dimension codes next to the
  harvest-era 3D row (dimension [81:79], 5-7 INVALIDn; bit 76 desc, 82 IM2COL, 83 SCATTER4
  outside); 36 DeepGEMM words.

Table metadata, all four tables: the loose per-wave notes that lived as `_meta.bug203` …
`_meta.bug416`, `fix_*`, `sigfix`, `pruned_junk` keys and (sm121a) 34 top-level `_errata_*`
records now live verbatim under `_meta.provenance.history` / `.errata`;
`_meta.provenance.sources` documents every `_src` tag a row carries (kind, date, cubit tool
and test), and `_meta.provenance.legend` says how `_src` / `_note` are to be read (rows
without `_src` = the 2026-06..08 harvest). cubit's `tools/sync_table.py` validates the
convention (every row tag documented, no loose keys, no undocumented top-level records).
Row data is unchanged by this step. Tool: `blackwell-isa-internal/tools/provenance_meta.py`.

The four HTML references are regenerated from these tables (they were last generated
2026-10-23 for SM120 and 2026-09-19 for the others). The generator is now driven by
`_meta.architecture`, states only table metadata and public facts in the overview, shows
the measured control-code layout (bits [108:105] stall, 109 yield, [112:110] write barrier,
[115:113] read barrier, [121:116] wait mask, [124:122] reuse) instead of the pre-measurement
table, prints each variant's `_src` / `_note`, and stamps the generation date and table
SHA-256. `index.html` lists the four pages with their table revisions.

## SM120 ATOMS [R+UR+imm], STSM.8.MT168 and UBLKPF.L2 (October 29, 2026)

The last three forms of FlashInfer 0.7.2's sparse-MLA prefill cubin (108 words of 217 200):
`ATOMS_R_ARURI_R / MAX,S32` replaced — the harvest row read the address UR byte [71:64] as a
fourth register, declared no offset and no guard; now the measured geometry of the ADD group
(guard, Rd, base R with RZ printed `[UR+imm]`, Rb, signed 24-bit offset, UR), opcode 0x98c with
bit 91. `STSM_ARI_R` gains `8,MT168` / `2,8,MT168` / `4,8,MT168` (8-bit transposed matrix store,
PTX `stmatrix ... .trans.b8`, bit 75). `UBLKPF_AURI_UR / L2` and `UBLKPF_AURI_UR_II / L2`
(bulk L2 prefetch, PTX `cp.async.bulk.prefetch.L2`, with and without `desc[UR]`) replace the
unusable `AUR` harvest spelling. nvdisasm 13.1 sweeps: 65+54, 118+34 and 70+36 words. With
this revision both sparse-MLA cubins of the census round-trip completely (cubit `accd984`).

## SM120 F2FP.BF16.E8.UNPACK_B (October 28, 2026)

`F2FP.BF16.E8.UNPACK_B_R_R` — the UE8M0 → BF16x2 unpack (PTX `cvt.rn.bf16x2.ue8m0x2`),
48 words of FlashInfer's DeepSeek-V4.1 full-BF16 sparse-MLA prefill kernel. nvdisasm 13.1 sweep
of 36 words: guard, destination [23:16], source [39:32], `.H1` = bit 88, `.reuse` = bit 123;
the source-format lattice ([87:80] groups, bit 76 BF16 destination legal only with E8) stays
outside the row.

## SM120 CCTL register-address forms (October 25, 2026)

`CCTL_ARI` (opcode 0x98f) gains the seven `.E` operations PF1, WB, IV, RS, PML2, DML2, RML2
next to the harvested `E,PF2` (operation [90:87]; the other eight codes are illegal) with the
measured guard / address register / signed 32-bit offset fields. Found by cubit's image gate on
vLLM-Moet's `moe_l2_discard` kernel (`CCTL.E.RML2 [R2]`); with it every extension cubin of
image r21 (11 cubins, 504 128 instructions) round-trips.

## SM120 CS2UR (October 23, 2026)

`CS2UR` (opcode 0x8cb) reads any system register into any uniform register: both groups now
carry the measured uniform guard [15:12], URd [23:16] (255 = URZ) and system-register id
[79:72] (all 256 ids accepted; bit 80 selects the 64-bit/plain form vs `.32`, as on CS2R). The
harvested word had the guard, UR7 and SR_CLOCKLO baked into `and_base`; the junk `CS2UR_UR_II`
row that re-read the register number as two immediates is removed (reported in PR #1).

## SM120 byte matrix transpose loads (October 22, 2026)

Add `LDSM.8.MT1616` and `.2` for the existing register, register-plus-uniform,
and uniform shared-address forms. CUDA 13.1 emits these instructions for
`ldmatrix.sync.aligned.m16n16.x1/x2.trans.shared.b8`; the previous table could
not decode an actual compiler word, `000e62000801390000000005000c783b`.

The six groups retain the corresponding `.16.MT1616` field geometry and masks,
with the measured datatype selector at bits 75–77 set to 7. Evidence includes
142 bounded NVIDIA operand/guard/address probes and literal full-word
reassembly, plus strict reconstruction of two complete three-kernel ELFs
(656 words) and 329 literal NVIDIA compiler/canary words. Existing groups and
other architectures are unchanged. No masks were widened and no raw-word or
completion fields were added. The `.4` count, `INVALID3`, and unsupported
datatype combinations remain unclaimed. This is encoding evidence; GPU
semantics, latency, throughput, and kernel performance require separate tests.

## Scheduling provenance correction (October 22, 2026)

Qualify the single SM120 `IMMA.SF.16832.U8.U8_R_R_R_R_R_R_UR` entry as an
unvalidated historical opcode hypothesis. The retained sweep accepted nonzero
output without checking an expected matrix result, and supplied no overlap timing.
Remove active hardware-validation, physical-pipeline and inferred-latency claims;
preserve them explicitly as historical, unvalidated provenance. The encoding-group
source annotation carries the same qualification. No encoding base, variable mask,
operand field, sample count or other instruction entry changes.

The HTML reference and README distinguish provisional instruction identity from
validated encoding and measured scheduling behavior. No new unsupported opcode was
executed to justify this correction.

## Fleet silicon wave (August 26, 2026): ATOMS/ARI coverage + decoder-junk cleanup
Fold of 29 silicon-verified fleet bug fixes into the canonical tables
(each carried a pinned corpus/silicon verdict upstream in the cubit fix
pipeline; compose report in the internal O2 registry).
sm120.json: +13 keys (ATOMS/ATOMG uniform-register ARI family, UTCATOMSWS
UR forms, UTMAREDG_AURI_AURI), -24 keys (dead LDC/LDCU cAI/cARI/cAURI rows
folded into their canonical carriers, REDG dARI P-pred junk, the duplicate
UPLOP3 lattice row); 61 keys content-refreshed.
sm103a.json: +2 keys (ATOMS_R_ARURI / ATOMS_R_AURI), -2 keys (SYNCS PHASECHK
junk rows), 48 keys content-refreshed.
sm100a.json: regenerated derivative of the new sm103a revision; offline
b4-corpus replay on the archived sm_100 population: 20,009,403/20,024,512
(99.9245%, gate 99.90 PASS; previous revision 99.9219%). Companion replays:
sm_103 population 99.8639% (was 99.8385%), sm_120 population 99.8573%
(unchanged). WARPSYNC.COLLECTIVE encode regression on the sm100a derivative
found by this wave and fixed on the tooling side (family-wide fixup
exemption; cubit BUG-182).
sm121a.json: unchanged.

## tcgen05 corpus wave-3 (August 2026): TMA-uniform tables
sm103a.json: full UTMALDG/UTMASTG/UTMAREDG/UTMACCTL coverage from the
fleet corpus (1.25M-word union, vendor-cluster method; 12 refreshed +
14 new groups + the previously unseen 5D/IM2COL no-desc key; the stale
pre-corpus 2D stub retired to encode_only — it shadowed every real
desc-carrying 2D word). Decode of the 59-probe CUTLASS sm100 corpus is
now 8550/8550 vendor-exact on the UTMA family (was 6,705; unknowns
734 -> 0, mis-decodes 1,111 -> 0); the 412-cubin regression gate stays
green (unknowns 7,060 -> 6,064; roundtrip 412/412) and the encode-side
audit (521 cubins) only improves (Match +2,525 total, no per-file
regressions). sm100a.json re-derived from the refreshed canonical;
offline replay unchanged at 99.9219% EXACT-minus-sched (20.0M instr).

## O2 consolidation (August 2026)
Canonical reconciliation with the cubit fleet (one wave, per R3):
+ 80 silicon-verified fleet instruction forms (cubit errata series evidence),
- 108 junk `_?` rows (R4 decoder census: 0 hits on 1.64M natural sm120
  instructions, 240/240 corpus cubins byte-identical),
- 1 silicon-pinned form: PRMT_R_R_R_II (encode trap — imm-selector in text
  position 4 is the PTX operand order and silently swaps sel/b; fleet pins
  fail-closed with an order hint; decode coverage unaffected),
154 shared forms: silicon fields (mod_groups/ctrl_class/scheduling/
encode_only/base_op/mercury) reset to the silicon-verified cubit content —
corpus roundtrip on the 240-file sm120 census (1,076,075 instructions)
swings errors 64,194 -> 651 (match rate 94.0% -> 99.86%).
The 3 fuzzy rename-pairs among them were rebutted as DISTINCT forms
(disjoint and_base/operand_sig: IMAD_R_R_II_R_P, IMAD_R_R_R_II_P,
LDS_R_AURI enter under their own names; canon twins keep their content).
sched_only extended to the
fleet superset (597). Tables now carry aux sections: operand_roles (encoding
semantics), plus cost_model/stallfix for SM103a. sm103a.json refreshed from
the production table (+3 forms, -RET_II). New: derived sm100a.json =
canonical sm103a copy + tcgen05 delta layer (119/119 probe-pair byte parity;
offline corpus validation, no B200 re-run — native campaign parked).
Aux-section provenance scrubbed to declarative form: internal campaign
ids, iteration tags and lab paths removed from cost_model / stallfix /
operand_roles; all silicon-measured values and source-checksum pins
(md5) retained verbatim.
Deferred: 12 internal-only LDGSTS *_AR_dARI harvest forms (no silicon
evidence; O2 registry), canonical-schema hygiene wave.

## Phase 24 — Firmware RE Integration (March 2026)
20 firmware-discovered scheduling IDs, 8 hardware-probed opcodes,
`_meta.hardware_config` with SM registers and SKU profiles.

## Phase 23 — SYNCS investigation (Feb 2026)
SYNCSU/USYNCS not in records.jsonl. Confirmed: Tier A stubs need ptxas Ghidra RE.

## Phase 22 — OPCODE_NNN resolution (Feb 2026)
All 133 stubs are ptxas-internal IDs. Added IMMA.16832.S8.S8 + SAT.

## Phase 21 — HW latency round 3 (Feb 2026)
42 opcodes measured. HADD2=4c, HMUL2=4c, F2F.F16.F32=8c, MUFU.TANH=8c.

## Phase 19 — BGMMA/HGMMA investigation (Feb 2026)
Confirmed: ptxas-internal scheduler aliases, not SM120 SASS opcodes.
`wgmma.mma_async` on SM120a compiles to QMMA, no new opcodes.

## Phase 16 — Encoding audit + latencies (Feb 2026)
882 InsKeys. audit_bits.py found 422 suspicious, fixed 67 entries.
27 instructions measured on RTX 5090: INT/FP=4c, IMAD.HI=6c, MUFU=8c, DFMA=16c.

## Phase 15 — Bug fixes: 869 → 882 InsKeys (Feb 2026)
QMMA E5M2 bit10 fix (97 entries), IMAD.WIDE immediate fix,
HMMA/IMMA modifier bits (42 entries), F2FP new encodings (+13 InsKeys).

## Phase 14 — QMMA hardware validation + SM100A (Feb 2026)
QMMA works on RTX 5090 with EF_CUDA_ACCELERATORS + 128-thread blocks.
SM100A shares SM120 base ISA. tcgen05 instructions cataloged.

## Phase 13 — QMMA and_base bug fix: 868 → 869 (Feb 2026)
Critical: modifier_flag in OPERAND_TYPES caused 174 QMMA entries to encode identically.

## Phase 12 — QMMA/HMMA complete grid: 712 → 868 (Feb 2026)
QMMA.SP.16864 new shape. Complete QMMA grid (non-SP + SP × 3 shapes × types).

## Phase 11 — IMMA/QMMA/HMMA/F2FP/DMMA: 654 → 712 (Feb 2026)
IMMA type variants, QMMA SP variants, HMMA.SP F32, F2FP variants, DMMA shapes.
Critical fix: accumulator register at hi-word bits 0-7 (shift=64).

## Phase 10 — HMMA BF16/TF32/sparse: 640 → 654 (Feb 2026)
19 HMMA variants via hi-bit sweep of 0x023c base.

## Phase 9 — OPCODE_NNN decoded: 605 → 640 (Feb 2026)
OPCODE_NNN stubs are QMMA/HMMA shape variants. QMMA bit map decoded.

## Phase 8 — RTX 5090 GPU execution: 571 → 605 (Feb 2026)
UBLKPF, UBLKRED, USETMAXREG, UTMALDG, UTMACMDFLUSH, CCTL, FENCE, UBLKCP.

## Phase 7 — nvdisasm brute-force: 529 → 571 (Jan 2026)
Key discovery: nvdisasm decodes instructions ptxas can't generate.
Found QMMA, MOVM, DMMA, F2FP, HMMA, I2I, F2IP, I2IP.

## Phase 6 — 523 → 529
IMMA.16816.S8.S8, ELECT, UIADD3.64, MOV_P_R_R, PLOP3.

## Phase 5 — 515 → 523
VIMNMX.S32, I2FP, IDP.4A, UFMUL, LEA.HI.X UR, HFMA2 mixed imm.

## Phase 4 — 499 → 515
SYNCS cluster variants, ISETP/FSETP forms, LOP3, NANOSLEEP, PLOP3, IMNMX.S64.

## Phase 3 — Mercury mapping: 37 → 310/313 (99.0%)
SYNCS = SM120 mbarrier. vabsdiff4 → VIADD.U8x4.

## Phase 2 — Encoding expansion: 284 → 499
215 InsKeys bit-probed on RTX 5090.

## Phase 1 — Scheduling 100%
All 284 original InsKeys scheduled. 57 filled via KNOWN_SCHEDULING.
