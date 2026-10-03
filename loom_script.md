# Loom walkthrough

About 310 spoken words, or 2-3 minutes at a natural pace. The screen cues are not spoken. The numbers below match the notebook's numbered section headings. Nothing has been recorded or published.

**0:00-0:30 | Sections 1-2: Imports and data loading, essential checks and useful charts**

Show the data-loading summary and the two charts. Briefly point to the comments above the checks.

This project predicts freight posted rates in dollars per load. I received 48,000 labeled loads from January through October, 12,000 unlabeled validation loads, and a fixed December scenario. Everything needed to run the workflow is in this one notebook.

I checked the IDs, dates, distances, and weights. The assertions stop the workflow if an important assumption fails, such as duplicated IDs or invalid distances. I kept expensive loads because a large error does not prove a bad label.

**0:30-0:55 | Section 3: Shared feature preparation and fixed model settings**

Show `prepare_features` and the fixed CatBoost settings, without explaining every line.

The same feature function handles training and prediction. It creates route and calendar features, excludes IDs and the target, and masks negative weights with a flag. Market index and quote signal are excluded because their definitions and availability are unconfirmed. Learned preprocessing uses training rows only.

**0:55-1:25 | Sections 4-5: July-August comparison and fixed model choices**

Show the four-model comparison table, then the selection explanation in section 5.

I compared mileage, Ridge, and two CatBoost models: train through June to predict July, then through July to predict August. Signal-free CatBoost had pooled mean absolute error of about 122 dollars. The December model uses only its six available inputs and scored about 127 dollars.

Weight masking gained less than a dollar, so its accuracy benefit is uncertain. I fixed these choices before holdout evaluation.

**1:25-2:00 | Section 6: September-October holdout evaluation**

Show the overall and monthly metrics, then scroll to the expensive-load scatter plot in the same section.

Training through August, I evaluated 9,523 September-October loads. Signal-free CatBoost had MAE of about 120 dollars, versus 257 for mileage. The December-feature model had about 137 dollars.

All 101 loads above the training price's 99th percentile were underestimated. October was worse than September, and the unseen-route sample was small. I did not tune against these results.

**2:00-2:35 | Sections 7-9: Final training, prediction CSVs, and the scorer chart**

Show the final training summary in section 7, the output checks in section 8, and the December chart in section 9.

Finally, I refitted the same models on all 48,000 labeled rows and generated both CSVs, matching validation IDs and preserving December inputs. The unchanged company scorer accepts them and creates this chart. It checks file validity, not accuracy. December and fixed-route forecasting accuracy remain unvalidated.

**2:35-2:50 | Section 10: Create the short assessment report**

Show the generated PDF, then finish on the README's deliverable links.

The README explains how to run everything, and the three-page report summarizes the measured results and limitations.
