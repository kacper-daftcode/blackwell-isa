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
- [`sm121a.json`](sm121a.json) — canonical SM121a ("GB10", DGX Spark Grace-Blackwell) ISA database; same schema, delta-driven vs SM120 with EXACT gates per entry; 15,443 encoding keys (md5 88142da9). Counting note: the SM121a table keeps a **finer per-key granularity** — its `instructions` dict holds one key per encoding row, so raw key counts are not comparable with the canonical instruction-*form* counts of sm120.json/sm103a.json
- [`SM120_ISA_REFERENCE.html`](https://kacper-daftcode.github.io/blackwell-isa/SM120_ISA_REFERENCE.html) — generated, searchable HTML reference ([browse online](https://kacper-daftcode.github.io/blackwell-isa/SM120_ISA_REFERENCE.html))
- [`SM103A_ISA_REFERENCE.html`](https://kacper-daftcode.github.io/blackwell-isa/SM103A_ISA_REFERENCE.html) — generated, searchable HTML reference for SM103a ([browse online](https://kacper-daftcode.github.io/blackwell-isa/SM103A_ISA_REFERENCE.html))
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

## Dataset Summary

SM120 (`sm120.json`):

| Item | Count |
|------|------:|
| Instruction forms (`instructions`) | 2,001 |
| Opcode families | 781 |
| Encoding variants (`mod_groups`) | 2,872 |
| Instruction forms with scheduling metadata | 1,865 |
| Scheduling-only entries | 593 |
| Pipeline classes | 37 |
| Instruction width | 128 bits |

SM103a (`sm103a.json`):

| Item | Count |
|------|------:|
| Instruction forms (`instructions`) | 397 |
| Opcode families | 163 |
| Encoding variants (`mod_groups`) | 1,291 |
| Corpus samples behind encoding records | 1,941,564 |
| Scheduling-only entries | 0 |
| Pipeline classes | 37 |
| Control-word classes | 29 |
| Instruction width | 128 bits |

Validation status for this release:

- 47,244 real instructions decoded across 178 cubins with 100% decode coverage
- 5,014 / 5,014 roundtrip fuzz cases passing through the companion assembler
- 936 / 936 targeted ground-truth instructions round-tripped, including the
  25-type `QMMA.SF` ptxas corpus and `LDG.E.LTC128B.128`
- all 36 dense `QMMA.SF...E8` type pairs covered: 25 emitted by ptxas and all
  11 combinations containing undocumented `E3M4` executed on RTX PRO 6000
- selected semantic bit fields validated by hardware patch tests on RTX 5090
- SM103a: disassemble→assemble roundtrip byte-exact on 323/323 cubins of the publish
  corpus (tcgen05 probe kernels, FA4 kernels, instruction samples) run under the exact
  published table; per-cubin verdicts identical to the production cubit table
  and RTX PRO 6000

## Data Model

At a high level:

```text
sm120.json
├── _meta
│   ├── architecture, codename, gpu, instruction_width
│   ├── stats
│   ├── pipe_classes
│   ├── ctrl_classes / ctrl_epochs
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
└── pipeline_config          # resource defaults, latency tables, per-opcode records
```

A typical encoding entry:

```json
{
  "and_base": "0x0000000000000000000000000000082e",
  "variable_mask": "0x1c00000000000000000000000000f000",
  "fields": [
    { "shift": 12, "bits": 4, "token_idx": 0, "extraction": "guard" },
    { "shift": 16, "bits": 8, "token_idx": 1, "extraction": "reg" }
  ]
}
```

`and_base` is the constant part of the 128-bit instruction word. `variable_mask`
identifies operand-dependent bits. `fields` tells an assembler/disassembler where to
read or write each operand field.

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
  `scheduling` and `ctrl_class` fields rather than deriving hazards from opcode names.

- **Control-word handling is instruction-class dependent.** The upper control bits are
  not a uniform free field. `ctrl_class` and `ctrl_epochs` describe which bits are fixed
  discriminators and which bits a scheduler can safely modify.

- **The public memory hierarchy numbers are easy to misread.** The database records
  RTX 5090 L2 as 96 MB; 128 MB is the access-policy window, not the cache size.

## Limitations

- `sm120.json` targets SM120 / consumer Blackwell; `sm103a.json` targets SM103a (B300). The pair is not a complete SM100/B200 ISA, though the measured tcgen05 encoding layer is byte-identical between SM100a and SM103a.
- Some entries are scheduling-only: they describe pipeline behavior without a complete
  encoding record.
- Some instruction names use provisional labels (`INVALID*`, recovered modifier names,
  or internal pattern names) where NVIDIA has no public terminology.
- Metadata under `_discovery` records provenance and confidence for unusual entries; users
  should inspect it before relying on those forms in production code.

## Related Work

- [`cubit`](https://github.com/kacper-daftcode/cubit) — companion SM120/SM103a SASS
  assembler/disassembler using this database.

## Citation

If this artifact is useful in a paper or project, please cite the repository and the
commit hash of `sm120.json` used in your work.

## License

MIT — see [`LICENSE`](LICENSE).

Reverse engineering for interoperability is protected under EU Directive 2009/24/EC
Article 6 and US DMCA §1201(f).
