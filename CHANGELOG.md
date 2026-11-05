# blackwell-isa Changelog

## tcgen05 LDTM/STTM and SM120 SHF rebuilt from measurement; `stallfix` v4 (November 5, 2026, session T-5's measurement, landed by T-6 on the wave-11 tables)

- **`sm100a.json` / `sm103a.json`: `LDTM_R_II` and `STTM_II_R` rebuilt** — 96 measured groups each
  (`cubit tools/probes/re_sm100_ldtm_sttm.py`, nvdisasm 13.1 `-b SM100` and `-b SM103`, 576 form + 80
  lattice words, text-identical on both targets; provenance `ldtm-sttm-sm100-2026-11-05`). The
  `16dp32bit_t16_t31` x2 / x8 / x32 harvest rows had no uniform-register field (the sample's URZ = 0xff
  half-baked as `0xc0` in `and_base`), a 2-3-bit register field and a 32-bit offset field overlapping the
  UR slot — the five tcgen05 words of the 2026-11-03 B300 strict round trip that re-encoded to a different
  word (cubit KNOWN_ISSUES V5 (a)): cubit printed `tmem[URZ+0x20]` and re-encoded `tmem[UR192+0x20]`.
  Measured layout: shape enum bits {87, 82, 81} (`16dp128bit` 0, `16dp256bit` 1, `32dp32bit` 2 = no
  modifier, `16dp64bit` 3, `16dp32bit_t0_t15` 4, `16dp32bit_t16_t31` 5; 6/7 INVALID), count `x2^n` at
  [85:83], bit 80 = `PACK16BIT` (LDTM) / `EXPAND16BIT` (STTM), 8-bit register (LDTM Rd [23:16], STTM Ra
  [39:32]) and 8-bit UR (LDTM [39:32], STTM [71:64]; 255 = URZ, printed `tmem[URZ]` with offset 0 and
  elided with one), the tmem offset [63:40] (24 bits; its high byte [79:72] stays outside the rows), bit 91
  = frame; the guard is **uniform** (`@UP0 LDTM …`). Variants 22 199 → 22 340 on both tables; keys unchanged.
- **`sm120.json`: the `SHF` family rebuilt** (`cubit tools/probes/re_sm120_shf.py`, 380 form + 78 lattice
  words, provenance `shf-sm120-2026-11-05`): the 40 dotted harvest keys (`SHF.L.U32_R_R_II_R`,
  `SHF.R.S32.HI_R_R_II_R_II_II_?`, … — one sample's modifiers baked, truncated fields; `SHF.L.U32_R_R_II_R`'s
  own canonical word printed `SHF.L.S64` for nvdisasm) are gone and the five generic keys, which mixed the
  opcodes of different forms, carry 32 measured groups each: forms 0x219 `Rd, Ra, Rb, Rc`, 0x419 `Rd, Ra,
  Rc, imm32` (the immediate is the fourth operand), 0x819 `Rd, Ra, imm32, Rc`, 0xc19 `Rd, Ra, URb, Rc`
  and 0xe19 `Rd, Ra, Rc, URb` (frame bit 91 on the UR forms); modifiers bit 73 unsigned / 74 32-bit
  (`U32` = both set, `S64` = both clear) / 75 `.W` / 76 `.R` / 80 `.HI`; bits 72, 77-79, 81-90 text-inert
  (outside the rows). The era alias `SHF.L.U32_P0_R_R_II_R` now points at `SHF_R_R_II_R`. Keys 16 143 →
  16 103, variants 26 612 → 26 656. (The wave-11 merge had touched three of the dotted keys — metadata only;
  they are among the 40 removed.)
- **`sm120.json` `stallfix` v4:** `guard_d0_forbid` false, **`floor_guard_d0` 13**, `cap_stall` 15,
  `legacy_stall_risk_from` 15. The v3 sweep ended at S11 (all wrong) and called a predicate producer directly
  followed by its `@P` guard consumer a violation; harness v4 (session T-5, 2026-11-05, `rtx` = 2× RTX 5090 /
  GB202, 98 kernels: the `guard` path swept to S15 and a new `guardbra` path `ISETP → @P BRA`) measures
  **S12 still wrong (618 880 / 655 360 threads), S13 / S14 / S15 clean** on the data consumer and on the
  guarded branch alike (d1 = 11, d2 = 9 as before). cubit's own `schedule()` writes S14 before a guarded
  branch (one above the floor); session V's silicon-validated kernel uses S14 at its 7 guard-d0 sites and was
  refused by v3. Confirmation on the RTX PRO 6000 Server Edition (the production SKU) is still owed — every
  other floor measured on both SKUs agreed field by field.

## Wave 11 (lane October stack) merged; ATOMG `.EL` plain form rebuilt from measurement (November 5, 2026)

- **Wave 11 integration (`e9b0808`, `e99598a`; coldface, B300 host, 07:33–07:39Z):** the lane lineage of
  30 table-only commits (BUG-477..523) merged onto `bb11659` with `merge_tables.py` (3-way at key →
  mod_group → field); the 26 distinct SM120 conflicts were adjudicated by measurement (each side's canonical
  word executed on an RTX PRO 6000 sm_120, driver 580.178.04, 96 probes; field geometry driven to 1 and
  to max; no baked payload in a declared operand slot): 17 same-word metadata conflicts resolved to the
  richer field set, 5 to main (`FADD_R_R_UR` SAT/RZ — the lane's 0x0221 rows trap from UR15, the 0x0e21
  uniform form runs for UR0..UR63; `ENDCOLLECTIVE_P`; `FCHK_P_R_FI` baked payload; `UIADD3.64` 4/5-bit
  ureg fields), 4 deletions kept (`ENDCOLLECTIVE_?_?`, the `F2FP.RELU…_P_R_R_R` phantom = `HMMA.16816.F16`
  for nvdisasm, both `WARPSYNC.COLLECTIVE _?` keys trap), 0 to the lane. On top: the IDP type law
  (`4A,U8,U8` row on sm120 / sm100a / sm103a; b73/b74 select U8/S8, b76 = 2A form, b75 = `-Rc`; heals 2 716
  vendor words), and the `claim_forbid` exclusions on ctrl[109:105] (15 sm120 + 17 sm121a rows) and
  reuse[124:122] (47 F2FP/HMMA/DMMA PACK_AB rows) removed: silicon executes those control values
  (96/96 IMAD / MOV.64 / MUFU; 8/8 F2FP.PACK_AB) while nvdisasm only refuses to render them. Sizes:
  sm120 16 107 → 16 143 keys / 20 894 → 26 612 variants (5 692 of the 5 718 new variants are the
  modifier lattices of `ATOM_P_R_dARI_R` and `ATOMG_P_R_dARI_R`, 46 → 2 886 groups each), sm103a
  14 879 → 14 902 / 16 451 → 22 199, sm100a 14 879 → 14 902 / 16 455 → 22 199, sm121a 23 825 → 23 850 /
  26 381 → 26 406. The lane's census: sm120 238 848 / 238 848 (578 vendor cubins), `validate_isa`
  0 errors × 4. **The new rows carry no `_src`** — their measurement record is the lane's own report,
  not yet tagged in the tables (listed under "Known issues"). The merged tables were written
  without the trailing newline of the canonical `json.dumps(indent=1)` form; restored in this commit for
  the three tables below.
- **`ATOMG.E.{ADD,OR}.EL.STRONG.GPU_P_R_ARURI_R` rebuilt on sm120, sm103a, sm100a** (cubit
  `tools/probes/re_sm120_atomg_el_aruri.py`, nvdisasm 13.1 `-b SM120` / `-b SM103` / `-b SM100`
  text-identical, 98 form + 25 lattice words per target; provenance `atomg-el-aruri-2026-11-05`). The
  sm103a / sm100a tables keyed the plain `[Ra.U32+URb(+off)]` word as `…_P_R_dARI_R` (rt98 era,
  2026-08-21): cubit printed `desc[UR6][RZ.64]` for a word nvdisasm reads as `[RZ.U32+UR6]`, and since
  wave 11 carries the real `.EL` descriptor rows (`ATOMG_P_R_dARI_R`, bits 76 + 63) that text re-encoded
  to a different word (cubit `tests/b4fill2_rows.rs` red on the merged tree). Measured layout: guard
  [15:12], Pd [83:81], Rd [23:16], Ra [31:24], Rb [39:32], URb [71:64] (8 bits, 255 = URZ), offset
  [62:40] **signed 23-bit** (bit 62 → `+-0x400000`); bit 63 is the address width (`[Ra.64+URb]`, another
  form) and bit 76 the descriptor form — both `claim_forbid` on the rows (the sm120 rows had a 24-bit
  offset field that swallowed bit 63, a 6-bit UR field and no Pd field). Keys / variant counts unchanged.

## SM121a text parity to 100 %; SM120 `stallfix` r2ur floor measured (November 4, 2026, session T-4)

The GB10 census of 2026-11-04 left 60 SM121a text differences in shared families whose SM120 rows
already print the vendor text plus two SM121a-only harvest rows. `blackwell-isa-internal/tools/t4_sm121a_parity.py`
adopts the measured SM120 rows for the shared keys (`sm121a.json` is delta-driven vs SM120, and
SM121a ≡ SM120 for these opcodes — 0 decode differences over 91 927 words):

- `R2P`, `IMNMX_P_P_R_R_UR_P_P`, `SEL_R_R_II_P` (the 33-bit `.64` immediate → `-0x1` glyph), `REDUX_UR_R`
  (the duplicate `.AND` op=0 group gone): the SM120 rows adopted verbatim.
- `BAR`: the era rows `BAR_II_II` / `BAR_II_P` / `BAR_R_R` (which baked a spurious 26-bit second
  immediate → `BAR.SYNC.DEFER_BLOCKING 0x0, 0x0`) dropped; SM120's clean per-form `BAR.*` keys adopted.
- `DEPBAR`: the harvest rows that dropped the `SBn` scoreboard token replaced by SM120's `DEPBAR_II_II`.
- `UIMNMX.U64_UP_UP_UR_UR_II_UP_UP`: the scrambled harvest row rebuilt from the SM120 U64 register-form
  layout (op5 = 32-bit immediate, trailing uniform-predicate negation at bits 80/90).

The rest are cubit printer facts shared by SM120 (`ATOMS`/`ARRIVES` `[Rm+URn]` glyph now fires for the
whole sm_120 encoding family incl. sm_121a; the `F2IP` modifier order `.U8.F32.TRUNC.NTZ`; the `IMMA`
`.ROW`/`.COL` matrix-layout print). Parity on the 373 recorded census-mismatch words: **372 / 373** —
the one residue is `LEPC` (a PC-relative address the isolated-word census renders at address 0; it is
byte-identical to the vendor in a full disassembly). Re-run of the full GB10 census: see cubit V3.

`sm120.json` `stallfix`: `floor_r2ur_d0` 11 → **4** (== `floor_global`), measured 2026-11-04 on the test
host (4× RTX PRO 6000 Server Edition, harness v3 r2ur kernels with warp-uniform inputs: `model_r2ur`
PASS, 26 floors PASS on every path). R2UR broadcasts one lane; the v2 model that summed per-thread
values was wrong, now corrected. cubit KNOWN_ISSUES A3 closed in full.

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
