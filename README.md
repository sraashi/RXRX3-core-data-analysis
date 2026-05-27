Perturbation Activity Scoring in RxRx3-core (Cell Painting)

Question: Can morphological embeddings from Cell Painting images recover biologically active perturbations, and do they place known positive controls where we'd expect relative to negative controls?

Approach: Using the public RxRx3-core dataset (222,601 wells; CRISPR knockouts and compound treatments) and the OpenPhenom morphological embeddings (384-dim), I score each perturbation by its within-plate cosine distance from the negative-control centroid — a plate-normalized measure of how strongly a perturbation shifts cell morphology away from baseline.

Key result: The pipeline passes its built-in control sanity check — negative controls cluster near zero (mean cosine distance ≈ 0.006) while positive controls PLK1/MTOR separate cleanly (mean ≈ 0.061), comfortably above a 2-SD activity threshold (≈ 0.026). After selecting the strongest guide per gene, ~51% of perturbations score as active, and the top hits are dominated by known bioactive compounds (proscillaridin A, digitoxin, bortezomib) and essential-gene knockouts (SRP54, COPG1, MDM2) — exactly the profile you'd expect if the embeddings carry real biological signal.


What the notebook does:

Load & merge — pulls RxRx3-core metadata and OpenPhenom embeddings directly from the HuggingFace Hub and joins them on well_id (384 morphological features per well).
Define well roles — labels negative controls (EMPTY_control), positive controls (PLK1/MTOR CRISPR knockouts), and excluded wells (CRISPR_control).
Within-plate normalization — for each of 1,744 plates, computes the negative-control centroid and measures every well's cosine distance from it, controlling for plate-to-plate batch effects.
Control QC — verifies negative controls sit near zero and positive controls clear a mean + 2·SD activity threshold before trusting any downstream hit.
Guide selection — for CRISPR perturbations, retains the single strongest-signal guide per gene to reduce guide-efficiency noise.
Hit ranking — ranks perturbations by mean distance with replicate counts, and visualizes distance distributions (negative vs. positive vs. test) for QC.


Why within-plate normalization matters:

High-content screening data can be dominated by plate- and batch-level technical variation. Measuring each well against its own plate's control centroid — rather than a global baseline — is what makes the positive/negative separation hold up. 
The control sanity check is deliberately placed before hit-calling: if PLK1/MTOR didn't clear threshold, the labels or pipeline would be wrong and no downstream result should be trusted.


Data & reproducibility:

Data: RxRx3-core (public, Recursion Pharmaceuticals) and OpenPhenom embeddings, both downloaded at runtime via huggingface_hub — no data is committed to this repo.
Stack: pandas, numpy, scikit-learn (cosine_distances), matplotlib, seaborn.
Run: open phenocopy_activity_scoring.ipynb (formerly RXRX3-core-3-2.ipynb) and run top to bottom; embeddings download automatically on first run.

Scope & honest limitations:

This implements perturbation activity scoring (distance from control), not full compound→gene mechanism-of-action matching. Cross-modality cosine similarity between compounds and CRISPR knockouts is the natural next step and is partially scaffolded in the notebook.
Activity is scored on the strongest guide per gene; a multi-guide consensus would be more robust.
No multiple-testing correction is applied to the activity threshold — it's a screening-grade filter, not a statistical claim about individual hits.


Next steps:


Complete compound→CRISPR cosine-similarity matching to surface candidate mechanism-of-action relationships.
Add a multi-guide consensus score and bootstrap confidence intervals on hit ranks.
