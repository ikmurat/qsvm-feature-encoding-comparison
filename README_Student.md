# Student Performance: QSVM Feature-Map Comparison

This README documents the supplied Student Performance experiment script.

## Dataset and source

[CSV copy on GitHub](https://github.com/notjustavibe/Student-CGPA-Predictor/blob/main/Student_performance_data%20__.csv); the repository identifies [the Kaggle source](https://www.kaggle.com/datasets/rabieelkharoua/students-performance-dataset). The repository describes the data as synthetic; verify exact provenance and license.

Expected input: `student.csv`. Output folder: `Student_GPA_SVM_QSVM_outputs`.

The code defines `Success_GPA = 1` when GPA > 2.5, otherwise 0; missing GPA rows are removed. `StudyTimeWeekly` is binarized at >= 10. The nine predictors are Age, Gender, ParentalEducation, StudyTimeWeekly, Absences, Tutoring, ParentalSupport, Sports and Music. GPA and GradeClass are excluded from the inputs.

## Models and experimental settings

The script compares 11 quantum feature maps (Angle-RY, H+RY, five dependency-weighted RY+CRY maps, Z, and ZZ with linear, circular and full connectivity) with a classical RBF-SVM. Dependency weights are computed from training data. For each pair i < j, the controlled rotation angle is `D[i,j] * (alpha[i] + alpha[j]) / 2`; i is the control. Pearson, Spearman and Kendall weights use absolute correlations. MI is symmetrized and normalized; distance correlation is nonnegative.

When the input has more than 500 eligible rows, the code selects 500 using stratified random sampling, then uses a stratified 75/25 train/test split; seed: 42. Group-aware mode is available but is disabled by default. Training-fitted preprocessing is applied to test data. Numerical missing values are median-imputed; categorical columns, when detected, use training-fitted most-frequent imputation and ordinal encoding. Inputs are scaled to [0, pi] and clipped. The student study-time feature has a special mode-based missing-value treatment.

All SVMs use C=1 and no class weighting. The RBF model uses `gamma="scale"`; the value is determined from its training input. Quantum SVMs use precomputed noiseless statevector fidelity kernels. Z/ZZ use one repetition. There is no hyperparameter tuning or separately tuned quantum input scale. Probabilities are produced by scikit-learn SVC with `probability=True` and `predict_proba`; AUC and log-loss use positive-class probabilities, while predictions for accuracy and F1 come from `predict`. In binary SVC, internal cross-validation is used for Platt probability calibration. Record the library versions used for the original experiment before publishing reproducibility claims.

## Installation and execution

Requires Python, NumPy, pandas, SciPy, scikit-learn, Matplotlib, Qiskit and Jinja2. In a fresh environment:

```bash
python -m pip install numpy pandas scipy scikit-learn matplotlib qiskit jinja2
```

Save the corresponding supplied experiment script as `experiment.py`, place the expected CSV alongside it, check `FILE_PATH`, `TARGET_COLUMN` and `POSITIVE_LABEL` in the script, then run:

```bash
python experiment.py
```

These README files do not contain the scripts or data. The default package install does not pin versions; for reproducible results, publish the actual environment and versions used in the reported run. Qiskit API compatibility and statevector memory requirements depend on the environment and feature count. The 14-qubit statevector runs can be costly.

## Outputs and reproducibility

The script writes outputs to the directory named above, including `configuration.json`, `split_manifest.csv`, `class_distributions.csv`, `performance.csv`, model prediction files, confusion matrices, dependency matrices, confidence intervals and paired comparison tables. The manifest records selected source rows, splits and labels; share the original input version, row ordering, manifest and software versions for reproducibility. Verify the script version actually used to produce the manuscript before releasing. Source row indices refer to the script's processed data and may not match original file line numbers if rows were removed or reindexed.

The default inference uses a stratified paired percentile bootstrap of test rows (10,000 iterations), exact McNemar tests for accuracy, DeLong tests for AUC, and paired prediction-swap permutation tests for positive-class F1 (20,000 permutations). Holm adjustments are reported by metric and jointly. Intervals condition on the fitted model and selected split; they do not include retraining or split variability. Noiseless simulation does not demonstrate quantum advantage. The `main` branch of an external dataset can change; archive the exact version or hash when permitted, and verify its license before redistributing.
