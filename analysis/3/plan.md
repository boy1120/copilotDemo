# Plan: Grounding Radiology Report Findings into Medical Image Segmentation (CF2Seg)

- Issue: https://github.com/boy1120/copilotDemo/issues/3
- Paper: `recommended/2026-08-20/cf2seg-report-guided-cxr-segmentation/paper.json`
- DOI: https://doi.org/10.1038/s41746-026-03051-0
- Physician's focus: 결절(nodule) 증례가 적은 코호트에서도 보고서 텍스트만으로 병변 위치를
  학습하는 CF2Seg 접근이 통할지, 특히 놓친 결절(거짓음성, false negative)이 걱정된다는 점.

## Proposed presentation

Dashboard entry to build for this paper, tailored to the false-negative/rare-nodule concern
rather than a generic "paper vs. our data" summary:

1. **Bar chart — finding-label frequency, nodule highlighted** (x: label, y: study count).
   A simple bar chart (not pie) is right here because the labels are not mutually exclusive
   (multi-label) and the physician needs to see how small the nodule count (12 studies with
   "Nodule" anywhere in a multi-label set, 7 as the sole label) is *relative to* the far larger
   "No Finding" (145) and "Infiltration" (21) groups — bars make that size gap and the class
   imbalance instantly legible, which is the crux of the false-negative worry (rare class →
   the model sees few nodule examples → the "learn location from text only" assumption is
   most fragile exactly here).
2. **Table — paper cohort vs. our cohort** (see Differences section below, reused verbatim in
   the dashboard) so the physician can see side-by-side why the paper's 53,386-exam, multi-source,
   pixel-annotated benchmark is a much stronger regime for learning localization than our 12-nodule,
   annotation-free cohort.
3. **Mermaid flow diagram** of the *proposed* (not yet run) analysis: report_text + CXR →
   CF2Seg-style text-guided localization → compare candidate localization against
   `findings_label`/`report_text` mentions of nodule location, stratified by nodule vs.
   no-nodule studies → manual radiologist spot-check of nodule cases only, because with 12
   cases an automated sensitivity/false-negative number would carry huge sampling error and
   must be framed as illustrative, not a validated FN rate.
4. **Text sections**: (a) one-paragraph restatement of the physician's question, (b) what the
   paper reports about annotation scarcity/robustness (their own low-annotation ablations),
   (c) explicit statement that we cannot compute a true false-negative rate without expert
   spatial ground truth, only a qualitative concordance check.
5. **Layout order**: question → label-frequency bar chart (shows why nodules are the hard case)
   → paper-vs-ours table → proposed flow diagram → caveats/open questions. This puts the
   "why this is risky" evidence before the "here's what we'd try" plan, matching how a physician
   would want to weigh feasibility before agreeing to a deep dive.

## Open questions

- Is a **qualitative** radiologist review of the 12 nodule-containing studies (checking whether
  CF2Seg-style localization lands in a plausible lung region from report text) an acceptable
  substitute for a quantitative false-negative rate, given we have no pixel-level ground truth?
- Should "nodule subgroup" include only the 7 studies where Nodule is the sole label, or all 12
  studies where Nodule co-occurs with other findings (e.g., Effusion+Nodule, Fibrosis+Nodule)?
  This changes both the denominator and the difficulty of the localization task.
- Do we have access to a pretrained CF2Seg checkpoint, or would a deep dive be limited to a
  much simpler heuristic (e.g., text-keyword-to-lung-zone mapping) as a stand-in — and is that
  still worth building given the paper's method is not runnable ready-made?
- Is INST0x institution_code stratification of interest, or should the deep dive treat the
  cohort as single-institution for this question (the paper's institution-generalization claims
  cannot be tested with our single-institution data)?
