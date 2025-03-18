_The following form is designed to provide information for your tool that should be included in the effort to verify the Rust standard library. Please note that the tool will need to be **supported** if it is to be included._

## Tool Name
**KMIR**

## Description
[KMIR](https://github.com/runtimeverification/mir-semantics) is a formal verification tool for Rust that defines the operational semantics of Rust’s Middle Intermediate Representation (MIR) in K (through Stable MIR). By leveraging the [K framework](https://kframework.org/), KMIR provides a parser, interpreter, and symbolic execution engine for MIR programs. This tool enables direct execution of concrete and symbolic input, with step-by-step inspection of the internal state of the MIR program's execution, serving as a foundational step toward full formal verification of Rust programs. Through the dependency [Stable MIR JSON](https://github.com/runtimeverification/stable-mir-json/) KMIR allows developers to extract serialised Stable MIR from Rust’s compilation process, execute it, and eventually prove critical properties of their code. It is available via our package manager, [kup](https://github.com/runtimeverification/kup), making it easily installable and integrable into various workflows.

This diagram describes extraction and verification workflow for KMIR:
**TODO**: Paste the image into the issue https://drive.google.com/file/d/1KNftl47__f1dWWUWwrhzbWhNwQHfhwxh/view?usp=sharing

Particular to this challenge, KMIR verifies program correctness using the correct-by-construction symbolic execution engine and verifier derived from the K encoding of the Stable MIR semantics. The K semantics framework is based on reachability logic, which is a theory describing transition systems in [matching logic](http://www.matching-logic.org/). Transition rules of the semantics are rewrite steps that match patterns and transform the current continuation and state accordingly. An all-path-reachability proof in this system verifies that a particular _target_ end state is _always_ reachable from a given starting state. The rewrite rules branch on symbolic inputs covering the possible transitions, creating a model that is provably complete, and requiring unification on every leaf state. A one-path-reachability-proof is similarto above but the proof requirement is that at least one leaf state unifies with the target state. One feature of such a system is that the requirement for an SMT is minimized to determining completeness on path conditions when branching would occur.

KMIR also prioritises UI with interactive proof exploration available out-of-the-box through the terminal KCFG (K Control Flow Graph) viewer, allowing users to inspect intermediate states of the proof to get feedback on the the path conditions that are successful and failing at unifying with the target state.
**TODO** Show diagram of Jost's max KCFG https://github.com/runtimeverification/mir-semantics/tree/tmp/example-proof/scratch/maximum-proof

## Tool Information

* [x] Does the tool perform Rust verification? 
* [x] Does the tool deal with *unsafe* Rust code? 
* [x] Does the tool run independently in CI? 
* [x] Is the tool open source?
* [x] Is the tool under development? 
* [x] Will you or your team be able to provide support for the tool?

## Comparison to Other Approved Tools
The other tools approved at the time of writing at Kani, Verifast, and Goto-transcoder (ESBMC).

- **Verification Backend:** KMIR is primarily different to all of these tools by utilising a unique verification backend through the K framework and reachability logic (as explained in Description above). KMIR has little dependence on a SAT solver or SMT solver. Kani's CBMC backend, and Goto-transcode (ESBMC) encode the verification problem into a SAT / SMT verification condition to be discharged by the appropriate solver. Kani recently has added a Lean backend through Aeneas, however Lean does not support matching or reachability logic currently, **TODO: CHECK (Not sure if true or worth mentioning)** and full automation of these proofs may not be possible yet. Verifast performs symbollic execution of the verification target like KMIR, however reasoning is performed by annotating functions with design-by-contract components in separation logic.
- **Verification Input:** KMIR takes input of Stable MIR JSON, an effort to serialise the internal MIR in a way that is portable and can be reusable by other projects.
- **K Ecosystem:** Since there is a common foundation of K for all tools in the K ecosystem, all projects benefit from development done by other K projects. This means that performance and user experience are projected to improve due to continued developement on other semantics. 

## Licenses
KMIR is released under an OSI-approved open source license. It is distributed under the BSD-3 clause license, which is compatible with the Rust standard library licenses. Please refer to the KMIR GitHub repository for full license details.

## Steps to Use the Tool

At RV, we generally strives to use [kup](https://github.com/runtimeverification/kup) , a `nix`-based software installer, to distribute our software.
This is future work, at the moment `kmir` requires a local build from source using the `K Framework` (installed with `kup`).

The future workflow we imagine is to
1. Download [kup](https://github.com/runtimeverification/kup) and install
2. Install KMIR `kup install kmir`
3. Compile the desired verification target with `kmir build module.rs`
   The `module.rs` should contain functions that act as property tests and are run with symbolic inputs.
4. Verify the target with `kmir prove module.rs --target target_function`
   This executes `target_function` with symbolic arguments. The test is expected to contain assertions about computed values.
5. Inspect KCFG of proof with `kmir view module::target_function`

At the time of writing, step 3 requires manual work to set up a _claim_ in the K language.
We will automate this process in the frontend code to avoid the user having to write K code.

## Artifacts & Audit Mechanisms

**TODO add papers about Matching Logic and about K framework**  
**TODO add links https://kontrol.runtimeverification.com and to the `evm-semantics` repository as examples of K framework integration.**  
**TODO craft 1-2 more examples of proofs we can already do. `unchecked_xyz` function challenge 11?**  

_If there are noteworthy examples of using the tool to perform verification, please include them in this section. Links, papers, etc._
_Also include mechanisms for the committee to audit the implementation and correctness of this tool (e.g., regression tests)._

## Versioning
KMIR and Stable MIR JSON are both developed using git, and will have semantic version numbers as soon as releases are made.
Stable MIR JSON depends on a nightly Rust compiler of a particular version (which is regularly updated, currently `nightly-2024-11-29`).

## CI
_If your tool is approved, you will be responsible merging a workflow into this repository that runs your tool against the standard library. For an example, see the Kani workflow (.github/workflows/kani.yml). Describe, at a high level, how your workflow will operate. (E.g., how will you package the tool to run in CI, how will you identify which proofs to run?)._
