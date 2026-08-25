# blackwell-isa Changelog

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
