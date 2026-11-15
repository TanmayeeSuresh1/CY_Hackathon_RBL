---
quick_task: 261115-uax
description: Create a feature branch and add an ML evaluation plots notebook
status: planned
autonomous: true
files_modified:
  - ml/plots.ipynb
---

<objective>
Create a feature branch off the current checkout and add `ml/plots.ipynb`, a self-contained notebook that visualizes the repository's existing ML evaluation artifacts without retraining or changing model code.

Purpose: Make the shipped evaluation results inspectable through reproducible confusion-matrix, per-class metric, and training-history visualizations while preserving the repository's existing PyTorch/scikit-learn artifact conventions.

Output: Branch `feature/261115-uax-ml-plots` (or reuse it if it already exists and is not `main`) and a valid, runnable `ml/plots.ipynb`.
</objective>

<context>
@.planning/STATE.md
@.planning/PROJECT.md
@requirements.txt
@ml/evaluate.py
@ml/train.py
@ml/artifacts/evaluation_report.json
@ml/artifacts/training_history.json

The project state has no active phase and all milestones are complete, so this is an isolated quick task. The evaluation report stores class labels, a row-true/column-predicted confusion matrix, and per-class precision/recall/F1. The training history stores epoch-indexed train/validation loss and accuracy arrays. Use existing installed dependencies only; do not add packages or regenerate artifacts.
</context>

<tasks>

<task type="auto">
  <name>Task 1: Create and verify the feature branch</name>
  <files>git branch metadata only</files>
  <action>From the repository root, inspect the current branch and working tree before changing anything. If `feature/261115-uax-ml-plots` already exists, check it out; otherwise create it from the current commit with `git switch -c feature/261115-uax-ml-plots`. Do not work on or create `main`, and do not reset, stash, or discard unrelated user changes. Confirm the resulting branch is the requested feature branch before editing files.</action>
  <verify>
    <automated>git branch --show-current; git status --short</automated>
  </verify>
  <done>The checkout is on `feature/261115-uax-ml-plots`, with pre-existing unrelated changes preserved.</done>
</task>

<task type="auto">
  <name>Task 2: Build the evaluation visualizations notebook</name>
  <files>ml/plots.ipynb</files>
  <action>Create `ml/plots.ipynb` using standard notebook JSON and the repository's existing artifact formats. Add markdown identifying the source files and metric orientation, then code cells that load `ml/artifacts/evaluation_report.json` and `ml/artifacts/training_history.json` via paths that work from the repository root and from the `ml` directory. Render: (1) a labeled confusion-matrix heatmap using `confusion_matrix.matrix` and `confusion_matrix.labels`, with rows documented as true labels and columns as predicted labels; (2) grouped per-class precision, recall, and F1 bars from `per_class_metrics`, preserving the report's class order and displaying values; and (3) two training-history figures, one for train/validation accuracy and one for train/validation loss, using the epoch count and the four corresponding arrays. Include concise titles, axis labels, legends, and grids; use only dependencies already present/available in the repository such as JSON/pathlib, NumPy, pandas if useful, and matplotlib. Keep the notebook read-only with respect to artifacts, avoid retraining/model imports, and do not hardcode the reported metric values into plotting data.</action>
  <verify>
    <automated>python -c "import json; from pathlib import Path; p=Path('ml/plots.ipynb'); n=json.loads(p.read_text(encoding='utf-8')); assert n.get('nbformat', 0) >= 4; assert n.get('cells'); src='\\n'.join(''.join(c.get('source', [])) for c in n['cells']); assert 'evaluation_report.json' in src and 'training_history.json' in src and 'confusion' in src.lower() and 'precision' in src.lower() and 'recall' in src.lower() and 'f1' in src.lower() and 'train_acc' in src and 'val_acc' in src and 'train_loss' in src and 'val_loss' in src; print('notebook structure and required data references valid')"</automated>
  </verify>
  <done>`ml/plots.ipynb` contains executable cells for all four requested visual outputs, reads the existing JSON structures dynamically, and has no model-training or artifact-writing behavior.</done>
</task>

<task type="auto">
  <name>Task 3: Execute and validate the notebook as far as the environment permits</name>
  <files>ml/plots.ipynb</files>
  <action>Validate the notebook JSON and execute it from the repository root using the available notebook runner (`jupyter nbconvert --execute --to notebook --stdout` or an equivalent installed runner), without overwriting the source notebook. Confirm the source JSON remains valid after execution, all referenced artifact files exist, and the notebook completes without errors when plotting dependencies are installed. If the environment lacks a notebook runner or plotting backend, record that limitation in the execution result rather than adding dependencies or weakening the notebook's artifact-driven implementation. Check the final diff to ensure only `ml/plots.ipynb` was added beyond branch metadata.</action>
  <verify>
    <automated>python -c "from pathlib import Path; import json; json.loads(Path('ml/plots.ipynb').read_text(encoding='utf-8')); assert Path('ml/artifacts/evaluation_report.json').is_file(); assert Path('ml/artifacts/training_history.json').is_file(); print('JSON and source artifacts present')" ; jupyter nbconvert --execute --to notebook --stdout ml/plots.ipynb *> $null; if ($LASTEXITCODE -ne 0) { python -c "import importlib.util; print('Notebook execution unavailable or failed; inspect dependency/error output before accepting')" }</automated>
  </verify>
  <done>Notebook JSON parses, source artifacts are present, execution succeeds when the existing environment supports it, and any environment limitation is explicitly reported without unrelated repository changes.</done>
</task>

</tasks>

<must_haves>
  <truths>
    <truth>The work is performed on `feature/261115-uax-ml-plots`, not `main`.</truth>
    <truth>A user can open `ml/plots.ipynb` and see confusion-matrix, per-class precision/recall/F1, accuracy, and loss visualizations generated from the checked-in JSON artifacts.</truth>
    <truth>The notebook remains reproducible when metric values or artifact contents change because plotting data is loaded dynamically.</truth>
  </truths>
  <artifacts>
    <artifact>ml/plots.ipynb</artifact>
    <artifact>ml/artifacts/evaluation_report.json</artifact>
    <artifact>ml/artifacts/training_history.json</artifact>
  </artifacts>
  <key_links>
    <key_link>Notebook JSON cells load evaluation_report.json confusion_matrix and per_class_metrics fields into the requested plots.</key_link>
    <key_link>Notebook JSON cells load training_history.json train_acc, val_acc, train_loss, and val_loss arrays against epoch indices.</key_link>
  </key_links>
</must_haves>

<verification>
Run the structural JSON check, then execute the notebook from the repository root with the available Jupyter runner. Inspect the resulting output/error status and `git diff --stat`/`git status --short` to confirm the surgical scope.
</verification>

<success_criteria>
- The current checkout is a non-main feature branch named `feature/261115-uax-ml-plots`.
- `ml/plots.ipynb` is valid nbformat JSON and references both existing metric artifacts.
- All requested visualization categories are implemented from loaded artifact data.
- Notebook execution succeeds where the installed environment supports Jupyter and matplotlib; otherwise the precise missing capability is reported.
- No dependencies, model files, evaluation reports, training history, or unrelated source files are changed.
</success_criteria>

<output>
After execution, create `.planning/quick/261115-uax-create-a-feature-branch-and-add-ml-plots/261115-uax-SUMMARY.md` with the branch name, notebook validation command/results, and any environment limitation.
</output>
