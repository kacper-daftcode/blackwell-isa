# Dual FP16 silicon evidence: SM103a and SM100a

Date: 2026-11-11.

This document accompanies an evidence-only candidate for independent and T
review. It changes no instruction table, generated reference, claim annotation
or canonical semantic status. It is not a main integration, merge, independent
approval or Done transition.

The complete raw evidence and offline replay harness are on the same-named
`hidden-b300/dual-fp16-silicon-2026-11-11` cubit branch at
`docs/evidence/dual-fp16-silicon-2026-11-11/`. The paired PRs carry direct
links. Original table provenance remains attached to those measurement inputs;
no prior single-family ref is recreated or represented as canonical.

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
exercised rounding variants. The separate BF16-H1 replay matches its original.
This does not claim SAT, other operand selectors/roles, universal NaN-payload
behavior, timing, cache behavior or a general hardware property. The newer
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
352 values per form. The complete mismatch arrays are retained. The HSET2
result does not derive a new semantic model by relabelling these mismatches,
and `EQ.XOR` is not accepted through the conflicting assembler/parser path.

The two stored HSET2 datasets and their hashes are distinct. In particular,
the older `8cced12a...`/`29f69287...` values serialize nested JSON batches of
hex strings. The byte-oriented little-endian u32 SHA256 values for the newer
A=B outputs are instead:

- F16: `6ad8bf38170ce8be55741c0bd84cf2fff909b3f23cd26d8e688f39e328e5875a`;
- BF16: `6d708ae82e7d5aad89a63766389365e5a7b9163ee0271dcc3c02171d447b0c72`.

The companion bundle specifies both encodings and keeps the original records
unchanged. The non-mirrored model-falsification series has separate digests
and is not counted as extra A=B coverage.

## CPU-only verification and provenance

The complete cubit evidence bundle includes 34 FHADD raw JSON files, four
HSET2 A=B raw JSON files, four non-mirrored falsification JSON files, selected
final builders/models/driver, SASS, decoded text, prepare metadata and cubins.
It excludes preparation-only wrappers, caches, donor junk and secrets.

`verify.py` recomputes exact FHADD finite and special-class expectations,
replay identity, HSET2 model failures, input/echo checks, raw equality,
operation-word identity and address bounds. `cpu_replay.py` freshly validates
38 selected cubin paths (36 unique cubin hashes, since replay images are
duplicates), giving 646 strict whole-word comparisons. The matching vendor
decodes and complete ELF records were checked, including four pointer
parameters and `EIATTR_EXIT_INSTR_OFFSETS=0x100`.

This verification launches no GPU. Historical cubit/table/tool versions and
mixed donor-versus-final container metadata are explicitly documented in the
cubit report. Original absolute source paths are retained as provenance;
standalone replay entry points use repository-local paths and explicit tool
arguments. GPU JUnit results are neither claimed nor synthesized.
