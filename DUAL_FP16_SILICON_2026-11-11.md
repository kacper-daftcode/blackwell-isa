# Dual FP16 silicon results: SM103a and SM100a

Date: 2026-11-11 (measurement); 2026-11-13 (this record).

This document records the results of two execution series of the B300 measurement
lane. It changes no instruction table, generated reference, claim annotation
or canonical semantic status.

The raw records, the executed images and the measurement and replay harness are
archived privately (231 files, 3 956 323 bytes; the SHA-256 aggregate of the
archive's manifest is
`60fb543d7fb6b81c0a65d8f2852084845e72210970afb77c24eacc3bd6a27fec`); they are
in neither repository. cubit carries the results in a form that can be
re-checked on a CPU: `tests/fixtures/sm103a_sm100a_fhadd_hset2_silicon.jsonl`
holds, for each of the 18 measured words, the vendor's text and the 352 rows
(both operand registers and the result register) of each series, and
`tests/hidden_b300_fp16_silicon.rs` decodes and re-assembles the words on both
tables and recomputes from the rows every statement marked *(re-checked)* below.
Statements without the mark rest on the archive alone.

## FHADD execution and scoped semantics

Two native GPU series are independently retained:

- B300 SXM6 AC, SM103a, driver 595.91.07, CUDA 12.8 host toolchain;
- B200, SM100a, driver 580.178.04, CUDA 13.4 host toolchain.

Each completed 187 launches with one CTA, 32 threads, four guarded 384-byte
owned buffers and bounded subprocess timeouts. Per host this is 5632 core
outputs plus 352 separately labelled replay outputs, not 5984 new core values.
All 5984 corresponding raw values match across architectures. The canonical
raw-observation SHA256 is
`079cc0b092252439b71a51627d0c87e1861277809e0fc41d1de400a1f0d28a21`
on each host.

The accepted numerical statement is **SEMANTICS_PASS_SCOPED_NON_SAT**. For the
16 measured images, full FP32 addition of the selected F16/BF16 H0/H1 narrow
half and the full FP32 operand matches the exact-rational model under the
exercised rounding variants *(re-checked: the 4800 rows with two finite
operands bit for bit, 34 of them overflowing; the 152 rows with an infinite
operand; a NaN in each of the 680 rows where the model gives one)*. The
separate BF16-H1 replay matches its original.
This does not claim SAT, other operand selectors/roles, universal NaN-payload
behavior, timing, cache behavior or a general hardware property. Two cases of
the model are not among the measured rows: the sum of two negative zeros, and
an exact sum beyond the largest finite FP32 number under a rounding direction
that keeps the result finite. The newer
CPU-only audit rechecks cached observations; no additional GPU validation is
claimed from it.

The replay is deliberately a duplicate observation, not additional core
coverage. The architecture-specific images are not assumed binary-identical.
An original wrong B200 donor encountered CUDA 209 before execution and was not
accepted. Proper standalone images carry correct target, entry parameter and
EXIT metadata; changing only an ELF architecture flag is not evidence.

## HSET2: raw effect only

For `HSET2.EQ.AND` F16 and `HSET2.BF16_V2.BF.EQ.AND`, the A=B control dataset
has 22 completed launches per host, 352 F16 and 352 BF16 values per host,
expected echo and intact recorded canaries. The values match exactly across
architectures. This is scoped **SILICON-EFFECT and reproducibility** evidence,
not semantic acceptance of Eq.

A separate non-mirrored dataset falsifies the simple per-halfword equality mask
with 9 F16 and 40 BF16 expected-vs-actual mismatches per architecture out of
352 values per form *(re-checked)*. The complete mismatch arrays are retained
in the archive. The HSET2
result does not derive a new semantic model by relabelling these mismatches,
and `EQ.XOR` is not accepted through the conflicting assembler/parser path.

The two stored HSET2 datasets and their hashes are distinct. In particular,
the older `8cced12a...`/`29f69287...` values serialize nested JSON batches of
hex strings. The byte-oriented little-endian u32 SHA256 values for the newer
A=B outputs are instead *(re-checked)*:

- F16: `6ad8bf38170ce8be55741c0bd84cf2fff909b3f23cd26d8e688f39e328e5875a`;
- BF16: `6d708ae82e7d5aad89a63766389365e5a7b9163ee0271dcc3c02171d447b0c72`.

The archive specifies both encodings and keeps the original records
unchanged. The non-mirrored model-falsification series has separate digests
(the same serialization, *re-checked*: F16
`bbd953f5e5119347c7bb52f7b81098aeab306ed29341898f19a44db2b5209982`, BF16
`374e9a24e5e9f32fddd9a83e7418fe2f3c0d99648f9834d7ad0ceada65049ba8`)
and is not counted as extra A=B coverage.

**Review note (2026-11-13, on the stored rows, CPU only).** The rejected mask
pairs H1 of the first source with H1 of the second and expects `0xffff` for
"equal" in both forms. The two measured words are
`HSET2.EQ.AND R10, R2, R4.H0_H0, PT` and
`HSET2.BF16_V2.BF.EQ.AND R10, R2, R4.H0_H0, PT`. With each half of the first
source compared against H0 of the second, as the operand text `R4.H0_H0`
reads — IEEE equality: a NaN equals nothing, -0 equals +0 — and with `0xffff`
as the value for "equal" in the first word and BF16 1.0 (`0x3f80`) in the
second, all 1408 stored values of the two series and two words are reproduced
*(re-checked)*. This is a statement about these rows and these two words. No
other comparison, half selector or boolean operand was measured, and the note
changes no canonical semantic status.

## CPU-only verification and provenance

The archive includes 34 FHADD raw JSON files, four
HSET2 A=B raw JSON files, four non-mirrored falsification JSON files, selected
final builders/models/driver, SASS, decoded text, prepare metadata and cubins.
It excludes preparation-only wrappers, caches, donor junk and secrets.

The archive's `verify.py` recomputes exact FHADD finite and special-class
expectations, replay identity, HSET2 model failures, input/echo checks, raw
equality, operation-word identity and address bounds. Its `cpu_replay.py`
validated 38 selected cubin paths (36 unique cubin hashes, since replay images
are duplicates), giving 646 strict whole-word comparisons. The matching vendor
decodes and complete ELF records were checked, including four pointer
parameters and `EIATTR_EXIT_INSTR_OFFSETS=0x100`.

This verification launches no GPU. Historical cubit/table/tool versions and
mixed donor-versus-final container metadata are explicitly documented in the
archive's report. GPU JUnit results are neither claimed nor synthesized.
