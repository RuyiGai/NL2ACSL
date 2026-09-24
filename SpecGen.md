# Summary

This paper addresses the limitations of traditional program specification generation methods, which often rely on fixed templates and struggle to capture complex program semantics, as well as the problem that LLM-generated formal specifications may fail formal verification. To address these issues, the authors propose **SpecGen**, an automated approach for generating rich and verifiable Java JML formal specifications. SpecGen uses **LLM + few-shot prompting + iterative conversation**, with OpenJML verification errors as feedback for refinement; if generation still fails, it applies four mutation operators—**Predicative, Logical, Comparative, and Arithmetic**—with heuristic selection to search for a correct specification. Experiments show that SpecGen generated verifiable specifications for **279 of 385 programs**, outperforming AutoSpec (247) and Houdini (98), while achieving a **semantic quality score of 4.54/5**, close to expert-written oracle specifications (4.83/5). Its limitations include difficulty with **complex programs** and limited support for specification types such as **frame conditions and termination specifications**.


# SpecGen Core System Architecture
                 Java Program
                      │
                      ▼
          ┌──────────────────────┐
          │   Phase 1            │
          │ Conversation-Driven  │
          │ Specification Gen.   │
          └──────────────────────┘
                      │
                      ▼
                GPT-3.5 LLM
          + Few-shot Prompting
                      │
                      ▼
              JML Specification
                      │
                      ▼
                ┌───────────┐
                │  OpenJML  │
                │ Verification│
                └───────────┘
                   │       │
             Pass  │       │ Fail
                  ▼        ▼
              Final     Error Feedback
              Spec          │
                            ▼
                     Back to LLM
                     (Iterative
                      Refinement)
                            │
                 up to 10 rounds
                            │
                            ▼
                  Still Fails?
                       │
                       ▼ Yes
          ┌─────────────────────────┐
          │   Phase 2               │
          │ Mutation-Based          │
          │ Specification Generation│
          └─────────────────────────┘
                       │
                       ▼
              Generate Mutations
        ┌────────┬────────┬────────┬────────┐
        │Predic. │Logical │Compar. │Arithmetic│
        └────────┴────────┴────────┴────────┘
                       │
                       ▼
              Heuristic Selection
                       │
                       ▼
                   OpenJML
                  Verification
                       │
                  ┌────┴────┐
                  │         │
                Pass      Fail
                  │         │
                  ▼         ▼
             Final Spec   Next Candidate


# Information Extraction Table

| Category                                  | Description                                                                                                                                                                                                                                                        |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **1. Problem**                            | Automatically generate **accurate and verifiable formal specifications** for Java programs. LLM-generated specifications may contain errors and fail formal verification.                                                                                     |
| **2. Method**                             | Two-phase approach: **(1) Conversation-driven generation** using GPT-3.5 + few-shot prompting + OpenJML feedback; **(2) Mutation-based generation** after conversational refinement fails, using Predicative, Logical, Comparative, and Arithmetic mutations. |
| **3. Specification Types**                | JML specifications, mainly **preconditions, postconditions, and loop invariants**.                                                                                                                                                                            |
| **4. Preconditions**                      | Yes — JML `requires`; describes input constraints and valid program states.                                                                                                                                                                                   |
| **5. Postconditions**                     | Yes — JML `ensures`; describes output and input-output relationships.                                                                                                                                                                                         |
| **6. Frame Conditions**                   | **Not systematically addressed.** No dedicated evaluation of `assignable`/frame conditions.                                                                                                                                                                   |
| **7. Other Specification Elements**       | **Loop invariants** are evaluated, including sequential, branched, single-path, multi-path, and nested loops. Termination and memory-safety specifications are not systematically studied.                                                                    |
| **8. Programming Language**               | Java                                                                                                                                                                                                                                                          |
| **9. Specification Language / Framework** | **JML (Java Modeling Language)** with **OpenJML** for formal verification.                                                                                                                                                                                    |
| **10. Dataset / Benchmark**               | **SV-COMP Java:** 265 programs; **SpecGenBench:** 120 programs; **Defects4J:** 50 real-world Java files.                                                                                                                                                      |
| **11. Reference Specifications**          | SpecGenBench contains reference specifications; 100 were written by **3 experts**, while 20 came from an existing dataset. Defects4J has no ground-truth specifications.                                                                                      |
| **12. Natural Language Input**            | **No.** Main task is **Java Code → JML Specification**. Natural language is mainly used in prompts and verifier feedback.                                                                                                                                     |
| **13. Tools / Models**                    | **GPT-3.5-turbo-1106**, 4-shot prompting, up to 10 conversation rounds; **OpenJML** for verification; mutation operators and a heuristic selector for refinement.                                                                                             |
| **14. Evaluation and Results**            | On 385 core programs, SpecGen verified **279/385 (72.5%)** and achieved a **59.97% success probability**. On Defects4J, **38/50** were successfully handled.                                                                                                  |
| **15. Evaluation — Verification**         | Uses OpenJML. **279/385** core programs were verifiable; nested loops remained particularly challenging.                                                                                                                                                      |
| **16. Evaluation — Human Evaluation**     | 15 PhD students rated semantic quality. SpecGen: **4.54/5**, close to expert-written Oracle: **4.83/5**.                                                                                                                                                      |
| **17. Evaluation — Ablation**             | Removing mutation types reduced performance: Predicative **259**, Logical **235**, Comparative **223**, Arithmetic **262**, vs. **279** for full SpecGen.                                                                                                     |
| **18. Evaluation — Efficiency**           | Heuristic selection reduced OpenJML verification calls compared with random selection: **5.30%** reduction on SV-COMP and **21.23%** on SpecGenBench.                                                                                                         |
| **19. Baselines**                         | **Daikon, Houdini, 0-shot, 2-shot, 4-shot, Conversational, and AutoSpec.**                                                                                                                                                                                    |
| **20. Limitations**                       | Prompt sensitivity; possible GPT-3.5 data leakage; OpenJML limitations; limited representation of complex programs in SV-COMP; nested loops remain difficult; repeated verification adds computational cost.                                                  |
| **21. Missing Capabilities**              | No systematic **frame-condition**, **termination**, or **arbitrary assertion** generation; memory safety is not an independent target; only Java/JML/OpenJML is evaluated.                                                                                    |

