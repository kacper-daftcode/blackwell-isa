## November 9, 2026 (sessions T-23 / T-24 / T-25): `sm121a.json` / `sm103a.json` / `sm100a.json` — the long tails verified row by row on `-b SM121` / `SM103` / `SM100`; rows without `_src` sm121a 2 654 → 811, sm103a / sm100a 893 → 357; the junk keys that printed vendor-nonexistent text deleted, the sm121a LEA family rebuilt, the sm103a / sm100a LDG UR field widened

`sm121a.json` `6428595` + `ad7804a` + `64db08b` + `daac080`, `sm103a.json` `ddac54d` + `64db08b` + `04bcb53`, `sm100a.json` `21fc69a` + `64db08b` +
`04bcb53` (`sm120.json` unchanged). **The per-row passes (T-23):** cubit's `tools/probes/re_sm120_rows.py --arch SM121 / SM103 / SM100` — the
canonical word of every harvest row and every single-bit flip of the 128-bit word named by nvdisasm 13.1, diffed token by token, cubit's
decode / print / re-encode of every word, the table's own corpus words as witnesses (the GB10 census for sm121a, the B300 publish corpus for
sm103a / sm100a) — over the eight families memory / atomic / tensor / half / intalu / uniform / misc / control: sm121a 1 185 rows measured,
511 shadowed harvest keys retired as `_meta.key_aliases` clones, 52 unreachable rows deleted (2 654 → 877); sm103a / sm100a 529 rows measured
each, 3 shadowed keys aliased, 1 dead row deleted (893 → 358). A measured row has its fields for its mask and fails closed on every other bit;
the rows left are named in cubit's fixtures (`tests/{sm121a,sm103a,sm100a}_rows_<family>.rs`). **T-24's correction** (`ad7804a`): the
`HFMA2_R_R_II_II_R` and `F2FP.BF16.F32.PACK_AB_P_R_R_R` alias retirements reverted — the targets could not decode the retired keys' words (the
II-encoded immediates fell to a field-less dotted row and printed `0x0, 0x0` for `1, 1`). **T-25's fixes, each with its measurement in the
commit:** on sm121a six junk harvest keys deleted whose rows printed vendor-nonexistent text for the measured rows' text-inert neighbours
(`LDS.128_R_AR` — `LDS.128 R91, ?AR` for nvdisasm's `LDS.U8 R91, [R10]` —, `UIADD3_UR_UP_UP_UR_UR_R_II_P`, `HADD2_R_R_II_II_II_II_?`, `MOV_P0_R_II`
(a UMOV word), `QMMA.16816.F16.E4M3.E4M3_P_R_R_R_R` (a MOVM word), `HFMA2_R_R_FI_FI_R` (31 rows at the illegal opcode 0x11, claiming the LEA
lattice's illegal words); 0 GB10 witnesses each); **the LEA family rebuilt from measurement on `-b SM121`** (cubit `tools/probes/re_sm120_lea.py
--arch SM121`: 615 form + 209 lattice words, every text identical to the SM120 fixture's — 24 keys / 36 rows for 23 harvest keys / 58 rows,
among them the era `LEA_P0_R_R_R_II` whose words nvdisasm prints `LEA R22, R18, -R19, 0x1`); the `LDG_P_R_ARI_P` / `LDG_R_ARI_P` `.64` rows
measured with the `_P_R_` claim exclusion ([83:81] = 7 belongs to the plain key — without it they printed `LDG.E.EL.64 PT, …`). On sm103a /
sm100a: the plain `LDG_R_ARURI` row's **UR field 6 → 8 bits** (nvdisasm `0x40 → UR64`, `0x80 → UR128`; cubit printed `UR0` for `UR64` and the
word decoded through the `E,LTC128B` sibling) and the row measured; the `LDG_P_R_dARI` rows re-measured on the table's own target (they printed
a phantom `PT,` for a measured neighbour); **`IMAD_R_R_R_R` `U32` added** (the plain `IMAD.U32 Rd, Ra, Rb, Rc` had no row on either table — cubit
minted bit 73 on the measured plain row and its decoder, fail-closed there, rejected the word); `REDG_P_dARI_R` restored as the key-level
`encode_only` retention key (cubit BUG-128), its 13 rows `_src: "modelled"` (sm121a 909 → 811, sm103a / sm100a 358 → 357). The GB10 census
(91 927 unique words) and the B300 publish corpus (85 721 / 85 792 words, 296 / 328 cubins strict-exact) are unchanged on the new tables;
cubit's whole production census (249 929 words) identical. Known, for the next pass: the row notes T-23 wrote on the three tables read
`-b SM120; session G run 5` where the measurement was `-b SM121` / `SM103` / `SM100` (the probe's note writer, fixed for T-25's rows); the forms
the measured tables lack that cubit's battery pins as holes (`STG.E.U8 [R.64+UR]` on the three tables, the predicated `.U32` ARURI loads on
sm121a); the 17 / 17 / 113 harvest siblings on sm103a / sm100a / sm121a that still print their own modifier for a measured neighbour's inert bit.

## November 8, 2026 (session T-23): the four tables rewritten in the compact one-row-per-line layout with the provenance written once per level — `sm121a.json` 87.0 → 32.8 MiB, `sm120.json` 73.6 → 28.0, `sm103a.json` / `sm100a.json` 67.8 → 23.7 / 23.6 (no row changed)

All four tables. **Why:** `sm121a.json` had reached 87 MiB after the ATOM / ATOMG lattice of T-22 (GitHub refuses files over 100 MiB;
the next sm121a lattices — the `.64` twins, the CAS forms, the descriptor-form lattices — would not have fit). Measured composition
at `3fb3b82`: the measurement note repeated on every row of a lattice (`_note`, 24.0 MiB on sm121a), the per-field `_src` tag equal
to the row's (6.6 MiB), the `indent=1` whitespace (24 MiB). **What changed (R's decision on cubit's NEEDS 61, options (a) + (b)):**
(a) provenance is written once per level — a lattice family's law lives in `_meta.provenance.sources[<tag>].note` when every row of
the tag shares it (26 / 16 / 16 / 13 tags on sm120 / sm103a / sm100a / sm121a), on `instructions.<key>._note` when every row of the key
shares it, and on the row only where the row's own note differs; `instructions.<key>._src` is set on every key whose rows all carry the
same tag (906 / 183 / 184 / 325 keys; the rows keep their own `_src`); a field carries `_src` only when measured by another tag than its
row (4 647 / 5 278 / 5 306 / 5 366 fields keep one). (b) the file layout: compact JSON, **one encoding row per line** (a diff of the
table reads row by row), `_meta` and the small sections indented. **Nothing else changed**: for every one of the 38 838 / 35 876 /
35 870 / 50 444 rows the encoding record (`and_base`, `variable_mask`, `fields`, `count`, `claim_forbid`, …) and the resolved provenance
(tag, note, per-field tag) are identical before and after (cubit `T/t23/probe/fmt/equiv_check.py`: 0 differences, the pass idempotent);
rows without `_src` unchanged at 7 / 893 / 893 / 2 654. The legend (`_meta.provenance.legend`) states the resolution order (row →
key → tag); cubit's `tools/tablefmt.py` is the canonical reader / writer (`load_table` / `save_table` / `row_src` / `row_note`), the
probes' `apply` write through it, and cubit's `tools/sync_table.py --validate-only` refuses a table in any other layout or with
un-hoisted provenance. The HTML references render the key's tag and the family's note once per key.

## November 8, 2026 (session T-22, batch 9): `sm103a.json` / `sm100a.json` / `sm121a.json` — the UIADD3 `.64` groups and `.X` rows measured on `-b SM103` / `SM100` / `SM121` (the B300 corpus' last `UIADD3.64` holes, the GB10 census' 16-word residue, bug459a's overprint)

`sm103a.json` / `sm100a.json` / `sm121a.json` `fddd1b3` + `3fb3b82`. The `64` groups of the 64-bit uniform add's UR form (opcode 0x297) and
immediate form (0x897) are sm120's measured rows (`UIADD3.64_UR_UP_UP_UR_UR_UR` dsv41-sm120-2026-10-22, the `64` group of
`UIADD3_UR_UP_UP_UR_II_UR` rows-uniform-2026-11-07: guard, URd, the two carry-out UPs at [83:81] / [86:84], URa with `-URa` 72, URb
with `-URb` 63 — the immediate form: imm32 [63:32] —, URc with `-URc` 75) placed as the `64` groups of the plain keys of the three
tables and re-measured there with cubit's `tools/probes/re_sm120_rows.py --family uniform --arch SM103` / `SM100` / `SM121`
(nvdisasm 13.1, the whole 128-bit circumference of every row, the corpus words as witnesses — sm103a 6 + 3, sm100a 3, sm121a 19 + 16);
the plain `""` rows of the two keys were measured on the way (4 rows per table). Before: sm103a / sm100a carried a junk `64` row on
the UR form (fields at shifts 6 / 69 / 77, a two-bit mask — it decoded nothing) and no `64` group on the immediate form, so the
publish corpus' 9 + 3 `UIADD3.64` words were holes; sm121a's `64` group of the immediate form was mis-declared (no sign fields, a
reuse bit) and its 16 GB10 words re-encoded to other registers. sm121a's three dotted `UIADD3.64_*` harvest keys — whole-key twins
(the `_II_II` one was the harvest's mis-tokenised URZ) — retired as `_meta.key_aliases`. Corpora: the B300 publish corpus **85 707 →
85 721 / 85 792 words, 288 → 296 / 328 cubins strict-exact** (sm103a 3 959 / 3 962 decoded, sm100a every word); the GB10 census
91 927 / 91 927 decode, **91 926 / 91 927 re-encode** (the one left is cubit's BUG-037 alignment refusal of a cuBLASLt `HMMA.1688.F32`
word). sm103a / sm100a 898 → 895 rows without `_src`, sm121a 2 663 → 2 656. **`3fb3b82`: the `.X` rows measured too** — once the plain rows
fail closed on their dead bits [90:77] (the measured convention), the harvest `UIADD3_…_UP_UP` `.X` rows were found claiming those
words through the sign-tolerant fallback: `UP15, !UPT` for the vendor's `!UPT, !UPT` on sm103a / sm100a (a 4-bit carry-in window
where the vendor has 3 bits + a sign), `!UPT, !UP6` on sm121a — cubit's bug459a had pinned the overprint as "standing" in 2026-09;
the two keys measured on the three targets with sm120's field lists (the carry-in predicates [79:77] + inv 80 and [89:87] + inv 90),
the overprint ends on the four tables; sm103a / sm100a 895 → **893**, sm121a 2 656 → **2 654**. Left for the uniform family's next
pass (counted by cubit's test): the sm121a junk key `UIADD3_UR_UP_UP_UR_UR_R_II_P` (it claims the UR form's words with bits in the
unused immediate window), sm103a / sm100a's harvest `ULOP3_UP_UR_UR_II_UR_II_UP` `LUT` row (a 6-bit `pred` and a 9-bit LUT where
sm120's measured row has 3 + 8 bits and the `.PAND` modifier: the ULOP3 immediate word renders vendor-false). The three HTML
references regenerated.

## November 8, 2026 (session T-22, batch 8): `sm121a.json` — ATOM / ATOMG in the plain uniform-indexed address form rebuilt from measurement on `-b SM121`

`sm121a.json` `ce51005`. cubit's new `tools/probes/re_sm120_atom_aruri.py` (modelled on G's `re_sm120_redg.py` and on T-6's
`re_sm120_atomg_el_aruri.py`, which measured the operand layout of this form on sm120) run with `--arch SM121 --witnesses <the GB10
census corpus>` (nvdisasm 13.1): all 18 398 form words print the text the law predicts and the 55 lattice words are illegal, a
vendor `INVALIDn` glyph, an inert-bit twin or another form's word (`-b SM120` / `SM103` / `SM100` print the same text for the
canonical words). The law: guard [15:12], Rd [23:16], Ra [31:24], Rb [39:32], the signed 23-bit byte offset [62:40], URb [71:64],
`.E` 72, the type [75:73] (U32 / S32 / 64 / S64 / **128**; 5..7 `INVALIDn`), the 16-value strength / scope / PRIVATE enum [80:77],
Pd [83:81], cache [86:84] (EF / none / EL / LU / EU / NA; 6 / 7 `INVALIDn`), the op [90:87] (ADD, MIN, MAX, INC, DEC, AND, OR, XOR,
EXCH; **SAFEADD on ATOMG only** — `INVALID9` on ATOM —; 10..15 `INVALIDn`), the uniform-indexed form 91; [95:92] text-inert and
fail-closed; bit 63 (the `.64` base) and bit 76 (the descriptor form) are other keys and `claim_forbid`. `ATOM_P_R_ARURI_R` /
`ATOMG_P_R_ARURI_R` created with 8 640 + 9 600 measured groups (2 (`.E`) × 9 / 10 ops × 5 types × 6 cache × 16 scope); the 3 456 +
3 456 dotted one-group keys of the GB10 line's 2026-08/09 rides (the `.E` plane of 9 ops × 4 types only — no no-`.E` words, no
`.128`, no SAFEADD) retired as `_meta.key_aliases` of the plain key of their opcode (every retired key's tokens name a group of
the target). The 15 ATOMG words of the GB10 census corpus are the descriptor form, outside this form: 0 witnesses; the census is
unchanged (91 927 / 91 927 decode, 1 `LEPC` mismatch, 91 910 re-encode). Rows 39 119 → 50 447, **rows without `_src` 9 575 →
2 663**, forms 9 063 → 2 153, aliases 14 835 → 21 747. **The rows carry a one-line `_note` pointing at the family's note in
`_meta.provenance.sources` and no per-field `_src`: `sm121a.json` is 87 MiB with this lattice, within 13 MiB of GitHub's 100 MiB
file limit — the one-row-per-group schema needs the owner's decision before the next lattice (cubit `T/NEEDS_OWNER.md` 61).**
cubit side: the printer prints the atomics' U32 uniform-indexed form with the width taken from the key (bit 90 is the top bit of
the 4-bit op field — the generic width rule printed a false `.64` on every EXCH / SAFEADD word, and the dotted `ATOM.E.*` keys of
this table printed the descriptor world's `desc[UR][R.64]` for a `[R.U32+UR]` word) and `SAFEADD` in the op slot of the modifier
order. The SM121a HTML reference regenerated (71 MiB).

## November 8, 2026 (sessions T-21 / T-22, batch 7 + NEEDS 59): `sm121a.json` / `sm103a.json` / `sm100a.json` — the float conversions (F2I / I2F / FRND / MUFU) and the FP32 ALU (FADD / FMUL / FFMA / FMNMX) rebuilt from measurement on `-b SM121` / `SM103` / `SM100`; VIMNMX3 measured on sm103a / sm100a; sm121a's junk key `MUFU_R_L` retired

`sm121a.json` / `sm103a.json` / `sm100a.json` `3574c5f` (T-21, batch 7) + `15435a0` (T-22, NEEDS 59). cubit's `tools/probes/re_sm120_conv.py`
and `re_sm120_fp32_alu.py` (G's batch 6 / 8 probes of 2026-11-07) run with `--arch SM121` / `SM103` / `SM100` (nvdisasm 13.1): every word
obeys the sm120 law on the three targets — the conversions 2 494 form + 18 glyph + 547 lattice words, the FP32 ALU 1 636 form + 30 glyph
+ 492 lattice + 16 neighbour words, per target. The 12 conversion keys (`F2I_R_{R,UR,FI}` with 512 groups each — round × NTZ × FTZ ×
destination type × source type, the immediate typed by the source lane —, `I2F_*`, `FRND_*`, `MUFU_R_{R,UR,FI}`) and the 14 FP32 ALU plain
keys carry the measured lattices; the dotted and mis-tokenised harvest keys (sm121a: 213 F2I, 19 I2F, 8 FRND, 9 MUFU, the integer-spelling
`*_II` twins, `F2I.TRUNC.NTZ_R_R_II_?`, `I2F.U32_R_R_R`, `MUFU.RCP_P_R_R`, …) are `_meta.key_aliases` of the plain key their `and_base`
names; the rows of another measurement outside these lattices keep their tag (T-14's sm103a MUFU packed-half lanes). The sm103a
`MUFU_R_FI` / `FMUL_R_R_FI` rows carry the immediate as an f32 field instead of the 2026-08 harvest's baked corpus constant
(`MUFU.RSQ Rn, -QNAN`: the vendor glyph is the display print, the lossless `0xffc00000F` literal is what re-encodes — cubit
KNOWN_ISSUES A8). **NEEDS 59, found by the new lattice tests:** (a) the sm103a / sm100a `VIMNMX3_R_R_R_R_P` ("" and `RELU`) and
`VIMNMX3_R_R_UR_R_P` harvest rows declared the fifth operand's predicate as a 4-bit field at shift 1 — inside the opcode nibble —
so the register form claimed FMNMX words (`FMNMX R4, R2, R3, PT` decoded as `VIMNMX3 R4, R2, R3, R1, P4`; the PLOP3 class of batch 6);
the form is real (`@P0 VIMNMX3 R0, R0, R0, R0, PT` for nvdisasm, harvest count 1 139 / 944), so the three rows were measured with
`re_sm120_rows.py --family intalu --arch SM103` / `SM100` (the predicate at [89:87] with its sign at 90 like IADD3's; bits 72 / 73 / 76 =
`.U32` / `.S16x2` / `.RELU`, other groups' words — the type lattice of VIMNMX3 stays a hole; 0 B300 corpus witnesses). (b) sm121a's
label-typed junk key `MUFU_R_L` (8 groups whose canonical words are the plain `MUFU.<fn> R0, R0` of the measured `MUFU_R_R`; its mask
tolerated the half-type bits and claimed `MUFU.INVALID6.F16`) retired as an alias of `MUFU_R_R` (0 GB10 witnesses). sm121a rows 36 884 →
39 119, **rows without `_src` 10 000 → 9 575**, forms 9 334 → 9 063, aliases 14 561 → 14 835; sm103a 33 532 → 35 875 (**1 280 → 898**),
sm100a 33 526 → 35 869 (**1 280 → 898**). Gates: the B300 publish corpus **85 704 → 85 707 / 85 792** (the `I2F.U16[.RZ]` ×2 and
`FMUL.RZ` ×1 holes closed), the GB10 census 91 927 / 91 927 with the strict residue **40 → 17** (the 23 `FFMA_R_R_R_II` words re-encode:
the measured `FFMA_R_R_R_FI` row carries the `.reuse` field of the second register the harvest row lacked; the 17 left: the harvest
`UIADD3` `64` group ×16, one BUG-037 refusal). The three HTML references regenerated.

## November 8, 2026 (session T-21, batch 6): `sm121a.json` / `sm103a.json` / `sm100a.json` — ISETP / UISETP, FSETP / DSETP (sm103a / sm100a) and PLOP3 / P2R / R2P rebuilt from measurement on `-b SM121` / `SM103` / `SM100`

`sm121a.json` / `sm103a.json` / `sm100a.json` `991137d` + `0525e95`. cubit's `tools/probes/re_sm120_isetp.py` (G's batch 2 of
2026-11-06) run with `--arch SM121 --witnesses <the GB10 census corpus>`: every one of the 1 944 form words prints the SM120 text —
the 64-bit compare (`.S64` / `.U64`, bit 80) included —, the 192 lattice words behave the same, and all 3 155 ISETP / UISETP words
of the GB10 corpus (579 of them 64-bit compares) obey the law word for word; the ten plain keys rewritten to the 96 measured groups
each, the 224 `INVALID3`-boolean harvest groups (nvdisasm's invalid glyph, a lattice plane) retired, the 146 dotted one-group keys
retired as `_meta.key_aliases` of the plain key of their form. With `--arch SM103` / `SM100`: the SM120 law holds for every word
**except bit 80 — the 64-bit compare does not exist on these targets** (`T/NEEDS_OWNER.md` 46: every nvdisasm 13.x build agrees, no
compiler word of 2 145 B300 cubins sets the bit); the ten keys rewritten to the 48 measured groups each with bit 80 outside the
masks (text-inert, fail-closed) and the four phantom `.S64` / `.U64` harvest groups of `ISETP_P_P_R_UR_P` retired. `re_sm120_fsetp_dsetp.py
--arch SM103` / `SM100` (NEEDS 58): the 1 022 form, 12 glyph and 223 lattice words obey the sm120 law of 2026-11-06 on both targets;
the six keys carry the 432 measured rows — the 2026-08 donors' reading of the boolean bit 75 as `.OR` (the vendor prints `.XOR`) and
the `|Rb|` baked into `DSETP_P_P_R_R_P` `LTU,OR` are gone. `re_sm120_plop3_p2r_r2p.py --arch SM103` / `SM100` / `SM121`: 202 form + 158
lattice words obey the law; the four keys rewritten to the 8 measured rows, `R2P_II_R` created, sm121a's 9 junk keys aliased. **Found on
the way: the sm103a / sm100a harvest `PLOP3_P_P_P_P_P_II_II` row carried a 4-bit "pred" field at bit 2 — inside the opcode nibble — and
claimed every word of opcodes 0x80c..0x83c that fit its remaining bits** (a bit-80 ISETP immediate word decoded as `PLOP3.LUT P0, P3,
PT, !P0, PT, 0x10, 0x30200`; its encoder wrote the fifth predicate into the opcode, so cubit's 2026-08 "PLOP3" word for `PLOP3.LUT P0,
P1, P2, P3, P4, 0x80, 0x8` was `R2P PR, R0.B3, 0x0` for nvdisasm). sm121a rows 37 077 → 36 884, **rows without `_src` 11 160 → 10 000**,
forms 9 488 → 9 334, aliases 14 406 → 14 561; sm103a 32 916 → 33 532 (**1 584 → 1 280**), sm100a 32 910 → 33 526 (**1 584 → 1 280**).
Gates: the GB10 census 91 927 / 91 927 (287 ISETP corpus words moved to the plain keys, texts unchanged); the B300 corpus unchanged at
85 704 / 85 792, 0 words with a changed text or key. The three HTML references regenerated.

## November 8, 2026 (session T-21, batch 5): `sm121a.json` — LDG.256 (three address forms), STG.256 and the REDG uniform-indexed forms rebuilt from measurement on `-b SM121`

`sm121a.json` `f254702`. cubit's `tools/probes/re_sm120_ldg256.py --form dARI / ARURI64 / ARURI`, `re_sm120_stg256.py` and
`re_sm120_redg.py` (G's batches 10–14 of 2026-11-07) run with `--arch SM121 --witnesses <the GB10 census corpus>` (nvdisasm
13.1): all five runs passed — every form word prints the same text as on `-b SM120` (LDG 4 888 + 81, 4 952 + 17, 4 952 + 17;
STG 2 140 + 93; REDG 9 334 + 58 form + lattice words), and every LDG.256 (99) and STG.256 (68) word of the GB10 census corpus
(gx4, 91 927 distinct words, their own control codes) obeys the law word for word (the corpus' 12 REDG words are the
descriptor form, outside the two uniform-indexed forms). The plain keys `LDG_R_R_{dARI,ARURI64,ARURI}[_II][_P]` (1 152 groups
each), `STG_{dARI,ARURI,ARURI64}_R_R[_II]` (288 each) and `REDG_ARURI_R` / `REDG_ARURI64_R` (6 144 / 3 072) are created — this
table had none of them, only the dotted one-group keys of 2026-08/09 (6 985 LDG, 1 157 STG, 6 144 REDG: the GB10 line's own
BUG-446..476 "rides") — and the 14 286 dotted keys are retired as `_meta.key_aliases` of the plain key of their operand
signature (an alias, not a deletion: 167 corpus words lived on them and moved to the plain keys with their texts unchanged).
The 19 `wave11-integration-2026-11-05` rows of the lane's dotted `LDG.E.NA.ELL2.256.*` keys are groups of the measured plain
keys now (legend 41 → 22), as on the other three tables. Rows 26 595 → 37 077; **rows without `_src` 25 427 → 11 160**; forms
23 754 → 9 488; `key_aliases` 120 → 14 406; `claim_forbid` rows 1 244 → 5 253 (the measured dARI rows carry the RZ-base
`claim_forbid`). Gate: the GB10 census 91 927 / 91 927 decode, text parity 91 926 (the `LEPC` residue), strict re-encode
91 887 (the same 40 words outside these families). The SM121a HTML reference regenerated.

## November 8, 2026 (sessions T-20 / T-21, batch 4): `sm103a.json` / `sm100a.json` — LDG.256 (three address forms) and STG.256 rebuilt from measurement on `-b SM103` / `SM100`

`sm103a.json` / `sm100a.json` `39d0372`. cubit's `tools/probes/re_sm120_ldg256.py --form dARI / ARURI64 / ARURI` and
`tools/probes/re_sm120_stg256.py` (G's batches 10–13 of 2026-11-07) run with `--arch SM103` and `--arch SM100` (nvdisasm 13.1):
all eight runs passed — every form word prints the same text as on `-b SM120` on both targets (LDG dARI 4 888 + 81, ARURI64
4 952 + 17, ARURI 4 952 + 17 and STG 2 140 + 93 form + lattice words per target; the L2 = 3 plane is `INVALID3` on every
target, as G recorded), asserted word by word by the probes. On each table the plain keys `LDG_R_R_{dARI,ARURI64,ARURI}[_II][_P]`
(1 152 groups each) and `STG_{dARI,ARURI,ARURI64}_R_R[_II]` (288 each) are created with the 13 824 + 1 728 measured rows, and
the 7 782 + 1 164 dotted one-group keys of 2026-09 (`LDG.E.<cache>.<L2>[.LTC].256.<scope>_R_R_<form>[_II][_P]`,
`STG.E.….256.<scope>_<form>_R_R[_II]`, the BUG-446..476 "ride" grafts) are retired as `_meta.key_aliases` of the plain key of
their operand signature; the 23 `wave11-integration-2026-11-05` rows (the lane's dotted `LDG.E.NA.ELL2.256.*` keys) are groups
of the measured plain keys now, as on sm120 since G's batches 10–12 (legend 51 → 28 entries per table). sm103a 8 784 forms /
25 517 rows → 650 / 32 916, sm100a 8 783 / 25 511 → 649 / 32 910; **rows without `_src` 9 714 → 1 584 on both**; `claim_forbid`
rows 6 978 → 10 977 / 6 979 → 10 978 (the measured dARI rows carry the RZ-base `claim_forbid`: `desc[UR][RZ.64]` is illegal).
Gate: the B300 publish corpus strict round trip unchanged at 85 704 / 85 792 words, 288 / 328 cubins (0 witnesses of the retired
keys); cubit's 2026-09 EFL2.256 wave tests re-pinned to the measured doctrine on their sm103a / sm100a legs (the NA.EFL2
descriptor spellings are legal words the vendor names on the three targets — the 2026-09 "dead spelling" pins stood on the
harvest's silence). The SM103a / SM100a HTML references regenerated.

## November 8, 2026 (session T-20, batch 3): `sm103a.json` / `sm100a.json` — the REDG uniform-indexed forms rebuilt from measurement on `-b SM103` / `SM100`, the first batch of the B300 line

`sm103a.json` / `sm100a.json` `6a468a2`. cubit's `tools/probes/re_sm120_redg.py` (G's batch 14 of 2026-11-07) run with
`--arch SM103` and `--arch SM100` (nvdisasm 13.1): every one of the 9 334 form words of the sweep prints the same text as on
`-b SM120` and the 58 lattice words (bit 91 clear, the text-inert bits, the INVALID type / cache planes, the descriptor bit,
the `.64` base without `.E`) behave the same — asserted word by word by the probe. On each table the two plain keys
`REDG_ARURI_R` (6 144 groups) / `REDG_ARURI64_R` (3 072) are created with the measured rows — the harvest had none, only
the 6 144 dotted one-group keys `REDG.E.<op>[.cache][.type].<scope>_{ARURI,ARURI64}_R` of 2026-08/09 — and the 6 144 dotted
keys of each table are retired as `_meta.key_aliases` of the plain key of their operand signature (0 witnesses in the B300
publish corpus; its strict round trip is unchanged at 85 704 / 85 792 words, 288 / 328 cubins). sm103a 14 926 forms /
22 445 rows → 8 784 / 25 517, sm100a 14 925 / 22 439 → 8 783 / 25 511; **rows without `_src` 15 858 → 9 714 on both**;
`claim_forbid` rows unchanged. The SM103a / SM100a HTML references regenerated.

## November 8, 2026 (session T-20, batch 2): `sm121a.json` — FSETP / DSETP rebuilt from measurement on `-b SM121`, the first batch of the GB10 line; a GB10 census regression closed

`sm121a.json` `75e9d20`. cubit's `tools/probes/re_sm120_fsetp_dsetp.py --arch SM121` (nvdisasm 13.1): the sm120 law of
2026-11-06 holds word for word on SM121 — every one of the 1 022 form, 12 glyph and 223 lattice words of the sweep, and all
2 688 FSETP / DSETP words of the GB10 census corpus (gx4, `/root/sparklab/results/q1/corpus`, with their own control codes).
The six keys `FSETP_P_P_R_{R,UR,FI}_P` / `DSETP_P_P_R_{R,UR,FI}_P` carry the 432 measured rows. On this table the harvest had
the plain keys with a third of the groups (33 / 32 / 32 of 96, 25 / 16 / 14 of 48), no `FSETP_P_P_R_UR_P` and 105 dotted
one-group keys `FSETP.<cmp>[.FTZ].<bool>_P_P_R_{R,FI,II}_P` carrying the census words: the missing key created, 312 groups
added, the 105 dotted keys and the same 7 junk keys as on sm120 (`FSETP.*_P_P_R_R_R` ×2, `_P_P_R_R_?`, `_P_P_R_R_P_?` ×2,
`FSETP_P_P_II_R_P`, the integer-spelling twin `FSETP_P_P_R_II_P`) retired as `_meta.key_aliases` of the plain key of their
operand signature (an alias, not a deletion, where the key carried harvest counts). Rows 26 421 → 26 595, rows without
`_src` 25 685 → 25 427, aliases 13 → 120. **Closed with it: since 2026-11-06 the GB10 census had 564 words without a decode**
— every signed FSETP word of the corpus — because the cubit decoder arm of G's batch 3 (FSETP off the sign-tolerant match)
applied to every table while this table's harvest rows declared no sign fields; 91 927 / 91 927 words decode again, text
parity 91 926 / 91 927 (the `LEPC` residue), strict re-encode 91 887 / 91 927 (the 40 words outside the family as before).
The inventories of the three remaining tables are in the README ("Known issues"). The SM121a HTML reference regenerated.

## November 8, 2026 (session T-20): the IMAD `neg@72` field retired — bit 72 is compute-inert on sm_120 silicon (NEEDS 53)

`sm120.json` `27b1272`. The 24 measured non-HI IMAD rows (`IMAD_R_R_R_R` ×3, `_R_R_R_UR` ×3, `_R_R_UR_R` ×3, `_R_R_II_R` ×4,
`_R_R_R_II`, `_R_R_R_R_P` ×2, `_R_R_II_R_P` ×2, `_R_R_UR_R_P`, `IMAD_R_P_R_R_R` / `_R_P_R_II_R` / `_R_P_R_UR_R` `U32,WIDE`,
`_R_P_R_R_R_P` / `_R_P_R_II_R_P` `U32,WIDE,X`) carried a `neg@72` field on the Ra register as cubit's own law (BUG-296: `-Ra`
at bit 72; the T-19 fixture's class `cubit_law` — nvdisasm prints nothing for the bit). **Measured on sm_120 silicon by
session T-19 (test host GPU 0, `pt-needs53`, 2026-11-08 05:19Z, a cubit-assembled 1-thread kernel with a = 0x70000007,
b = 5, c = 3): plain `IMAD` and the same word with bit 72 set both compute a·b+c (0x30000026); the `-Rc` control (bit 75)
computes a·b−c (0x30000020); `IMAD.U32` and its bit-72 word agree.** nvdisasm 13.1 prints nothing for bit 72 on any IMAD
form on `-b SM120` / `SM121` / `SM103` / `SM100` (bits 62 / 63 text-inert as well; 73 = `.U32`, 74 = `.X`, 75 = `-Rc`). The
field goes: `variable_mask` narrows by bit 72 on the 24 rows, `and_base` unchanged, the `_note` of each row names the
measurement; the family re-probed on the applied table (cubit `tools/probes/re_sm120_rows.py --family intalu`, whose
`MANUAL_FIELDS` entry for the law is retired with the field — `RETIRED_FIELDS` is the inverse mechanism): 183 / 183 rows
measured, bit 72 in the text-inert class of every IMAD row. Rows 38 838, rows without `_src` 7, aliases 14 918, `_meta`
unchanged. The whole production census (249 929 distinct words) decodes and re-encodes bit-exact before and after with 0
text changes (0 census words carry bit 72 on IMAD). cubit (same hour): the encoder refuses `-Ra` / `|Ra|` / `-Rb` / `|Rb|`
on IMAD on SM120 instead of minting the inert bit (`-Rc` / `-URc` stay as the measured fields), the decoder fails closed
on the bit-62 / 63 / 72 words (until now the bit-63 neighbour decoded through the T-3 `U32` twin as
`IMAD.U32 R32, |R48|, -R80, R24`), bug296's IMAD control re-pinned. The SM120 HTML reference regenerated at `27b1272`.

## November 8, 2026 (session T-19): the last 111 harvest rows of `sm120.json` closed — field-narrowing pass, renames to the vendor's spelling, 12 junk keys deleted, 6 design rows marked `modelled`; 7 documented rows left

The decisions of T/NEEDS_OWNER.md 55 (R, 03:05Z), measured with cubit's `tools/probes/re_sm120_rows.py` (nvdisasm 13.1, -b SM120)
over the whole 128-bit word; the prep script is blackwell-isa-internal `tools/t19_prep_sm120_20261108.py`. **`sm120.json`: 38 838 rows,
38 831 with `_src` — 7 harvest rows left, every one a documented owner item** (111 at `7f12b68`); forms 1 015 → 979, `_meta.key_aliases`
14 906 → 14 918.

- **71 rows measured with their field lists corrected from the sweep** (the probe's `MANUAL_ROW_FIELDS`): the destination-predicate
  windows of the IMAD P-forms and ULEA narrowed to 3 bits (a destination predicate has no `!` bit); `STS_ARURI_R` ×7 / `STL_AURI_R` ×3
  `sub_ur` 9 → 8 bits (bit 72 is text-inert); IADD3's second carry-out predicate [86:84] declared (`IADD3_R_P_P_R_R_R`, 5 423 census
  witnesses; the `.X` form's 66); `BSSY_B_II`'s target = `imm_shr2` at [63:34] (the harvest had an unnamed 30-bit field);
  `CALL.REL.NOINC_II` on the measured branch-target layout; `WARPSYNC` `ALL` without its unprinted register field; the `.H1` merge
  selector (bit 72, `opmod:H1`) on `F2FP_R_R_R_R` E4M3 / E5M2, `F2FP.F16.F32.MERGE_C_R_R_R`, `F2IP.S8.F32.NTZ` and UF2FP; `LD_R_dARI`
  `E,STRONG,SYS` gains its offset field; `UIADD3` `64` its third UR source; the two ULOP3 LUT rows their LUT / UR / predicate operands;
  UI2F / UI2FP register fields widened to 8 bits; `HFMA2_R_R_R_UR`'s `.reuse` bit 123 belongs to the third register.
- **15 HADD2 / HFMA2 immediate-pair rows measured** (NEEDS 55b): bit 85 (`BF16_V2`) re-types both packed-half immediates together with
  the modifier — the other group's word, a modifier bit (`PAIRED_TYPE_BITS`); the `SAT` / `FTZ` / `FTZ,SAT` rows' immediates typed f16.
- **`F2FP_R_R_R_R`**: the three spellings of one group per E4M3 / E5M2 (the sorted-name row was `encode_only`, the unsorted twin
  decoded) collapsed into the sorted name — 2 759 production words move group, text and re-encode unchanged.
- **The four PT-elided `IMAD.HI` / `R2UR.FILL` rows** (G's NEEDS 11): the plain groups added (`IMAD_R_R_UR_R` `HI`, `IMAD_R_R_UR_R_P`
  `HI,U32,X` / `HI,X`, `R2UR_UR_R` `FILL`; PT baked at [83:81], the ISETP precedent); the `_P_` rows measured too.
- **17 mis-named harvest keys renamed to the vendor's spelling** (0 census witnesses each): `STL_P_ARI_R` → `STL_ARI_R` `EF,U8`;
  `F2IP.S8.F32.NTZ_R_R_R_II` → `_R_R_R_R`; `HSET2_R_R_II_II_P` → `HSET2_R_R_FI_FI_P`; the two UF2FP `_UR_?_?_?` keys →
  `UF2FP_UR_UR_UR_UR`; `UI2F.RP_UR_UR` → `UI2F_UR_UR` `RP`; `UI2FP.F32.S32_UR_UR` → `UI2FP_UR_UR` `F32,S32`; `VIADD.U8x4_R_R_R` →
  `VIADD.32_R_R_R`; `UBLKCP.S.G_AUR_AUR_UR` → `UBLKCP_AURI_AURI_UR` `G,G`; the two `UBLKRED.G.S.ADD` keys → `UBLKRED_AURI_AURI_UR`
  `ADD,G,S` / `ADD,G,S,S32` (bit 74 = the `.S` source code the harvest left clear — nvdisasm printed `INVALID0`);
  `UIADD3.64_UR_UP_UP_UR_II_UR` → `UIADD3_UR_UP_UP_UR_II_UR` `64` (160 production words move key, text unchanged);
  `UIMNMX_P_P_UR_UR_UR_P_P` `S64` → `UIMNMX_UP_UP_UR_UR_UR_UP_UP` `U32`; `CCTL.E.PF1_AR` / `CCTL.E.PML2_AR` → `CCTL_ARI` `PF1` / `PF2`;
  `UCGABAR_ARV` / `UCGABAR_WAIT` group `_ARV` / `_WAIT` → `` (cubit printed the suffix twice).
- **12 keys retired as `_meta.key_aliases`** (twins): the five `CALL.REL.NOINC_*` spellings → `CALL_II`; the five
  `F2FP.*.PACK_AB_P_R_R_R` phantom-predicate keys → `F2FP_R_R_R` / the dotted PACK_AB keys; `IADD.X_R_P_R_R_R` → `IADD_R_P_R_R_P` (a
  one-sample row, no test names it); `ULOP3.LUT_UP_UR_UR_II_UR_II_UP` → `ULOP3_UP_UR_UR_II_UR_II_UP` (the dotted twin of its measured
  `LUT` group); `UI2FP.F32.S32_UR_UR` → `UI2FP_UR_UR` replaces the alias `UI2FP_UR_UR` it pointed at.
- **4 dead rows deleted** (unreachable duplicates of measured rows: `WARPSYNC_R` `ALL`, `CS2R_R_R`, `CCTL.E.PF2_AR`,
  `STSM.8.MT168.2_AR_R`) and **12 junk keys deleted whole** (R's class (d)): the six `IADD.64*` keys, `SEL.64_R_R_R_P`,
  `BAR.RED.POPC.DEFER_BLOCKING_II_II_P` — every canonical word nvdisasm rejects; `BAR.SYNC.DEFER_BLOCKING_P_II` (its words are the
  register form `BAR.SYNC Ra, Rb` no row models — a documented hole now), `MOV_P_R_II` (UMOV words), `VIMNMX.S32_R_R_UR_P` (VIMNMX.U32
  immediate words the measured row fails closed on), `ATOMG.E.ADD.F64.RN.STRONG.GPU_P_R_AR_R` (count 0; an `ATOMG.ADD.EF.F16x2.RN` word).
- **6 design rows marked `_src: "modelled"`** (R's decision; each note names the pinning cubit test): `IADD3.X_R_P_P_R_II_R_P_P` /
  `IADD3_R_P_P_R_II_R` `X` (bug089 / bug012; the sweep's second carry-out predicate added to the decode row), `HFMA2_R_R_R_II_FI` `` (the
  wave-built guard-capable twin key, bug274 / 279 / 285 / 330), `UBLKPF.L2_AUR_UR` (sm120_ublkpf_l2), the `encode_only` retention rows
  `REDUX_UR_R` `AND` (bug193) and `UISETP_UP_UP_UR_UR_UP` `` (bug092). `_meta.provenance.sources.modelled` names the class.
- `UI2F_UR_UR` `RP,U32` (bug241): the mask narrowed to its fields (bit 74 fail-closed) so the `RP` row can decode.

**Whole production census (249 929 words): 249 929 decoded, 249 929 re-encoded bit-exact, 0 text changes** — the only key / group
moves are the two deliberate renames above. The 7 rows left: `F2FP_R_R` ×2, `FLO_R_UR`, `WARPSYNC` `` / `COLLECTIVE`, `WARPSYNC_R`
`COLLECTIVE` (the alias-able groups of mixed keys — a group-level alias is a schema item) and `WARPSYNC.COLLECTIVE_R_II` (the 50-bit
immediate, NEEDS 55c). cubit side: the six `tests/sm120_rows_*.rs` regenerated, `tests/sm120_rows_common.inc` knows the `deleted` and
`modelled` statuses, the printer prints branch targets as full signed values (`BSSY` / `CALL` backward targets printed `-0x…` like
nvdisasm, not wrapped to 32 bits) and the `UF2FP` / `UBLKRED` modifier orders, the encoder maps `-Rn` onto the `inv` bit of the
IADD3 / IADD `.X` forms (BUG-048 kept on the measured rows), 30 bug-wave tests re-pinned to the measured doctrine.

## November 8, 2026 (session T-18, the exceptions phase): 32 whole-key twins of the long-tail named exceptions retired as `_meta.key_aliases`; 111 harvest rows left

The key-naming decisions of G's NEEDS 10–12, decided by the table's existing convention "aliases for twins": 32 harvest
keys whose name does not spell the vendor text of their own canonical word, each a whole-key twin (the single group's
canonical word decodes through an existing family key vendor-true and re-encodes bit for bit; 0 census witnesses),
retired as `_meta.key_aliases`. **`sm120.json`: 38 866 rows, 38 755 with `_src` — 111 harvest rows left** (143 at
`fd736a7`); forms 1 047 → 1 015, `_meta.key_aliases` 14 874 → 14 906. Kept as named exceptions (owner decision — the
project models them deliberately): IADD.X_R_P_R_R_R, IADD3.X_R_P_P_R_II_R_P_P, UBLKPF.L2_AUR_UR; and the four mixed keys
F2FP_R_R / FLO_R_UR / WARPSYNC / WARPSYNC_R. Whole production census (249 929 words) unchanged. The 111 left: 20 live
census-bearing harvest rows (over-wide fields / undeclared bits — a field-narrowing probe), 15 HADD2/HFMA2 bit-85
immediate pairs, and the remaining owner decisions (NEEDS_OWNER 55). cubit side: the five family tests regenerated.

## November 8, 2026 (session T-18): the 192 wave-pinned rows take their measured status on `sm120.json` — the 2026-08/09 bug-wave suites re-pinned to the measured doctrine; 143 harvest rows left

Session G run 5 had verified 192 of the 304 remaining harvest rows field by field but left them harvest because 46 cubit test binaries
(67 tests) of the 2026-08/09 bug waves pinned their harvest structure or the tolerant decode of a text-inert bit. T-18 lifts the pin and
re-pins those suites to the measured doctrine (a measured row's mask is its fields, fail closed on the text-inert bits — the convention
since 2026-11-06). **`sm120.json`: 38 898 rows, 38 755 with `_src` — 143 harvest rows left** (304 at `37d030f`); forms 1 083 → 1 047,
`_meta.key_aliases` 14 839 → 14 874. Of the 192: **93 measured** (`rows-half` 43 → 51, `rows-intalu` 164 → 167, `rows-memory` 199 → 270,
`rows-atomic` 57 → 68), **65 aliased** (the HFMA2 integer-immediate spellings `*_R_R_II_II_R*` → `*_R_R_FI_FI_R*`, the HADD2.F32 dotted
twins, `F2F.F64.F32_R_R_R` → `F2F.F64.F32_R_R`, the LDC junk, `ATOMS_R_ARI` → `ATOMS_R_ARURI`), **3 deleted** (unreachable;
`F2IP.U8.F32.NTZ_P_R_R_R_R` is an HMMA.1688.F16 word under a conversion key), **31 stay harvest** (their field-by-field probe finds a real
problem — the HADD2 / HFMA2 bit-85 immediate-pair rows, the STS / STL over-wide `sub_ur`, F2FP's `.H1` — and they fold into the named
exceptions). The 143 left = those 31 + the 112 named exceptions (NEEDS_OWNER). **Whole production census (249 929 words) before → after:
0 text / re-encode / key-group changes** — the measurement only narrows masks on words with no census witness. cubit side: the four
`tests/sm120_rows_{half,intalu,memory,atomic}.rs` + the 46 re-pinned bug-wave binaries (`24cdfcc`).

## November 7, 2026 (session T-17 / session G run 5, batch 16): the long tail verified row by row on `sm120.json` — half-precision, integer ALU, uniform datapath, control, memory, atomics; 304 harvest rows left (192 of them verified, awaiting cubit's test re-pins)

Session G measures, session T re-measures every fixture word with nvdisasm 13.1, checks the whole production census before and after,
gates and pushes. **`sm120.json`: 38 966 rows, 38 662 with `_src` — 304 harvest rows left** (1 034 of 39 125 at `3dde6e1`, 18 841 of
26 598 at `c896e8f`); forms 1 231 → 1 083, `_meta.key_aliases` 14 691 → 14 839.

- **The long tail verified row by row** (`37d030f` = G's `2f56ae5` re-applied on `b214b0e` by G's own `apply`; cubit
  `tools/probes/re_sm120_rows.py --family {half,intalu,uniform,control,memory,atomic}`, nvdisasm 13.1 `-b SM120`: the 1 034 remaining
  harvest rows of 85 mnemonics on their canonical word and every single-bit flip of bits 0..104 / 122..125 — ~113 700 words named, 512
  census witnesses): **571 rows measured** (`rows-half-2026-11-07` 43, `rows-intalu-2026-11-07` 164, `rows-uniform-2026-11-07` 95,
  `rows-control-2026-11-07` 13, `rows-memory-2026-11-07` 199, `rows-atomic-2026-11-07` 57 — every declared field bit moves exactly its
  own operand token, no undeclared bit moves an operand of the same form; `variable_mask` is now exactly the field bits, text-inert
  bits fail closed; 237 fields added or retyped: undeclared guard / sign / abs / invert / `.reuse` bits, f16 / bf16 immediates the
  harvest spelled as integers, 4-bit predicate windows split into predicate + negation), **148 shadowed twin keys retired as
  `_meta.key_aliases`** (dotted twins, integer spellings of float-immediate forms, `?` / phantom-token junk, byte-identical rows; 0
  census witnesses except the 272 `S2R_R_II` words, which the target `S2R_R_L` prints identically), **11 unreachable rows deleted**
  (dead rows of staying keys — the six `F2FP_R_R_R_R` junk groups, three IADD3 spellings, one IMAD, one WARPSYNC; 0 census witnesses).
  Laws recorded in the fixtures per row; print orders found: `FHADD.BF16.RM.SAT`, `I2FP.F32.U32.RZ`, `LDGMC.E.ADD.F32.RN.STRONG.SYS`.
  Forms 1 231 → 1 083, variants 39 125 → 38 966, rows without `_src` 1 034 → 304. **The 304 rows left:** 192 verify field by field
  (their records are in the fixtures) but stay harvest until cubit's 2026-08/09 bug-wave suites, which pin the tolerant decode of their
  text-inert bits, are re-pinned to the measured doctrine (session T's next batch); the other 112 are named exceptions (the
  `EXCEPTIONS` of cubit's `tests/sm120_rows_<family>.rs`): 82 keys whose name does not spell the vendor text of their own word, 17
  over-wide harvest fields, 15 HADD2 / HFMA2 immediate-pair rows (bit 85 re-types both immediates), 12 undeclared operand bits, 8 keys
  whose every canonical word nvdisasm rejects, the PT-elided `IMAD.HI` / `R2UR.FILL` spellings, `WARPSYNC.COLLECTIVE`'s 50-bit
  immediate — key-naming and convention decisions, listed for the owner. Reviewed by T-17: 101 393 fixture words re-measured
  identical; the census's 249 929 words unchanged in text and re-encoding.
- **The IMAD HI groups keep bit 72 text-inert** (`37d030f`): the row probe's manual `-Ra` law at bit 72 (BUG-296, kept by cubit on the
  non-HI IMAD forms although nvdisasm prints no sign for it — session G's NEEDS 13 for the owner's silicon confirmation) was applied to
  all 34 measured IMAD rows; on the 12 HI rows the bit is not a negation (cubit's `native_hi` law, `tests/sm120_oracle_semantics.rs`), so
  the field is withdrawn and the bit fails closed; the rows stay measured.
- The SM120 reference page regenerated at `37d030f`; the SM103A / SM100A / SM121A pages regenerated at `b214b0e` (their tables'
  legend re-count of the previous push; the pages' content is unchanged but for the header's revision and table SHA-256).

## November 7, 2026 (session T-16 / session G run 5): the tensor-core long tail (QMMA / HMMA / OMMA / DMMA / IMMA) verified row by row on `sm120.json`; 15 stale legends re-counted

Session G measures, session T re-measures every fixture word with nvdisasm 13.1, checks the whole production census before and after,
gates and pushes. **`sm120.json`: 39 125 rows, 38 091 with `_src` — 1 034 harvest rows left** (1 376 of 39 203 at `3a94df3`, 18 841 of
26 598 at `c896e8f`); forms 1 309 → 1 231, `_meta.key_aliases` 14 613 → 14 691.

- **The tensor-core long tail verified row by row** (`1886972` = G's `b1b332b` re-applied on `3a94df3` by G's own `apply`; cubit
  `tools/probes/re_sm120_rows.py --family tensor`, nvdisasm 13.1 `-b SM120`: per harvest row the canonical word and all 109 single-bit
  flips of bits 0..104 / 122..125 — 37 620 words named, 14 554 form words): **264 rows measured** (`rows-tensor-2026-11-07`: QMMA 209,
  HMMA 31, OMMA 16, DMMA 4, IMMA 4 — every declared field bit moves exactly its own operand token, no undeclared bit moves an operand of
  the same form; `variable_mask` is now exactly the field bits, text-inert bits fail closed; 149 fields the vendor names were added —
  `.reuse` on the B operand at 123, the guard, `-Ra` 72 / `-Rb` 63), **78 shadowed twin keys retired as `_meta.key_aliases`** (73
  byte-identical `_P_` twins, 4 `_?` junk spellings, the dotted `HMMA.1684.F32.TF32_R_R_R_R`; 0 deletions, 0 census witnesses). Laws:
  QMMA (0x27a) shape 75 / accumulator 77 / A type [79:78] + 83 / B type 79 + 85 / `.SP` 80 / the `UP` gate [89:87] (inverted) + `!` 90 /
  `.reuse` on B only (123); HMMA (0x23c) shape 75 + 78 (`INVALID3`) / accumulator 76 / BF16 82 / TF32 83 / `.SP` 73; DMMA (0x23f)
  rounding [79:78], `|Ra|` 73 / `-Ra` 72 / `|Rb|` 62 / `-Rb` 63 / `|Rc|` 74 / `-Rc` 75; IMMA `ROW` / `COL` glyph planes 73 / 74. The 25
  `INVALIDn`-keyed rows verified as faithful glyph keys (keep vs fail closed: the owner's). Forms 1 309 → 1 231, variants 39 203 →
  39 125, rows without `_src` 1 376 → 1 034. Reviewed by T-16: 28 288 fixture words re-measured identical; the census's 6 249 words on
  the seven rewritten census-bearing keys unchanged in key, text and re-encoding.
- **15 stale provenance legends re-counted to their live rows** (`3dde6e1`): `atoms-aruri-sm120-2026-10-29` 1 → 0, `bug165` 5 → 3,
  `bug172` 3 → 2, `bug203` 56 → 55, `bug221` 14 → 13, `bug226b` 5 → 0, `bug226d` 36 → 28, `bug226e` 17 → 4, `bug229` 1 → 0, `bug230` 1 → 0,
  `bug339` 9 → 0, `dsv41-sm120-2026-10-22` 529 → 516, `f2-181` 7 → 6, `sm120-native-capture-control-2026-10-22` 22 → 21,
  `ctrl-silicon-sm120-2026-11-04` 13 → 12 (identical in every table since; none of the T1 batches introduced it). No row changes.
- The SM120 reference page regenerated at `3dde6e1`.
- **`b214b0e` — the same legend hygiene on the other tables:** `sm103a.json` (`b3001-lea-rebuild-2026-11-05` 1 → 0, `b3004-lea-x-2026-11-05` 2 → 1,
  `rows_total` 22 439 → 22 445 — the six MUFU packed-half rows of `8aa5f34` never updated it), `sm100a.json` (the two LEA legends), `sm121a.json`
  (`bug226d-2026-08-28` 1 → 0). No row changes; cubit re-vendored (`124299b`).

## November 7, 2026 (session T-16 / session G run 4): the 256-bit global load's U32 uniform-indexed form, the 256-bit global store and the global reduction's uniform-indexed forms measured whole on `sm120.json`

Session G measures, session T re-measures every fixture word with nvdisasm 13.1 on `-b SM120` / `SM103` / `SM100`, checks the
whole production census (249 929 distinct words) against the vendor's text before and after each batch, gates and pushes.
**`sm120.json`: 39 203 rows, 37 827 with `_src` — 1 376 harvest rows left** (11 789 of 34 079 at `5cb10e5`, 18 841 of 26 598 at
`c896e8f`); forms 11 725 → 1 309, `_meta.key_aliases` 4 185 → 14 613 (every retired dotted key names a group of its plain target).

- **LDG.256 ARURI measured whole** (`10b1a6a` = G's `020ab68`; cubit `tools/probes/re_sm120_ldg256.py --form ARURI`, 4 952 form + 17
  lattice words — the same text on the three targets for every form word, the L2 = 3 lattice plane `INVALID3` on SM103 / SM100):
  the 256-bit global load (opcode 0x97e, frame bit 91) in its `[Ra.U32+URb(+off)]` form (bits 75 / 76 clear) — the four new plain
  keys `LDG_R_R_ARURI` / `_II` / `_P` / `_II_P` with 1 152 measured groups each, provenance `ldg256-family-2026-11-07` (the legend
  9 216 → 13 824: the whole opcode on its twelve plain keys); the 3 120 dotted one-group harvest keys of the form are
  `_meta.key_aliases` (11 rows carried `wave11-integration-2026-11-05` — reproduced, 36 → 25; 5 carried a `claim_forbid`, 10 833 →
  10 828 — RZ is a legal elided base). Law = batches 10–11's with 75 / 76 clear. Forms 11 725 → 8 609, variants 34 079 → 35 567,
  rows without `_src` 11 789 → 8 680.
- **STG.256 measured whole** (`3216477` = G's `722e69e`; cubit `tools/probes/re_sm120_stg256.py`, 2 140 form + 93 lattice words — the
  same text on the three targets for every word): the 256-bit global store (opcode 0x97f, frame bit 91) in its three address
  forms — the six new plain keys `STG_dARI_R_R`, `STG_ARURI_R_R`, `STG_ARURI64_R_R` and their `_II` twins with 288 measured groups
  each (6 cache × 3 L2 × the store's 16-value enum), provenance `stg256-family-2026-11-07` (1 728 rows); the 1 164 dotted one-group
  harvest keys of the opcode are `_meta.key_aliases` (4 wave11 rows reproduced, 25 → 21). Law: Rs1 [23:16], Ra [31:24] (RZ illegal
  on the descriptor form → `claim_forbid`; elided on the uniform-indexed forms), Rs0 [39:32], a signed 19-bit 32-byte-unit offset
  [58:40], the trailing byte split [63:59] + [90:88] (elided at 0xff), URb [71:64], 75 / 76 the address form, the store's enum
  [80:77] (0 none, 1 CONSTANT.PRIVATE, 2 CONSTANT.CTA, 3 CONSTANT.CTA.PRIVATE, 4 STRONG.SM.PRIVATE, 5 STRONG.SM, 6
  STRONG.GPU.PRIVATE, 7 STRONG.GPU, 8 MMIO.GPU, 9 CONSTANT.SM, 10 STRONG.SYS, 11 CONSTANT.SM.PRIVATE, 12 MMIO.SYS, 13 SYS.ORDERED,
  14 STRONG.SYS.ORDERED, 15 MMIO.SYS.ORDERED), L2 [82:81], cache [86:84]; no LTC width, no trailing predicate — [74:72], 83, 87,
  [95:92] text-inert, fail closed. Forms 8 609 → 7 451, variants 35 567 → 36 131, rows without `_src` 8 680 → 7 520, claim_forbid
  rows 10 828 → 11 013 (the 576 measured descriptor rows carry the RZ exclusion).
- **REDG uniform-indexed forms measured whole** (`3a94df3` = G's `13ff289`; cubit `tools/probes/re_sm120_redg.py`, 9 334 form + 58
  lattice words — the same text on the three targets for every word): the global reduction (opcode 0x98e) in its `[Ra.U32+URb(+off)]`
  (`REDG_ARURI_R`, 6 144 groups = 2 `.E` × 8 ops × 4 types × 6 cache × 16 scope — the no-`.E` plane of the U32 form is legal, 3 072
  groups the harvest never saw) and `[Ra.64+URb(+off)]` forms (`REDG_ARURI64_R`, 3 072 groups — the `.64` base needs `.E`; without
  it the vendor prints `REDG.???0.<op>`), provenance `redg-aruri-family-2026-11-07` (9 216 rows); the 6 144 dotted one-group
  harvest keys are `_meta.key_aliases` (0 census witnesses). Law: [23:16] text-inert (the atomic's destination slot), Ra [31:24]
  (RZ: `[RZ.U32+URb]` on U32, the `???255` glyph on `.64`), Rb [39:32], a signed 24-bit byte offset [63:40], URb [71:64], `.E` 72,
  the type [75:73] (none / S32 / 64 / S64), the load's enum [80:77] with STRONG.SM.PRIVATE at 4, cache [86:84], the op [89:87]
  (ADD MIN MAX INC DEC AND OR XOR), 90 = `.64`, 91 = the uniform-indexed form, 81 = the descriptor form (another key); 76, 82, 83,
  [95:92] inert, fail closed. The descriptor form `REDG_dARI_R` (census-bearing) and `REDG_ARI_R` are untouched. Forms 7 451 →
  1 309, variants 36 131 → 39 203, rows without `_src` 7 520 → 1 376.
- Reviewed by T-16: the 16 594 fixture words re-measured identical; the whole production census (249 929 distinct words) decodes
  to the same key, text and re-encoding before and after each batch, the memory family (19 387 words on 18 keys) printing the
  vendor's text. The SM120 reference page regenerated at `3a94df3`.

## November 7, 2026 (session T-15 / session G run 3 → 4): the 256-bit global load's 64-bit uniform-indexed address form measured whole on `sm120.json`

Session G measures, session T re-measures every fixture word with nvdisasm 13.1 on `-b SM120` / `SM103` / `SM100`, checks the
whole census witness set of every touched key against the vendor's text, gates and pushes. **`sm120.json`: 34 079 rows,
22 290 with `_src` — 11 789 harvest rows left** (14 865 of 32 555 at `8aa5f34`, 18 841 of 26 598 at `c896e8f`).

- **LDG.256 ARURI64 measured whole** (`5cb10e5` = G's `af3329f`; cubit `tools/probes/re_sm120_ldg256.py --form ARURI64`, 4 952 form
  + 17 lattice words — the same text on the three targets for every form word (the L2 = 3 lattice plane is `INVALID3` on
  SM103 / SM100, `RML2.???` on SM120, as in batch 10); provenance `ldg256-family-2026-11-07`, the legend 4 608 → 9 216; cubit
  `tests/sm120_ldg256_aruri64.rs`): the 256-bit global load (opcode 0x97e, frame bit 91) in its 64-bit uniform-indexed address
  form `[Ra.64+URb(+off)]` (bit 75 = `.64`, bit 76 clear). The four new plain keys `LDG_R_R_ARURI64` / `_II` / `_P` / `_II_P` carry
  1 152 measured groups each — 6 cache × 3 L2 × 4 LTC × the 16-value strength / scope / PRIVATE enum; **the 3 084 dotted
  one-group harvest keys of the form are `_meta.key_aliases`** of the plain key of their operand signature, every one naming a
  group of its target (8 of the retired rows carried `wave11-integration-2026-11-05` — reproduced, that legend 44 → 36; 0
  census witnesses; forms 14 805 → 11 725, variants 32 555 → 34 079, aliases 1 101 → 4 185, claim_forbid rows 10 837 → 10 833 —
  the four retired dotted rows that carried one; the measured `.64` rows have none: every base is legal, RZ elides). Law =
  batch 10's with bit 76 clear: guard [15:12], Rd1 [23:16], Ra [31:24] (**RZ legal, elided — `[URb(+off)]`, the U32 twin's
  text; cubit mints the `.64` word for the bare spelling, BUG-464**), URb [39:32] (255 = `URZ`, printed), the signed 17-bit
  32-byte-unit offset [56:40], the trailing byte [63:57] + 72 elided at 0xff (carried by the `_II` keys), Rd0 [71:64], LTC
  [74:73], the strength / scope / PRIVATE enum [80:77], L2 [82:81], cache [86:84], the trailing predicate [90:87] (`pred_inv4`,
  the `_P` keys); bit 83 and [95:92] text-inert, fail-closed; bit 91 clear illegal; 75 = 0 with 76 = 1 illegal. Reviewed by
  T-15: the 4 969 words re-measured identical; the whole production LDG / STG / LD / `ARURI` census witness set (13 keys, 18 346
  words) prints the vendor's text with the new table.
- The SM120 reference page regenerated at `5cb10e5`.

## November 7, 2026 (session T-14 / session G run 3, pushed by T-15): the 256-bit global load's descriptor address form measured whole on `sm120.json`; the sm103a MUFU packed-half lanes measured on `sm103a.json`; NEEDS 46 (the 64-bit ISETP compare on sm103a / sm100a) measured

Session G measures, session T re-measures every fixture word with nvdisasm 13.1 on `-b SM120` / `SM103` / `SM100`, checks
the whole census witness set of every touched key against the vendor's text, gates and pushes. **`sm120.json`: 32 555 rows,
17 690 with `_src` — 14 865 harvest rows left** (15 647 of 28 729 at `94ff281`, 18 841 of 26 598 at `c896e8f`).

- **LDG.256 dARI measured whole** (`348535f`; cubit `tools/probes/re_sm120_ldg256.py --form dari`, 4 888 form + 81 lattice
  words — the same text on the three targets for every form word (the L2 = 3 plane is `INVALID3` on SM103 / SM100,
  `RML2.???` on SM120: lattice either way); provenance `ldg256-family-2026-11-07`; cubit `tests/sm120_ldg256_dari.rs`): the
  256-bit global load (opcode 0x97e, frame bit 91) in its descriptor address form `desc[URb][Ra.64(+off)]`. The four plain
  keys `LDG_R_R_dARI` / `_II` / `_P` / `_II_P` carry 1 152 measured groups each — 6 cache (EF / none / EL / LU / EU / NA,
  [86:84]) × 3 L2 (EFL2 / ENL2 / ELL2, [82:81]) × 4 LTC ([74:73]) × the 16-value strength / scope / PRIVATE enum [80:77]
  (4 607 groups added); **the 781 dotted one-group harvest keys of the form are `_meta.key_aliases`** of the plain key of
  their operand signature, every one naming a group of its target (0 census witnesses; forms 15 583 → 14 805, variants
  28 729 → 32 555, aliases 320 → 1 101). Law: guard [15:12], Rd1 [23:16], Ra [31:24] (RZ illegal → `claim_forbid`), URb
  [39:32], the signed 17-bit 32-byte-unit offset [56:40] (`sub_imm2_shr5`), the trailing byte [63:57] + 72 elided at 0xff
  (baked by the plain keys, carried by `_II` as `imm@57:7` + `imm_shr7@72:1`), Rd0 [71:64], bits 75 / 76 = base width +
  descriptor form, the trailing predicate [90:87] (`pred_inv4`, PT elided: baked by the plain keys, carried by `_P`); bit 83
  and [95:92] text-inert, fail-closed. Reviewed by T-14: the 4 969 words re-measured identical; the whole production LDG /
  STG / LD census witness set (8 020 words) prints the vendor's text with the new table.
- **The sm103a MUFU packed-half lanes measured** (`8aa5f34`, `sm103a.json`; cubit `tools/probes/re_sm103_mufu_x2.py`, run on
  the B300 host with nvdisasm **13.3.73** on `-b SM103` — 13.0.85 and 13.1 text-identical on every word; provenance
  `mufu-x2-sm103a-2026-11-07`, 6 rows; cubit `tests/sm103a_mufu_x2.rs`): `MUFU.EX2.BF16x2` / `MUFU.EX2.F16x2` in the register
  (0x308), uniform (0xd08 + bit 91) and immediate (0x908) forms — `.BF16x2` = bit 80 with the half code 0, `.F16x2` = the half
  code 3; EX2 ([77:74] = 2) is the only function the packed lanes admit (the others print `INVALIDn`); Rd [23:16], Ra / URa
  [39:32] with `|x|` 62 / `-x` 63; the immediate form carries two halves (hi [63:48] printed first, lo [47:32]) as bf16 / f16
  literals; bit 60 text-inert in the lanes; bit 80 with a half code and half 3 with bit 80 are illegal. On `-b SM120` /
  `SM100` bit 80 is text-inert and half 3 illegal — the lanes exist on sm103a only (found by G's conversion batch, where 6
  of the 3 059 SM120 sweep words differed on `-b SM103`). New groups `BF16x2,EX2` / `EX2,F16x2` on `MUFU_R_R` and `MUFU_R_UR`,
  the new key `MUFU_R_FI_FI` (forms 14 925 → 14 926, variants 22 439 → 22 445). The scalar sm103a MUFU rows stay harvest rows
  (the family proper, later). Silicon not measured (the B300 GPU was busy); no compiler word of the B300 corpora uses the lanes.
- **NEEDS 46 measured — the 64-bit ISETP compare on sm103a / sm100a** (T-14 on the B300, nvdisasm 13.3.73 + 13.0.85): bit 80
  is text-inert on `-b SM103` / `SM100` in every 13.x build (the `.S64` / `.U64` suffix is SM120-only; 0 of 2 136 sweep words
  carry it there). The four `S64` / `U64` harvest groups of `ISETP_P_P_R_UR_P` on `sm103a.json` / `sm100a.json` (AND,GE,U64 /
  AND,GT,U64 / AND,NE,S64 / NE,OR,S64) are text phantoms — their canonical words print `.U32` compares there (`.U64` on
  SM120) or are illegal on every target — and no ISETP word of 2 145 corpus cubins (2.08 M ISETP words over sm_100 / sm_103 /
  sm_120) sets bit 80. Not changed: whether the silicon compares 64 bits needs a probe on an idle B300 (listed in the README).
- The SM120 and SM103A reference pages regenerated at `8aa5f34`.

## November 7, 2026 (session T-13 / session G run 2): the T1 production line continues — UFSETP, the FP32 ALU, the uniform FP32 ALU, the conversions and FSET / FSEL / FCHK measured whole on `sm120.json`; the QMMA.SF opcode-bit-2 variant identified as MXINT8 × MXINT8

Session G measures, session T re-measures every fixture word with nvdisasm 13.1 on `-b SM120` / `SM103` / `SM100`,
gates and pushes. **`sm120.json`: 28 729 rows, 13 082 with `_src` — 15 647 harvest rows left** (16 905 of 26 442 at
`e1e8598`, 18 841 of 26 598 at `c896e8f`).

- **UFSETP measured whole** (`63e5040`; cubit `tools/probes/re_sm120_ufsetp.py`, 394 form + 6 glyph + 66 lattice words;
  provenance `ufsetp-family-2026-11-06`, 192 rows; cubit `tests/sm120_ufsetp.rs`): the two keys
  `UFSETP_UP_UP_UR_{UR,FI}_UP` (0x253 + 91 / 0x853 + 91; bit 91 clear is illegal) — the 2026-10-22 "mode grid" rows
  (16 comparisons × 3 booleans × FTZ = 96 per key) confirmed field by field (masks and bases unchanged) and re-emitted
  with provenance. **The family is SM120-only: on `-b SM103` / `SM100` nvdisasm 13.1 rejects every one of the 466 sweep
  words** (the sm103a / sm100a tables carry no UFSETP key — consistent). Six keys retired as `_meta.key_aliases`
  (0 census witnesses): `UFSETP_UP_UP_UR_II_UP` → `UFSETP_UP_UP_UR_FI_UP` (the encode-only integer-spelling twin; the
  hexadecimal f32 literal still encodes through the alias), `UFSETP.GT.AND_UP_UP_UR_UR_UP`,
  `UFSETP.NEU.{AND,OR}_UP_UP_UR_UR_UP` → `UFSETP_UP_UP_UR_UR_UP`, `UFSETP.NEU.AND_UP_UP_UR_{FI,II}_UP` →
  `UFSETP_UP_UP_UR_FI_UP` (dotted encode-only harvest samples with a baked guard and predicates). Law = FSETP in the
  uniform datapath: guard [15:12] (`@[!]UPn`); [23:16] text-inert; URa [31:24], `-URa` 72, `|URa|` 73; URb [39:32] with
  `|URb|` 62, `-URb` 63 ([61:40] and [71:64] text-inert there) or the f32 immediate at [63:32]; the boolean [75:74]
  (AND / OR / XOR; 3 = `INVALID3`); the comparison [79:76] (F LT EQ LE GT NE GE NUM NAN LTU EQU LEU GTU NEU GEU T);
  `.FTZ` 80; UPd0 [83:81]; UPd1 [86:84]; UPc [89:87] + `!` 90; [95:92] text-inert; no `.reuse` (nvdisasm rejects the
  reuse bits on the uniform opclass). Reviewed by T-13: the 466 words re-measured identical under the probe's control
  code, 464 of them illegal on the two other targets (the 2 vendor-rejected lattice words are rejected on all three).
- **FP32 ALU measured whole — FADD / FMUL / FFMA / FMNMX** (`dad8393`; cubit `tools/probes/re_sm120_fp32_alu.py`, 1 636
  form + 30 glyph + 492 lattice + 16 neighbour words, the same text on the three targets; provenance
  `fp32-alu-family-2026-11-07`, 696 rows; cubit `tests/sm120_fp32_alu.rs`): the 14 plain keys `FADD_R_R_{R,FI,UR}` (0x221 /
  0x421 / 0xe21 + 91), `FMUL_R_R_{R,FI,UR}` (0x220 / 0x820 / 0xc20 + 91), `FFMA_R_R_R_R` / `_R_R_R_FI` / `_R_R_FI_R` /
  `_R_R_UR_R` / `_R_R_R_UR` (0x223 / 0x423 / 0x823 / 0xc23 + 91 / 0xe23 + 91), `FMNMX_R_R_{R,FI,UR}_P` (0x209 / 0x809 /
  0xc09 + 91) — 355 harvest rows rewritten in place (`FADD_R_R_R` baked a sample's guard, claimed the unused reuse bit
  123 and lacked the `|Rb|` 62 field) and **341 missing groups added** (`FMUL_R_R_FI` / `_UR` had 3 of 168 groups,
  `FADD_R_R_R` 5 of 16 — legal words nvdisasm names); 27 junk keys retired as `_meta.key_aliases` → the plain key of
  their opcode (12 FADD, 5 FMUL, 5 FFMA, 5 FMNMX: the `_?` / `_L` / `_R_R_R_R` mis-tokenised harvest keys, dotted twins
  with the modifier in the key — `FADD.RZ_R_R_R`'s row decoded the `.RM` word as `.RZ` —, and the integer-spelling
  `*_II*` twins of the f32 forms, whose hexadecimal literals still assemble through the alias; the 185 production census
  words of `FMUL_R_R_II` decode on `FMUL_R_R_FI` and print the vendor's text). Law: guard [15:12]; Rd [23:16]; Ra [31:24]
  (`-` 72, `|` 73, `.reuse` 122); B [39:32] (Rb / URb, `|` 62, `-` 63) or f32 [63:32]; the C slot [71:64] (FFMA: Rc,
  `|` 74, `-` 75; text-inert on FADD / FMUL / FMNMX). FADD `[.FTZ][.RM|.RP|.RZ][.SAT]` (FTZ 80, rounding [79:78], SAT 77);
  FMUL `[.FTZ|.FMZ][.D8|.D4|.D2|.M2|.M4|.M8][.RND][.SAT]` (FMZ 76, both = `INVALID3`; scale [86:84] with 4 = unspelled, 0 =
  `INVALID0`); FFMA the same without the scale; FMNMX `[.FTZ][.NAN][.XORSIGN]` (80 / 81 / 82) + Pc [89:87] / `!` 90.
  `.reuse`: FADD Ra 122 / Rb 124, FMUL 122 / 123, FFMA 122 / 123 / 124 by slot, FMNMX 122 / 123; none on UR / f32 operands.
  Measured and left unclaimed (candidates for later batches): bit 81 on the register / UR forms of FADD / FFMA = `FHADD` /
  `FHFMA` (SM120 mixed precision; text-inert on the immediate forms), bit 65 on FMNMX = `FMNMX.IS_A Rd, Pd, Ra, Rb, Pc`
  (Pd [68:66]; no key in any table). Reviewed by T-13: the 2 174 words re-measured identical on the three targets; 733
  sampled census words of the 14 keys print nvdisasm's text on the new rows.
- **NEEDS 44 closed on silicon (`dac48ad`): the QMMA.SF opcode-bit-2 variant (0x47e) is the block-scaled MMA over MXINT8 ×
  MXINT8** — `QMMA.SF.16832.F32.MXINT8.MXINT8.E8_R_R_R_R_R_R_UR` (was `…UNIDENTIFIED.E8` since 2026-11-06; the hypothesis key
  `IMMA.SF.16832.U8.U8_R_R_R_R_R_R_UR` and the interim key are `_meta.key_aliases`; provenance
  `qmma-sf-mxint8-silicon-2026-11-07`, `nvdisasm_sm120: "no text"`). Measured by T-13 on the test host GPU 0 (RTX PRO 6000
  Blackwell SE) with a cubit-assembled kernel that runs the five documented diagonal type pairs and the 0x47e word on
  identical fragments: byte-value sweeps of A and of B against three references give 32 · (int8(a)/64) · (int8(b)/64)
  exactly for all 6 × 256 cases — OCP MX INT8 elements (two's-complement int8 with six fractional bits, range
  [−2, 1.984375], under the E8M0 block scale), fp32 output; one-hot bytes light one D row (A) / one D column (B) exactly
  as the `E4M3.E4M3` row does (the m16n8k32 8-bit fragment layout); a byte is one element (0x04 → 0.0625, 0x44 → 1.0625);
  D · 2^(s−127) for every E8M0 scale with NaN at 0xff. Fields unchanged (the family's). Side result of the same run: the
  documented `E2M1` type code computes exactly the E2M3 map on bits [5:0] of the 8-bit container for every byte value
  (PTX places `.e2m1` in bits [5:2] with zero padding; the silicon does not mask bits [1:0]) — the E2M1 / E2M3 rows are
  correctly named and compute the same arithmetic on well-formed inputs. Not measured: the type-field lattice under
  opcode bit 2. cubit `tests/fixtures/sm120_qmma_sf_mxint8_silicon.jsonl`, `tests/needs44_qmma_sf_bit2.rs`.
- **Uniform FP32 ALU measured whole — UFADD / UFMUL / UFFMA / UFSEL** (`51d2785` = G's `784183d` re-applied on `dac48ad`
  by T, byte-identical to G's table plus the NEEDS 44 rename; cubit `tools/probes/re_sm120_ufp32_alu.py`, 1 025 form + 30
  glyph + 278 lattice + 4 neighbour words; provenance `ufp32-alu-family-2026-11-07`, 441 rows + 3 dsv41 rows kept; cubit
  `tests/sm120_ufp32_alu.rs`): the 9 plain keys, all with bit 91 — `UFADD_UR_UR_{UR,FI}` (0x254 / 0x454; **URb lives in the
  C slot [71:64]**), `UFMUL_UR_UR_{UR,FI}` (0x256 / 0x856), `UFFMA_UR_UR_UR_UR` / `_UR_FI` / `_FI_UR` (0x255 / 0x455 /
  0x855), `UFSEL_UR_UR_{UR,FI}_UP` (0x251 / 0x851) — 143 harvest rows rewritten (T-10's `UFMUL_UR_UR_UR` row had no sign
  fields; the `audit-fix-2026-11-06-t10` legend follows its live count 25 → 24) and **298 missing groups added**
  (`UFMUL_UR_UR_UR` had 1 of 168, `UFADD_UR_UR_UR` 1 of 16, `UFFMA_UR_UR_UR_UR` 1 of 24, `UFMUL_UR_UR_FI` 112 of 168); 9
  junk keys retired as `_meta.key_aliases` → the f32 key of their opcode (`UFADD_UR_UR_II`, `UFMUL_UR_UR_II`,
  `UFFMA_UR_UR_UR_II`, `UFFMA_UR_UR_UR_L`, `UFSEL_UR_UR_?_?`, `UFSEL_UR_UR_?_UP`, `UFSEL_UR_UR_II_?`, `UFSEL_UR_UR_II_UP`,
  `UFSEL_UR_UR_L_UP`; the hexadecimal f32 literals still assemble through them). **The family is SM120-only: every one of
  the 1 337 sweep words is illegal on `-b SM103` / `SM100`** (like UFSETP; the sm103a / sm100a tables carry none of the four
  mnemonics). Law = the FP32 ALU law in the uniform datapath: guard [15:12] `@[!]UPn`; URd [23:16]; URa [31:24] (`-` 72,
  `|` 73); B [39:32] (`|` 62, `-` 63) or f32 [63:32]; the C slot [71:64] (`|` 74, `-` 75); UFADD `[.FTZ][.RM|.RP|.RZ][.SAT]`,
  UFMUL `[.FTZ|.FMZ][.D8…M8][.RND][.SAT]` (scale [86:84], 4 unspelled, 0 = `INVALID0`, FTZ + FMZ = `INVALID3`), UFFMA
  `[.FTZ|.FMZ][.RND][.SAT]`, UFSEL `[.FTZ]` + UPc [89:87] / `!` 90 (`UPT` spelled); no `.reuse` (nvdisasm "undefined opex");
  bit 81 on the register forms of UFADD / UFFMA = `UFHADD` / `UFHFMA` (neighbours, no key anywhere). Reviewed by T-13: the
  1 337 words re-measured identical on SM120 and illegal on the two other targets; the 112 `UFMUL_UR_UR_FI` rows and the 3
  kept rows byte-identical in bases / masks / fields.
- **Conversions measured whole — F2I / I2F / FRND / MUFU** (`9d054e4` = G's `bfbcf85` re-applied on `51d2785` by T, byte-identical
  to G's table plus the NEEDS 44 rename; cubit `tools/probes/re_sm120_conv.py`, 2 494 form + 18 glyph + 547 lattice words;
  `-b SM100` identical, **`-b SM103` differs on 6 MUFU words — the B300 MUFU has packed-half lanes (bit 80 → `.BF16x2`,
  half code 3 → `.F16x2`), text-inert / illegal on SM120 and SM100**; provenance `conv-family-2026-11-07`, 2 020 rows + 10
  `mufu-ur-sm120-2026-11-04` rows kept; cubit `tests/sm120_conv.rs`): the 12 plain keys `F2I_R_{R,UR,FI}` (0x305 / 0x311 with
  the UR frame 0xc00 + bit 91 and the immediate frame 0x800), `I2F_R_{R,UR,II}` (0x306 / 0x312), `FRND_R_{R,UR,FI}` (0x307 /
  0x313), `MUFU_R_{R,UR,FI}` (0x308) — `F2I_R_FI`, `I2F_R_II`, `FRND_R_UR`, `FRND_R_FI` are new keys (legal forms the harvest never
  saw); 130 harvest rows rewritten in place, **1 890 missing groups added** (the F2I lattice is 512 groups per form; the 16
  `.F16` / `.BF16` MUFU UR rows gain the `.H1` selector); the measured-but-superseded rows of the 2026-09 bug waves retired
  (the legends `bug380-2026-09-05`, `bug390-2026-09-06`, `bug403-2026-09-06` follow their live count 0; `audit-fix-2026-11-06-t10`
  24 → 23); **213 junk keys retired as `_meta.key_aliases` — the 207 dotted F2I keys** (206 `_R_R` and
  `F2I.FTZ.NTZ.TRUNC.U32_R_UR`; `F2I.FTZ.U32.TRUNC.NTZ_R_R` carried 190 production census words, which decode on `F2I_R_R` and
  print the vendor's text), `I2F.RP_R_R`, `FRND.CEIL_R_R` / `.F16` / `.FLOOR` / `.TRUNC`, `MUFU_R_II` → `MUFU_R_FI`. Law: guard
  [15:12]; Rd [23:16]; the source [39:32] (Ra / URa with `|` 62 / `-` 63 on F2I / FRND / MUFU, text-inert on I2F) or the
  immediate at [63:32] (f32; the f16 / bf16 half at [47:32]; the f64 high dword; the integer by the group's type); no `.reuse`
  (text-inert). F2I: dst width [76:75] (8 / 16 / 32 / 64) + signed 72 (S32 unspelled), src [86:84] (1 F16, 2 F32 unspelled, 3
  F64, 4 BF16), NTZ 77, rounding [79:78], FTZ 80 → `F2I[.FTZ][.dst][.src][.rnd][.NTZ]`; the 64-bit dst and the F64 src on 0x311.
  I2F: dst [77:75] (1 F16, 2 F32 unspelled, 3 F64 on 0x312, 4 BF16), src width [85:84] + signed 74, rounding; `.B1..B3` at
  [61:60] / `.H1` at 60 on the 8- / 16-bit sources; the immediate form for 32- / 64-bit sources only, signed hex for S-types.
  FRND: type [77:75] = [86:84] (F64 on 0x313), FTZ 80, rounding → `FRND[.FTZ][.type][.rnd]`. MUFU: fn [77:74] (COS SIN EX2 LG2 RCP
  RSQ RCP64H RSQ64H SQRT TANH), half [73:72] (F16 / BF16). **The F16 / BF16 sources of F2I / FRND / MUFU take `.H1` at bit 60
  too, composed inside the sign pipes (`-|R2.H1|`)**; the `???n` / `INVALIDn` states fail closed. **F2F is excluded** — its
  type pair is order-significant (`F2F.F16.F32` ≠ `F2F.F32.F16`) and the sorted group-name scheme cannot carry it on a plain
  key (the typed-key convention of the IMMA / IDP families is the way; cubit `G/NEEDS_OWNER.md` 3). Reviewed by T-13: the
  3 059 words re-measured identical on SM120 (the 6 SM103 differences as recorded); 610 sampled census words of the touched
  keys print nvdisasm's text on the new rows.
- **FSET / FSEL / FCHK measured whole** (`94ff281` = G's `b94a63e` re-applied on `9d054e4` by T, byte-identical to G's table
  plus the NEEDS 44 rename; cubit `tools/probes/re_sm120_fset_fsel_fchk.py`, 885 form + 15 glyph + 375 lattice words, the
  same text on the three targets; provenance `fset-fsel-fchk-family-2026-11-07`, 297 rows; cubit `tests/sm120_fset_fsel_fchk.rs`):
  the 9 plain keys `FSET_R_R_{R,FI,UR}_P` (0x20a / 0x80a / 0xc0a + 91; the `_FI_P` / `_UR_P` keys are new), `FSEL_R_R_{R,FI,UR}_P`
  (0x208 / 0x808 / 0xc08 + 91), `FCHK_P_R_{R,FI,UR}` (0x302 / 0x902 / 0xd02 + 91) — 102 harvest rows rewritten in place (the
  2026-10-22 FSET mode grid, the FSEL / FCHK rows, three of the four `bug249-2026-08-28` FSEL rows with the same fields; that
  legend 4 → 1) and **195 missing groups added**; 8 junk keys retired as `_meta.key_aliases` (`FSET.BF.F.AND_R_R_R_P` →
  `FSET_R_R_R_P`; `FSEL_R_R_?_P_P`, `FSEL_R_R_II_P_P` → `FSEL_R_R_FI_P`; `FSEL_R_R_R_P_II_?` / `_II_II_?` / `_II_II_II_?`,
  `FSEL_R_R_R_P_P` → `FSEL_R_R_R_P`; `FCHK_P_R_II` → `FCHK_P_R_FI`); **`FSEL_R_R_II_P` kept** — the BUG-178 / BUG-249 canonical
  `+QNAN` lane (0x7fc00000 baked), measured by that wave, not harvest. The junk `FCHK_P_R_UR` row carried a `claim_forbid`
  on the opcode nibble; the measured row pins the whole opcode (claim_forbid rows 7 012 → 7 011). Law: guard [15:12]; Rd
  [23:16] (FSET / FSEL) or Pd [83:81] (FCHK); Ra [31:24] (`-` 72, `|` 73, `.reuse` 122); B [39:32] (Rb / URb, `|` 62, `-` 63;
  Rb `.reuse` 123) or f32 [63:32]. FSET = the FSETP law with a register destination: the boolean [75:74] (3 = `INVALID3`), the
  comparison [79:76], FTZ 80, Pc [89:87] + `!` 90, **`.BF` on every word** (no bit selects an integer-result form on SM120)
  → `FSET.BF.<cmp>[.FTZ].<bool>`. FSEL = `[.FTZ]` (80) + Pc (PT spelled). FCHK = Pd, no modifier, no `.reuse` (text-inert).
  Unused bits fail closed. Reviewed by T-13: the 1 275 words re-measured identical on the three targets; 220 sampled census
  words of the touched keys print nvdisasm's text on the new rows.
- The SM120 reference page regenerated at `94ff281`.

## November 6–7, 2026 (session T-12 / session G): the T1 production line — HSETP2 and ISETP / UISETP measured whole on `sm120.json`

The harvest rows of 2026-06..08 (rows without `_src`) are being replaced family by family: session G (rtx) and the B300
lane measure, session T re-measures every fixture word with nvdisasm 13.1 on `-b SM120` / `SM103` / `SM100`, gates and
pushes. **`sm120.json`: 26 662 rows, 9 101 with `_src` — 17 561 harvest rows left** (18 841 of 26 598 at `c896e8f`).

- **HSETP2 measured whole** (`dc1f485`; cubit `tools/probes/re_sm120_hsetp2.py`, 1 002 form + 88 lattice words, the
  same text on the three targets; provenance `hsetp2-family-2026-11-06`, 768 rows; cubit `tests/sm120_hsetp2.rs`).
  The 384 + 384 "mode grid" rows of `HSETP2_P_P_R_R_P` / `HSETP2_P_P_R_UR_P` (2026-10-22) were field-correct and are
  re-emitted from the measurement with provenance; three dotted harvest keys are `_meta.key_aliases` of the plain keys
  (`HSETP2.GEU.AND_P_P_R_R_P`, an encode-only twin; `HSETP2.GEU.AND_P_P_R_R_R` / `_P_P_R_UR_UR`, a `R2.H0_H0, R3.H0_H0`
  sample mis-tokenised as a third register; 0 census witnesses). Law: opcode 0x234 (register form; bit 91 illegal) /
  0xe34 + bit 91 (uniform); guard [15:12]; Ra [31:24], `-Ra` 72, `|Ra|` 73, selector [75:74] (0 none, 2 `.H0_H0`,
  3 `.H1_H1`, 1 = the vendor's `.INVALID1`); Rb / URb [39:32], selector [61:60], `|Rb|` 62, `-Rb` 63; the type [65:64]
  (0 fp16, 2 `.BF16_V2`, 1 / 3 `INVALID1` / `INVALID3`), the boolean [70:69] (AND / OR / XOR / `INVALID3`), `.H_AND` 71,
  the comparison [79:76] (F LT EQ LE GT NE GE NUM NAN LTU EQU LEU GTU NEU GEU T), `.FTZ` 80, Pd0 [83:81], Pd1 [86:84],
  Pc [89:87] + `!` 90; [23:16], [59:40], 66–68, [95:92] text-inert (fail-closed). `.reuse`: Ra 122, Rb 124.
- **ISETP / UISETP measured whole** (`6dff9c7`; cubit `tools/probes/re_sm120_isetp.py`, 1 944 form + 192 lattice
  words; provenance `isetp-family-2026-11-06`, 576 rows; cubit `tests/sm120_isetp.rs`): 960 groups on the ten keys
  `ISETP_P_P_R_{R,UR,II}_P` (0x20c / 0xc0c + 91 / 0x80c), `UISETP_UP_UP_UR_{UR,II}_UP` (0x28c + 91 / 0x88c + 91) and
  their `.EX` twins (`_P` / `_UP`) — 509 harvest rows rebuilt (they baked PT guards and a sample's predicates, their
  masks excluded their own fields), 67 missing immediate-form groups added, the 384 dsv41 rows of 2026-10-22 reproduced
  field for field and kept with their tag. Law: guard [15:12]; [23:16] inert; Ra / URa [31:24]; Rb / URb [39:32] or
  imm32 [63:32]; the `.EX` carry-in `[!]Pn` at [71:68] is read only with bit 72 (`.EX`) — without it the window is
  text-inert and nvcc writes PT (7 414 / 7 414 census words), so the non-EX rows bake `[70:68] = 7` and fail closed
  elsewhere; 73 signed (32-bit signed prints no type, clear = `.U32`); [75:74] the boolean (3 = `INVALID3`); [78:76] the
  comparison (F LT EQ LE GT NE GE T); 79 inert; **80 = the 64-bit compare (`.S64` / `.U64`) — SM120-only: on `-b SM103`
  / `SM100` nvdisasm 13.1 renders no 64-bit compare and the bit is text-inert** (977 of the 2 136 sweep words differ by
  that suffix alone; the sm103a / sm100a harvest `S64` / `U64` ISETP groups are therefore a question for the B300 lane);
  Pd0 [83:81], Pd1 [86:84], Pc [89:87] + `!` 90; 91 = the uniform frame. `.reuse`: Ra 122, Rb 123 (register form);
  UISETP none. The BUG-092 encode-only retention group of `UISETP_UP_UP_UR_UR_UP` stays.
- For the record (met in both reviews): nvdisasm prints `.reuse` only when the control word's yield bit (109) is set —
  a fixture's `.reuse` texts reproduce only under the probe's control code.
- **NEEDS 44 (`39bafbb`): the `IMMA.SF.16832.U8.U8_R_R_R_R_R_R_UR` hypothesis row (opcode 0x47e, 2026-03-26, no numerical
  oracle) is `QMMA.SF.16832.F32.UNIDENTIFIED.E8_R_R_R_R_R_R_UR`** — measured on sm_120 silicon by session R on 2026-11-06
  (test host GPU 0, RTX PRO 6000 SE; cubit-assembled probes next to the documented QMMA.SF E4M3.E4M3 / E2M1.E2M1 forms on
  identical fragments): the word executes, writes fp32, honours the E8M0 block scales (scale 0xff → NaN on every element,
  scale ×2 → result ×2) and matches neither documented input type (A = 0x38, B = 0x40, scales 1.0: 64.0 / 0.0 / 28.0) —
  an undocumented block-scaled MMA of the QMMA.SF family (0x47a) selected by opcode bit 2, input element types
  unidentified, not an integer MMA. The row carries the family's fields (Rd@16 Ra@24 Rb@32 Rc@64 sfA@40 sfB@52 UR@60+73;
  the hypothesis row read R6 at [55:48] and had no UR field), provenance `qmma-sf-opcode-bit2-silicon-2026-11-06`
  (undocumented form measured on silicon, `nvdisasm_sm120: "no text"`), the old key as a `_meta.key_aliases` entry; the
  IMMA / U8 claims are withdrawn (`_discovery`). What stays open: the input element types (~1 h of silicon time with R's
  harness). With it the SM120 dynamic audit's replay shows **0 rows with a finding** (cubit `T/audit/dyn23`).
- **FSETP / DSETP measured whole** (G's `8e6c8ae`, re-applied on the NEEDS 44 table as `5b4b03c`; cubit
  `tools/probes/re_sm120_fsetp_dsetp.py`, 1 022 form + 12 glyph + 223 lattice words, the same text on the three targets;
  provenance `fsetp-dsetp-family-2026-11-06`, 432 rows; cubit `tests/sm120_fsetp_dsetp.rs`): the six keys
  `FSETP_P_P_R_{R,UR,FI}_P` (0x20b / 0xc0b + 91 / 0x80b) and `DSETP_P_P_R_{R,UR,FI}_P` (0x22a / 0xe2a + 91 / 0x42a) —
  the 2026-10-22 "mode grid" rows confirmed field by field and re-emitted with provenance (DSETP's `reuse` fields
  dropped: text-inert there). 112 junk keys gone: 100 dotted encode-only twins `FSETP.<cmp>[.FTZ].<bool>_P_P_R_{R,UR,FI,II}_P`
  (byte-identical copies of the plain keys' groups; deleted — the SHF / HADD2 precedent), `FSETP_P_P_L_L_P` (five
  vendor-rejected rows; deleted), 11 `_meta.key_aliases` (`FSETP.{GE,NAN}.FTZ.AND_P_P_R_R_R`,
  `FSETP.{GEU,NEU}.AND_P_P_R_UR_UR`, `FSETP.NEU.OR_P_P_R_UR_UR`, `FSETP.NE.FTZ.AND_P_P_R_R_?`,
  `FSETP.NEU.{AND,OR}_P_P_R_R_P_?`, `FSETP_P_P_II_II_P`, `FSETP_P_P_II_R_P` → the plain key of their opcode;
  `FSETP_P_P_R_II_P` → `FSETP_P_P_R_FI_P`). Law: guard [15:12]; [23:16] inert; Ra [31:24], `-Ra` 72, `|Ra|` 73; Rb / URb
  [39:32], `|Rb|` 62, `-Rb` 63 (register forms) or the immediate at [63:32] (f32 / the f64 high half); the boolean
  [75:74] (AND / OR / XOR / `INVALID3`); the comparison [79:76] — FSETP F LT EQ LE GT NE GE NUM NAN LTU EQU LEU GTU
  NEU GEU T, **DSETP names 0 / 15 `MIN` / `MAX`**; 80 `.FTZ` on FSETP, text-inert on DSETP; Pd0 [83:81]; Pd1 [86:84];
  Pc [89:87] + `!` 90; 91 the uniform frame; [95:92] inert. `.reuse` Ra 122 / Rb 123 on FSETP, text-inert on DSETP.
- **PLOP3 / P2R / R2P measured whole** (G's `eb9fad1`, re-applied as `e1e8598`; cubit
  `tools/probes/re_sm120_plop3_p2r_r2p.py`, 202 form + 158 lattice words, the same text on the three targets;
  provenance `plop3-p2r-r2p-family-2026-11-06`, 8 rows; cubit `tests/sm120_plop3_p2r_r2p.rs`). PLOP3 (0x81c): guard
  [15:12]; imm8 [23:16]; [63:24] text-inert; the LUT = [66:64] + [76:72]; bit 67 = the third source predicate is a
  uniform predicate (the `_UP_` key — the harvest row had it right but baked `UPT`); Pc [70:68] + `!` 71; Pb [79:77] +
  `!` 80; Pd0 [83:81]; Pd1 [86:84]; Pa [89:87] + `!` 90. P2R (0x803): Rd [23:16]; Ra [31:24]; **the mask is imm32 at
  [63:32]** (`P2R R17, PR, RZ, 0x80000040`; the 2026-10-22 rows had swept 0..255 and carried an 8-bit field, so masks
  above 0xff did not decode); [77:76] the byte selector as the mnemonic suffix `.B1` / `.B2` / `.B3`; `.reuse` Ra 122.
  R2P (0x804): [23:16] text-inert; Ra [31:24] with the byte selector [77:76] as the operand suffix (`R20.reuse.B2`);
  imm32 [63:32], **elided by the vendor when it is 0xff** (`R2P PR, R0`) — the new encode-only `R2P_II_R` carries that
  bare spelling (mask baked), the explicit `, 0xff` stays a bitwise alias; the masks print unsigned. Bit 91 illegal on
  all three. 10 junk keys retired as aliases (`PLOP3.LUT_P_P_P_P_P_II_II` and its era alias, seven P2R shapes with `PR`
  read as a register / label and 6–7-bit masks, `R2P_?_R_II`, `R2P_L_R_II`).
- **`sm120.json` after the four batches and NEEDS 44: 26 442 rows, 9 537 with `_src` — 16 905 harvest rows left**
  (18 841 of 26 598 at `c896e8f`). Where the rest is: LDG 7 019 + REDG 6 162 + STG 1 211 rows — the dotted `dARI` /
  `ARURI` lattice keys with one group each (≈ 6 970 LDG keys, 6 146 REDG, 1 179 STG), the B300 lane's assignment.
- The SM120 reference page regenerated at `e1e8598`.

## November 6, 2026 (session T-11): the IDP family on the four tables (coldface's D2 keys reviewed and completed), the SM120 audit's named exceptions 14 → 1

- **`sm120.json` / `sm103a.json` / `sm100a.json` / `sm121a.json`: the `IDP` family measured whole** (cubit
  `tools/probes/re_sm120_idp.py`, nvdisasm 13.1: 128 form + 480 lattice words per table, the same text on `-b SM120` /
  `SM103` / `SM100` / `SM121` for every word; provenance `idp-family-2026-11-06`, test cubit `tests/sm120_idp.rs`).
  Law: guard [15:12], Rd [23:16], Ra [31:24], Rb [39:32] (opcode 0x7226) / URb [39:32] (opcode byte 0xc26 + bit 91 —
  the frame bit flipped either way is illegal), Rc [71:64]; the enum window [79:72]: [77:76] the mode (00 `4A`,
  01 `2A.LO`, 11 `2A.HI`, 10 illegal — the 64 vendor-rejected values), 73 = the first type signed (`S8` / `S16`),
  74 = the second type signed (`S8`), 75 `-Rc`; 72, 78, 79, [63:40], [95:80] text-inert (fail-closed on the measured
  rows). Twelve legal forms per operand shape: the mixed-type forms carry typed keys (`IDP.4A.S8.U8_R_R_R_R`, …,
  `IDP.2A.HI.U16.S8_R_R_UR_R` — NEEDS 36 = the owner's D2 decision of 2026-11-06; the B300 lane's
  `IDP.4A.{S8.U8,U8.S8}_R_R_R_R` rows, `42192f0`, are the first of them and were taken as they are after a word-by-word
  review), the symmetric forms stay mod-group rows of `IDP_R_R_R_R` / `IDP_R_R_UR_R` (sm121a keeps typed keys for every
  register form, its schema). **24 rows per table.** What the measurement closed: the uniform-register rows (the 2026-08
  harvest donor `IDP_R_R_UR_R [4A,S8,S8]` and D2's three typed copies of it) had no `-Rc` field, failed closed on the
  inert bit 72 and baked `Rc = 0xff` / UR bit 34 under their fields — `IDP.4A.U8.U8 R15, R15, UR6, -R7` decoded as the
  `S8.S8` row through cubit's sign-tolerant fallback; the 2A half of the family had no rows (the mixed `LO` types and
  every `HI` form: 6 register + 8 uniform forms) and a `2A.LO.U16.U8` word with bit 72 set printed as `2A.LO.S16.S8`;
  the two 2A.LO harvest rows baked `Ra = R4` / `Rc = R4`; sm121a's register rows lacked `-Rc`,
  `IDP.2A.LO.U16.U8_R_R_R_R` carried no mode bit and baked `Rc = RZ`, its two `_?` junk twins of the S8.S8 row are retired
  (`key_aliases` → `IDP.4A.S8.S8_R_R_R_R`). Counts: sm120 15 953 → 15 964 forms, 26 580 → 26 598 variants;
  sm103a / sm100a 14 913 → 14 925, 22 420 → 22 439; sm121a 23 850 → 23 860, 26 406 → 26 421. The lane's `42192f0` had
  carried its `_src` tag without a `_meta.provenance.sources` entry (cubit's validator refuses that) — the legend entry
  is in.
- **The named audit exceptions closed from measurement** (`audit-fix-2026-11-06-t11`, nvdisasm 13.1 on the four targets;
  cubit `tests/sm120_audit_t11.rs`): `DEPBAR_II_II [LE]` forbids the scoreboard values 6 / 7 (`INVALID6` / `INVALID7`);
  `FADD.SAT_R_R_R` (sm120, sm121a) retired — a SAT-less harvest row (no bit 77 in its and_base, `Ra = RZ` baked, its
  `abs@62` the `|Rb|` of a plain FADD; alias → `FADD_R_R_R`, the `[SAT]` group carries bit 77); `ULEA_UR_UR_UR_II []`
  gains `neg@72` (`-URa`, the first source; bit 63 stays `-URb`); `ULOP3_UR_UR_II_UR_II_UP [LUT]` gains the source
  predicate fields `upred@87` + `inv@90` (the harvest row baked `!UPT`; the UPT-destination words of the `_UP_` rows —
  printed by the vendor with six operands — decode through it now). With cubit's side of the same push (the
  `[???255.64+UR..]` glyph on the plain 64-bit-base LDG / STG / ST forms, `BMOV.64 ATEXIT_PC`, USETMAXREG off the
  sign-tolerant fallback) the replay of the complete audit word set at `c896e8f` (cubit `T/audit/dyn22`, 85 886 words)
  leaves **1 row**: the documented `IMMA.SF.16832.U8.U8` hypothesis (every probe word vendor-rejected; the owner's rule
  for undocumented forms).
- The four references regenerated at `c896e8f` (SM120 15 964 forms / 26 598 variants; SM103A / SM100A 14 925 / 22 439;
  SM121A 23 860 / 26 421); README: the rows without `_src` (18 841 of 26 598 SM120 variants), the audit end state, the
  IDP family in the "rebuilt from measurement" list.

## November 6, 2026 (session T-10): BRA on sm103a / sm100a, the BRX / BRXU / RET and dense IMMA families, the SM120 dynamic audit at its end state

- **`sm103a.json` / `sm100a.json`: the `BRA` family is the measured SM120 family** (cubit `tools/probes/re_sm120_bra.py
  probe --arch SM103` / `SM100` — the 551 words of the SM120 sweep print the same text on the three nvdisasm 13.1
  targets, the uniform-predicate form 0x547 included: the T-9 note that it was sm_120-only was wrong, the sm103a
  harvest row `BRA_UP_II [U]` had counted 17 806 such words; fixtures cubit `tests/fixtures/{sm103a,sm100a}_bra_nvdisasm.jsonl`,
  test `tests/sm103a_sm100a_bra.rs`): 5 harvest keys / 9 rows per table (`BRA_II` with a guard-only field list,
  `BRA_P_UR_II [CONV]` / `[DIV]` with `imm@18:6` + `imm@34:5` junk target fields, `BRA_UP_II [ANY,U]` with a `pred@24:4`
  field over the UP operand) → the same six keys / 66 rows as sm120 (`bra-family-2026-11-06`). Counts 14 912 → 14 913
  forms, 22 363 → 22 420 variants per table.
- **`sm120.json`: the `BRX` / `BRXU` / `RET` family measured** (cubit `tools/probes/re_sm120_brx_ret.py`, nvdisasm 13.1
  `-b SM120`, 506 form words every text predicted by the layout + 29 lattice words; provenance `brx-ret-family-2026-11-06`,
  test cubit `tests/sm120_brx_ret.rs`): register [31:24] (255 = RZ / URZ); the offset is the BRA split — a signed 56-bit
  dword at [23:16] + [81:34] — printed `Rn off*4` by BRX / BRXU (relative to the register, 0 elided: `BRX R8`) and
  RET.ABS, `Rn PC+16+off*4` by RET.REL; predicate operand [89:87] + `!` 90 before the register (`BRX !P3, R8 0x100`);
  BRX / BRXU 85 `.INC`, 86 `.DEC` (both = `INVALID3`), BRXU mode [33:32] plain / `.U` / `.DIV` / `.CONV`, bit 91 required
  on BRXU and illegal on BRX; RET 85 `.ABS` (clear = `.REL`, always printed), 86 `.NODEC`, bit 91 = the uniform-register
  form (`RET.REL URZ 0x110`); bits 82-84 and 92-95 text-inert. Eight keys `{BRX,BRXU,RET}[_P]_{R,UR}_II` / **46 rows
  replace 13 harvest keys / 16 rows** (`BRX_II` with `imm@18:20` + `reg@25:8` junk fields, `BRX_L`, `BRXU_II` — it printed
  `BRXU 0xfffe0220` for the vendor's `BRXU UR20 -0x1fdf0` —, `BRXU_L`, `RET_II`, `RET_L`, `RET.REL.NODEC_II` (no register),
  `RET.REL.NODEC_R`, `RET.ABS.NODEC_?` (no fields), the duplicated `RET_R_II` groups and the three `dsv41-sm120-2026-10-22`
  rows the measurement supersedes; aliases `RET.REL.NODEC_R_II` / `RET.REL.NODEC_R` → `RET_R_II`).
- **`sm120.json`: the dynamic audit of the harvest rows is at its end state — 14 rows, every one a named exception**
  (cubit `docs/KNOWN_ISSUES.md` T1, replay `T/audit/dyn21`: the T5 `IMMA.SF` hypothesis, the seven RZ-base 64-bit
  descriptor words cubit's decoder tolerates (BUG-461), the `UPT`-destination elision of ULEA / ULOP3, `BMOV.64
  ATEXIT_PC`, `DEPBAR SB7`, `FADD.SAT` bit 62, `USETMAXREG` bit 63; 22 before the dense IMMA family below). Five batches
  (blackwell-isa-internal `tools/audit_fix_20261106_t10.py` + `_t10b.py` … `_t10e.py`, provenance
  `audit-fix-2026-11-06-t10`, every vendor text asserted against nvdisasm 13.1 before the write; cubit
  `tests/sm120_audit_t10.rs`): **15 harvest-row field repairs** — ATOMG CAS
  Pd = [83:81] (the harvest's `pred@7` sat on the opcode bits), STS.64 opcode bit 9 baked (the 0x188 words are illegal)
  and the text-inert [71:64] fail-closed, the MOV / MOV.64 junk destination copies at [31:24] (text-inert, every census
  word 0), WARPSYNC.ALL has no operands, VOTE / VOTEU phantom predicate fields, IMAD.X Ra = RZ baked, UFMUL's three 8-bit
  URs at 16 / 24 / 32, UTMAREDG operands [39:32] / [31:24], USETMAXREG's 10-bit count [41:32], the LDCU size groups'
  uniform index field (U8's index is a UR), BAR.RED.{AND,OR} count-less twins rebuilt from the measured `_II_II_P` row
  (the vendor elides a zero thread count), HADD2_R_R_FI_FI fp16 immediates, the SYNCS RED.A1T0 destination [23:16], the
  F2I.NTZ UR-form fields — and **105 junk keys / 5 groups retired** under the census-witness rule (0 witnesses among the
  249 929 distinct words of the production census + our kernels, no alias target, every probe word vendor-invalid /
  illegal or decoded by the remaining rows): the `_?` partial keys (CS2R ×2, DADD, DMUL, F2I.TRUNC.NTZ ×3, FMNMX ×2,
  HADD2.F32 ×2, IADD.64, MOV ×4, PRMT, VIMNMX.S32, HADD2), the shifted-field / phantom-token twins (F2I ×3, FLO, I2FP,
  POPC, SEL, FSETP ×2, FSEL, IADD.64 ×6, IADD.X, IADD3 ×4, IADD, UIADD3 ×3, the `_P_R_R_R_R` HMMA ×5 / QMMA ×51 twins with
  2-bit-shifted register fields, HMMA.SP.1688, LDSM `_R_AUR` ×2, SULD.D.BA.2D (nvdisasm names its words
  `SULD.P.1D.EF.INVALID0.IGN`), PLOP3.LUT_P_P_P_P_UP_II_II, MOV_P_R_R) and the groups `IADD3_R_P_P_R_UR_R [X]`,
  `LDGSTS_ARI_ARURI_P [64,E,LTC128B]`, HADD2_R_R_II_II [""], MUFU_R_II [RCP64H], F2I_R_UR [NTZ,U32]. Three batch-1
  retirements were refuted by cubit's pins and **undone with the measurement** (`_t10c.py` / `_t10d.py`): `LD_R_ARURI [E]`
  (BUG-097: the plain U32+UR `LD.E` form — fields Rd [23:16] / Ra [31:24] / URb [39:32] / offset [63:40] measured),
  `IADD3_R_P_P_R_II_R [X]` (BUG-089's canonical row, restored verbatim), `LDGSTS_ARI_ARI_P` / `LDGSTS_ARI_ARURI_P
  [128,E,LTC128B]` (BUG-164; plus the PT-baked twin `LDGSTS_ARI_ARI [128,E,LTC128B]` — the vendor elides the PT
  operand, the two-operand text now encodes); `STS_ARI_R [64]` is the f2d67b6 row + opcode bit 9 baked (`_t10e.py`: the
  BUG-425 / 425b tolerance of the text-inert bits 72 / 76 / 77 / 80, [71:64], 91 stays as arbitrated in 2026-09).
- **`sm120.json`: the dense `IMMA` family measured** (cubit `tools/probes/re_sm120_imma.py`, nvdisasm 13.1 `-b SM120`, 240
  form words every text predicted by the layout + 16 lattice words; provenance `imma-family-2026-11-06`, test cubit
  `tests/sm120_imma.rs`; measured by the session's second instance T2, reviewed and applied by T-10): opcode 0x237, Rd
  [23:16], Ra [31:24] `.ROW`, Rb [39:32] `.COL`, Rc [71:64]; bit 86 the dense frame (clear = `INVALID0`), 75 the shape
  (`16816` / `16832`), 76 A = `S8` (else `U8`), 78 B = `S8`, 82 `.SAT`, 73 / 74 the only legal layouts (`ROW` / `COL`;
  `???1` / `???0` otherwise — baked, fail-closed), 77 / 79 / 83 / 84 / 85 the vendor's `INVALID` codes, the fifth operand
  `[!]UPn` = the 4-bit `pred_inv4` window [90:87] (`UPT` elided unless negated), bit 72 the sparse form (`IMMA.SP.*`,
  untouched), 91 illegal, 80 / 81 / 92-95 text-inert. The type pair is order-sensitive and a sorted mod-group name cannot
  tell `S8.U8` from `U8.S8`, so the family uses typed keys `IMMA.<16816|16832>.<A>.<B>[.SAT]_R_R_R_R[_UP]` — **32 keys ×
  1 row replace the 14 dense harvest keys** (2-bit-shifted register fields that printed `R4, R6.ROW, R8.COL, R10` for the
  vendor's `R16, R24.ROW, R8.COL, R40`, `neg` junk fields on the layout bits whose canonical words rendered `R8.???0`,
  the `_P_` / `_?` twins → `_meta.key_aliases`). No census witness decodes through a dense IMMA row; the audit's eight
  IMMA exception rows leave the table. `IMMA.SF.16832.U8.U8_R_R_R_R_R_R_UR` (the T5 hypothesis) is untouched.
- Counts (from the T-9 tables 16 045 forms / 26 646 variants): the BRX / BRXU / RET family 16 040 / 26 676, the audit
  batches 15 935 / 26 562, the IMMA family **15 953 / 26 580**; **18 845 rows without `_src`**. The production census
  stays 1 811 600 / 1 811 600.
- The SM120 reference regenerated at `c258f8a`, the SM103A / SM100A references at `c95e264` (their tables' last change).

## November 6, 2026 (session T-9): the SM120 BRA family rebuilt from measurement

- **`sm120.json`: the `BRA` family rebuilt from measurement** (cubit `tools/probes/re_sm120_bra.py`, nvdisasm 13.1
  `-b SM120`; the 0x947 words print identically on `-b SM103` / `-b SM100`, the 0x547 form is sm_120-only; 519 form
  words every text predicted by the layout + 32 lattice words; provenance `bra-family-2026-11-06`, test cubit
  `tests/sm120_bra.rs`): target = PC + 16 + a signed 56-bit dword offset ([23:16] + [81:34]; nvdisasm prints the sum
  signed, `BRA -0x14a0` for an isolated backward word); guard [15:12]; predicate operand [89:87] + `!` 90 (PT elided —
  the keys without `_P_` bake it, `BRA !PT, 0x50` is a word of its own); mode [33:32] plain / `.U` / `.DIV` / `.CONV`;
  85 `.INC`, 86 `.DEC` (both = `INVALID3`); 84 `.ANY` and 83 `.NOT_TID0` with `.U` only (inert otherwise); the UR
  operand [31:24] + `~` 82 behind the frame bit 91, legal with `.DIV` / `.CONV` only (`INVALID0/1` otherwise — the
  harvest `BRA_UP_II [U]` claimed those words); 0x547 + 91 = the uniform-predicate form `BRA.U[.ANY] [!]UPn, target`
  (UP [26:24], `!` 27; `.U` required, `???n` without it; bit 83 inert there; a vector predicate may precede the UP);
  bits 92-95 text-inert, outside the rows. Six keys `BRA_II`, `BRA_P_II`, `BRA_UR_II`, `BRA_P_UR_II`, `BRA_UP_II`,
  `BRA_P_UP_II` × one group per legal modifier set = **66 rows replace 15 harvest keys / 31 rows**: `BRA_II_II`,
  `BRA_II_II_?`, `BRA_II_II_II_?`, `BRA_II_II_II_II_?`, `BRA_P_II_II`, `BRA_P_II_II_?` (junk twins of `BRA_II` /
  `BRA_P_II`), `BRA.U_?_II`, `BRA.DIV_UR_II`, `BRA.U_UP_II` (now `_meta.key_aliases` of the plain keys), the
  triplicated `CONV` / `DIV` groups with `guard@29` / `imm_shr3@28` junk fields, `BRA_UP_II [""]` / `BRA_UR_II [""]`
  (twins of `BRA_II [""]`, bits 29 / 30 baked), `BRA_P_UR_II [CONV]` / `[DIV]` (they printed `PT, UR6` for the
  vendor's `~UR6`), `BRA_P_P_II`, `BRA_II [NOT_TID0,U]`. Counts: 16 054 → 16 045 forms, 26 611 → 26 646 variants,
  19 000 rows without `_src`. The production census stays 1 811 600 / 1 811 600. The sm103a / sm100a BRA rows are
  still harvest rows (the measured rows apply there verbatim minus the 0x547 form — a follow-up).
- The SM120 reference regenerated at this revision.

# blackwell-isa Changelog

## November 6, 2026 (session T-8): the LEA family rebuilt from measurement on the three tables, the ATOMS uniform-indexed lattice, junk keys retired, the audit oracle corrected

- **`sm120.json` / `sm103a.json` / `sm100a.json`: the `LEA` family rebuilt from measurement** (cubit
  `tools/probes/re_sm120_lea.py`, nvdisasm 13.1 `-b SM120`; `-b SM103` / `-b SM100` text-identical for every
  one of the 824 probe words per target; provenance `lea-family-2026-11-06`, test cubit
  `tests/sm120_lea.rs`): `LEA[.HI][.X][.SX32] Rd, [Pd,] Ra, <b>, [Rc,] imm5 [, Pcin]` in its three operand
  forms — 0x211 register, 0x811 imm32 [63:32], 0xc11 uniform register (frame bit 91) — × the six legal
  modifier groups (bit 80 `.HI`, 74 `.X`, 73 `.SX32`, the latter legal only with `.HI`) × {a `Pd` token,
  `PT` baked} = 36 rows on 24 keys `LEA_R_[P_]R_<b>[_R]_II[_P]`. Fields: guard [15:12], Rd [23:16], Pd
  [83:81] (the vendor elides `PT`), Ra [31:24], Rb / URb [39:32], Rc [71:64] (printed by `.HI` without
  `.SX32` only), imm5 [79:75], carry-in [89:87] + `!` 90 (printed by `.X` only); the sign bits 72 (Ra) and
  63 (Rb / URb) are the arithmetic negation `-R` on the forms without `.X` and the **bitwise inverse `~R`
  on the `.X` forms** (`inv` fields); the text-inert bits (Rc and the carry-in where not printed, 84-86,
  92-95, the imm32 window on the register / UR forms) are baked at nvcc's values (Rc = RZ, carry-in
  `!PT`, the rest clear — checked on the 26 372 LEA words of the production census) and stay outside the
  mask (fail-closed). Before: 56 keys / 100 harvest rows on sm120 — dozens of `?` junk keys with one
  sample's operands baked into `and_base` and fields at the wrong bit, the modifier bits neither in
  `and_base` nor in the mask (the plain `LEA R4, R8, R10, 0x1` word decoded as `LEA.HI.X R4, R8, R10,
  RZ, 0x1, PT`), the `.X` carry-in `, PT` dropped from the text, `~R12` printed as `-R12`; on sm103a /
  sm100a coldface's B300-1 / B300-4 rows of 2026-11-05 (UR form) agree with the measurement field by
  field except the sign glyph of the `.X` row (`neg` → `inv`) and are replaced by the family rows
  together with the harvest rows next to them that shadowed their `Pd = PT` words. Keys sm120
  16 097 → 16 065 (variants 26 663 → 26 599), sm103a / sm100a 14 906 → 14 912 (22 348 → 22 363).
- **`sm120.json`: F2FP junk twins retired** (`blackwell-isa-internal tools/audit_fix_20261106.py`): the
  keys `F2FP.BF16.F32.PACK_AB_R_R_R_?` and `F2FP.F16.F32.PACK_AB_R_R_R_?` (harvest counts 22 / 32, `Ra = RZ`
  and the guard baked into `and_base`) — every word they claim decodes through the measured
  `F2FP_R_R_R` PACK_AB rows with the same text — and the byte-identical duplicate group `TF32,F32,PACK_B`
  of `F2FP_R_R` (`F32,PACK_B,TF32` stays). Keys 16 065 → 16 063, variants 26 599 → 26 596. The cubit
  printer now prints the `UNPACK_B_MERGE_C` conversions in the vendor's order (`F2FP.E4M3.BF16.
  UNPACK_B_MERGE_C`: source type first — 96 audit words).
- **`sm120.json`: `ATOMS_R_ARURI_R` measured over its whole operation × type lattice** (cubit
  `tools/probes/re_sm120_atoms_aruri_ops.py`, nvdisasm 13.1 `-b SM120`, 720 form + 27 lattice words;
  provenance `atoms-aruri-ops-sm120-2026-11-06`, test cubit `tests/sm120_atoms_aruri_ops.rs`):
  `ATOMS.<op>[.S32|.64|.128] Rd, [Ra+URb(+imm)], Rb` (0x98c + frame bit 91) — operation [90:87] ADD / MIN /
  MAX / INC / DEC / AND / OR / XOR / EXCH (9+ `INVALIDn`), type [75:73] plain / S32 / 64 / 128 (others
  `INVALIDn`), guard, Rd, Ra (RZ elided: `[UR4]`), Rb, signed 24-bit offset, URb (255 = URZ) — 36 groups replace
  the one measured on 2026-10-29 (`MAX,S32`) and six harvest groups that read the UR byte as a fourth register
  (`ATOMS.AND R4, [R7+UR4], R8` printed `ATOMS.AND R4, [R7], R8, R4`); retired with them the four 0x98c junk
  groups of `ATOMS_R_ARI_R_R` and the mis-keyed `ATOMS_R_AURI_R` [EXCH] (a base-register field under the RZ-base
  key; it absorbed `ATOMS.INVALID10` words as `EXCH`). Variants 26 596 → 26 611.
- **`sm120.json`: nine harvest keys whose every probe word nvdisasm rejects retired** (`DADD_R_II_R`,
  `DMUL_R_R_R_II_?`, `DMUL_R_R_R_II_II_II_?`, `DMUL_R_R_R_R`, `MOV_R_UR_II_?`, `MOV_R_UR_II_II_?`,
  `MOV.64_R_UR_II_?`, `MOV.64_R_UR_II_II_?`, `MOV.64_R_UR_UR`; `blackwell-isa-internal tools/audit_fix_20261106.py`):
  re-measured after the oracle correction below, no witness among the 237 844 distinct words of the
  production census, no `key_aliases` target, a measured sibling carries the real form (`MOV_R_UR` 0xc02,
  `DMUL_R_R_{R,II,UR,FI}`, `DADD_R_R_*`). Keys 16 063 → 16 054.
- **Dynamic-audit oracle corrected:** the 2026-11-04 nvdisasm record set behind the audit replays had
  386 legal words marked as rejected through a scratch-file race in cubit's `tools/audit_dynamic.py`
  (two rows probing the same word in parallel); re-measured on 2026-11-06 and the tool fixed. With the
  corrected records, the LEA rebuild, the ATOMS lattice and the retired keys the audit stands at **230 rows**
  with at least one defect (418 on the T-7 tables); rows without `_src` are 19 025 / 15 873 / 15 873 / 25 696
  (sm120 / sm103a / sm100a / sm121a). The three references regenerated at this revision.

## November 6, 2026 (session T-7): HADD2 register form, the `.EL` 64-bit twin, coldface's B300 rows reviewed, wave-11 provenance stamped, the references regenerated

- **`sm120.json` / `sm103a.json` / `sm100a.json`: the 64-bit-base twin of the plain `.EL` atomic**
  (`cubit tools/probes/re_sm120_atomg_el_aruri64.py`, nvdisasm 13.1, 74 form + 8 lattice words per target,
  text-identical on the three; provenance `atomg-el-aruri64-2026-11-06`): `ATOMG.E.{ADD,OR}.EL.STRONG.GPU
  Pd, Rd, [Ra.64+URb(+off)], Rb` — the fields of the 2026-11-05 U32 rows (guard [15:12], Pd [83:81], Rd
  [23:16], Ra [31:24], Rb [39:32], URb [71:64], offset [62:40] signed 23-bit) with the width bit 63 set,
  bit 76 (the descriptor rows) excluded; base 255 prints the degenerate `???255.64` glyph like the REDG
  twin. The U32 rows exclude bit 63 since 2026-11-05, so these words had no row (the four `[R8.64+UR7]`
  decode misses of cubit's dynamic audit). Keys +2 per table.
- **`sm120.json`: `HADD2_R_R_R` carries 12 measured groups** (`cubit tools/probes/re_sm120_hadd2_sel.py`,
  nvdisasm 13.1 `-b SM120`, 189 form + 28 lattice words; provenance `hadd2-rrr-sm120-2026-11-06`):
  `HADD2[.F32|.BF16_V2][.FTZ][.SAT] Rd, Ra, Rb` with the fp16 half-selectors Ra [75:74] / Rb [61:60]
  (0 natural, 2 `.H0_H0`, 3 `.H1_H1`; code 1 = nvdisasm `INVALID1`, outside the rows), `-Ra` 72, `|Ra|` 73
  (text-inert under `.F32`, not a field there), `-Rb` 63, `|Rb|` 62, `.SAT` 77, `.F32` 78, `.FTZ` 80,
  `.BF16_V2` 85 (`.F32` + `.BF16_V2` illegal), bit 91 illegal, every other bit of [64:96) text-inert.
  Before: the plain group had no `|Ra|` / `|Rb|` fields, `BF16_V2` no sign fields at all, and the `F32`
  group was a harvest sample with `-Ra`, `Rd = RZ` and `Rb.H0_H0` baked into `and_base`; a word with
  `|R5|` fell to the junk row `HADD2_R_R_II_II_II_II_?` and printed `HADD2 R9, R2, 0x7000, 0x5, 0, 0,
  UR0`. Removed: the dotted twins `HADD2.F32_R_R_R`, `HADD2.BF16_V2_R_R_R` (their words are the plain
  key's groups) and the register-opcode junk rows `HADD2_R_R_R_R`, `HADD2_R_R_R_II_?`,
  `HADD2_R_R_R_II_II_?`, `HADD2_R_R_II`, `HADD2_R_R_II_II_II_II_?`. Keys 16 103 → 16 096, variants
  26 656 → 26 658. Found while measuring the form behind cubit's legacy input spelling `HADD2.H1H1A`
  (cubit BUG-484: the 2026-08-08 mnemonic-suffix row `8f773da`, retired at the O2 cut; the production
  `exl3-wave-m8` kernels were written with it) — the spelling is a cubit parser alias now, the table
  carries only the vendor's text.

- **Wave-11 provenance stamped** (`afc8162`, author coldface, run by T-7 with its `stamp_provenance.py`): the
  5 736 / 5 748 / 5 744 / 44 rows the integration added without a `_src` carry `wave11-atom-lattice-2026-11-05`
  (the BUG-509 ATOM/ATOMG [63:60] lattice, 5 692 / 5 697 / 5 693 / 3) or `wave11-integration-2026-11-05`
  (the lane-lineage rows of the merge, 44 / 51 / 51 / 41); zero bit changes (deep comparison), every tag's
  `rows` recounted from the data (the tool had written the "added since bb11659" counts — 84 for the 160 SHF
  rows). Rows without `_src` are now exactly the 2026-06..08 harvest: 19 140 / 15 892 / 15 892 / 25 696.
- **coldface's B300 rows (sm103a / sm100a), re-measured by T-7 on nvdisasm 13.1 `-b SM103` / `SM100` /
  `SM120` and taken:** `LEA_R_P_R_UR_R_II` / HI rebuilt (B300-1: guard [15:12], Pd [83:81], `-Ra` 72, `-URb`
  63, imm [79:75], bit 0 = the LEA / IADD3 discriminator — 62 vendor IADD3 words had shadowed into the old
  row and re-encoded as IADD.XOR); `LDG_R_dARI` / E,SM,STRONG,U16 gets the desc offset [63:40] signed
  (B300-2a, sm100a + sm103a: 13 680 census words re-encoded with the offset zeroed); `LDGSTS_ARURI_ARI_P` /
  128,BYPASS,E,LTC128B loses the phantom `sub_imm1_shr5` field (B300-2b: the imm is 12-bit signed at
  [43:32], [87:85] are `.LU` / `.NA` and the predicate LSB) and its UR field is 8 bits (bit 72 is the LTC
  size, T-7); `UTMALDG_AURI_AURI` / 2D reactivated with measured fields (B300-3: 70 census words of
  libcublas.so.75); `IADD.XOR_R_P_R_UR_R` and `LEA.HI.X_R_P_R_UR_R_II_P` (B300-4). Hygiene on all of them:
  per-row `_src`, and_base clear in the field spans, mask = field union. **Not taken:** coldface's
  `IADD3.X_R_P_P_R_UR_R_P` — it baked the second carry-in predicate as a constant `!PT` and printed `-R` for
  the inverted operands; measured, the vendor text carries two carry-in predicates (Pc [89:87] + negation 90,
  Pd [79:77] + negation 80) with the `~` glyph — the harvest row `IADD3_R_P_P_R_UR_R_P_P` / X (count 20 864)
  keeps both and gets its one repair, a 3-bit `pred@87` + `neg@90`, on **sm120 / sm103a / sm100a**
  (`iadd3x-urform-carry-neg-2026-11-06`); the proposed retirement of that row is not applied.
- **`UTMALDG` plain `[URa], [URb]` form, five dimensions on the three tables** (`utmaldg-plain-dims-2026-11-06`):
  the enum [81:79] = 1D / 2D / 3D / 4D / 5D (5-7 INVALID); sm120 had no plain key — its dotted era keys
  `UTMALDG.1D_AUR_AUR` / `UTMALDG.2D_AUR_AUR` shared one and_base without the dimension bits and are retired.
- The four HTML references regenerated at these tables (`export_html.py`); README provenance paragraph.
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
