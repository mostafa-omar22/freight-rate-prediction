# Freight rate prediction

Predict freight posted rates in dollars per load from physical, location, and date inputs. The entire workflow is in [freight_rate_prediction.ipynb](freight_rate_prediction.ipynb): checks and charts, model comparison, fixed selection, holdout evaluation, final training, predictions, scoring, and a three-page report.

## Setup and execution

Use Python 3.11 (verified with 3.11.5) and run from the repository root. In PowerShell:

```powershell
py -3.11 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

Keep the four original company files in `data/` with their supplied names:

- `train-test.csv`
- `validation.csv`
- `validation-predictions-template.csv`
- `december-chart-inputs.csv`

They are deliberately excluded from Git. The local copy includes them, a new checkout needs the company-provided files.

Open the notebook in VS Code or Jupyter, select this environment's Python kernel, and **Run All**. Alternatively, execute and save it from PowerShell:

```powershell
@'
import nbformat
from nbclient import NotebookClient

path = "freight_rate_prediction.ipynb"
notebook = nbformat.read(path, as_version=4)
NotebookClient(notebook, timeout=600, kernel_name="python3").execute()
nbformat.write(notebook, path)
'@ | .\.venv\Scripts\python.exe -
```

Allow a few minutes on CPU. The notebook refits models in memory and regenerates both predictions, the scorer chart, and the report. It needs no helper package, configuration files, or saved models. It does not submit anything.

## Main results

January-June predicts July, January-July predicts August. Pooled July-August MAE is $250.94 for mileage, $212.27 for Ridge, $122.39 for signal-free CatBoost, and $127.37 for December CatBoost. Model choices and settings were fixed using those comparisons and the earlier error review.

The September-October holdout uses January-August training: **38,477 training / 9,523 evaluation rows**, identical across models. No holdout tuning or model switching is performed.

| Holdout model | MAE ($/load) | RMSE ($/load) | R² |
| --- | ---: | ---: | ---: |
| Signal-free CatBoost | 120.04 | 638.51 | 0.82494 |
| December CatBoost | 136.65 | 642.78 | 0.82259 |
| Mileage baseline | 256.95 | 684.25 | 0.79896 |

The fixed CatBoost settings include 450 trees, depth 6, learning rate 0.08, MAE loss, seed 2025, and two CPU threads. Validation uses physical/date inputs plus four coordinates, December uses only its six available inputs. Both mask negative weights and add an indicator. The historical Ridge benchmark retains negative weights to reproduce its recorded comparison. IDs, target, `market_index`, and `quote_signal` are excluded from predictors.

After evaluation, both selected models are refitted on all **48,000** labeled January-October rows. These final fits generate submissions, the holdout scores describe the earlier fits.

## Deliverables

- [Executed notebook](freight_rate_prediction.ipynb)
- [Validation predictions](validation_predictions.csv): 12,000 unique IDs in template order
- [December predictions](december_predictions.csv): 31 dates, supplied physical inputs unchanged
- [Original scorer chart](candidate_december.png)
- [Three-page assessment report](assessment_report.pdf)
- [Loom speaking notes](loom_script.md): 2-3 minute walkthrough, no recording published

The unchanged [score.py](score.py) accepts both files. To run it independently:

```powershell
.\.venv\Scripts\python.exe score.py --predictions validation_predictions.csv --december-predictions december_predictions.csv --output-dir .
```

## Limitations and verification

Rare expensive loads are underestimated: all 101 holdout loads above training price P99 are underpredicted and account for about 86% of squared error. October errors and downward bias are higher. Only 21 holdout loads have unseen routes and 59 have negative weights, future validation has 12.175% development-unseen routes. Negative-weight masking produced less than $1 pooled MAE improvement, so its accuracy benefit is uncertain.

Signal definitions and prediction-time availability are unresolved. One year and calendar trees provide limited future extrapolation. Reduced-feature holdout performance does not validate December seasonality or the specific fixed route. Earlier descriptive exploration saw all development labels, the holdout was unused for selection rather than completely analyst-unseen. The scorer checks validity, not hidden-label accuracy.

The consolidated notebook was executed top to bottom and checked against the recorded evaluation results and preserved submissions. Original inputs and scorer are unchanged. Earlier experiments and results were preserved in a local backup outside this repository before cleanup.
