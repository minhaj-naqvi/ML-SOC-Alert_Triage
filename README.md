# A Reproducible Framework for Machine Learning-Assisted Alert Triage in Security Operations Centers

This repository contains the datasets, source code, trained models,
configuration examples, and deployment components associated with the
study “A Reproducible Framework for Machine Learning-Assisted Alert
Triage in Security Operations Centers.”

The proposed framework separates telemetry acquisition, behavioral normalization, data quality
assurance, feature engineering, model training, decision optimization,
operational deployment, and continuous improvement into distinct stages.

The framework itself is intended to be SIEM- and model-agnostic. The
proof-of-concept implementation provided here uses Wazuh, Windows Sysmon
telemetry, and Privilege Escalation activity.

## Overview

The repository demonstrates how to:

-   Build a unified dataset from locally generated Wazuh Sysmon
    telemetry and public Mordor Sysmon datasets.
-   Normalize heterogeneous telemetry into a common behavioral
    representation.
-   Reduce environment-specific/domain bias by masking identifiers and
    other environment-dependent information.
-   Construct malicious labels from controlled Atomic Red Team (ART)
    executions.
-   Refine weakly supervised attack-window labels through manual
    behavioral review and process-lineage analysis.
-   Train and evaluate Logistic Regression, Random Forest, and Extreme
    Gradient Boosting (XGBoost) using a common preprocessing and
    feature-engineering pipeline.
-   Compare model behavior using precision, recall, F1-score, PR-AUC,
    ROC-AUC, false positives, and false negatives.
-   Analyze decision thresholds to study the operational trade-off
    between false positives and false negatives.
-   Deploy the selected model as an independent ML triage layer without
    modifying the underlying Wazuh detection workflow.
-   Enrich Wazuh alerts with ML outputs and visualize them through
    OpenSearch/Wazuh Dashboard.

## Proposed Framework

The proposed framework consists of eight functional stages:

1.  Telemetry Acquisition — Collect security telemetry from one or more
    monitoring sources.
2.  Behavioral Normalization — Transform heterogeneous telemetry into a
    common behavioral representation.
3.  Data Quality Assurance — Address domain leakage, label quality,
    consistency, deduplication, and related data-quality concerns before
    training.
4.  Feature Engineering — Convert normalized events into ML-compatible
    representations. The proof-of-concept combines TF-IDF textual
    features with contextual numerical attributes.
5.  Model Layer — Support different supervised learning algorithms
    within the same upstream and downstream architecture.
6.  Decision Optimization — Analyze prediction scores and thresholds
    according to operational requirements such as malicious recall,
    false-positive reduction, and analyst workload.
7.  Operational Deployment — Integrate model predictions into an
    existing SOC workflow as an independent triage/enrichment layer.
8.  Continuous Improvement — Incorporate analyst feedback, labeling
    corrections, emerging attack behavior, and operational requirements
    into subsequent iterations.

## Proof-of-Concept Instantiation

The implementation evaluated in the accompanying study uses:

-   SIEM: Wazuh
-   Endpoint telemetry: Windows Sysmon
-   Attack scope: Privilege Escalation
-   Controlled attack execution: Atomic Red Team (ART)
-   Additional public telemetry: Mordor
-   Models: Logistic Regression, Random Forest, and XGBoost
-   Text representation: TF-IDF using unigram and bigram features
-   Operational integration: Python-based alert enrichment with
    OpenSearch/Wazuh Dashboard visualization

The present proof-of-concept intentionally focuses on a single telemetry
source and ATT&CK tactic. These choices define the scope of the
evaluated instantiation rather than the intended scope of the framework.

## Data Quality and Label Refinement

Initial Wazuh malicious labels were generated using time windows
surrounding controlled ART executions. Because attack-window labeling
can also capture unrelated background activity, the Wazuh events
initially labeled as malicious were subsequently reviewed using event
behavior and process-lineage information.

Events determined to be unrelated to the controlled attack execution
were relabeled before retraining and reevaluating the models. The
repository therefore retains artifacts associated with the original
dataset as well as the post-refinement dataset used for the updated
experiments.

For reproducing the final manuscript results, use the post-refinement
dataset and corresponding post-refinement model material.

## Models

Three supervised learning algorithms are included to demonstrate the
model-agnostic nature of the framework:

-   Logistic Regression
-   Random Forest
-   XGBoost

The src/ directory contains the training/evaluation material associated
with the experiments, including the post-refinement versions for all
three models.

The reported comparison uses a common feature representation, dataset
partitioning approach, and surrounding preprocessing pipeline so that
the models are evaluated under consistent upstream and downstream
conditions.

XGBoost was selected for the proof-of-concept operational deployment
because it provided a favorable balance between malicious-event recall
and false-positive reduction. Decision thresholds can subsequently be
adjusted according to organizational risk tolerance and operational
requirements.

## Repository Structure

    src/                # Data preparation, preprocessing, training, evaluation,
                        # Wazuh integration code
                        # Includes post-refinement LR, RF, and XGBoost model work

    data/               # Source/processed data used by the experiments
                        # Includes the post-refinement dataset

    artifacts/           # Serialized/trained model

    configs/             # Wazuh, Filebeat, and OpenSearch configuration examples

As mentioned before, post-refinement files should be used when
reproducing the final reported results.

## Operational Integration with Wazuh

The framework does not replace Wazuh’s rule-based detection engine.
Instead, ML operates as an independent triage layer on top of the
existing alert workflow.

The src/integration/ml_triage.py service can be deployed alongside the
Wazuh environment to:

1.  Process newly generated Wazuh alerts.
2.  Apply the same preprocessing and feature representation used during
    model development.
3.  Generate model prediction scores.
4.  Enrich the original alert with ML output.
5.  Forward enriched alerts for indexing in OpenSearch.
6.  Present both the original alert information and ML assessment
    through the existing dashboard workflow.

Example Filebeat, Wazuh, and OpenSearch configuration material is
provided under configs/.

## Reproducing the study

A complete reproduction should follow the same logical sequence as the
proposed framework:

    Telemetry Acquisition
            |
            v
    Behavioral Normalization
            |
            v
    Data Quality Assurance
            |
            v
    Feature Engineering
            |
            v
    Model Training / Evaluation
            |
            v
    Decision-Threshold Analysis
            |
            v
    Operational Deployment

For reproducing the final manuscript results, use the post-refinement
dataset and the corresponding post-refinement Logistic Regression,
Random Forest, and XGBoost training/evaluation material.

The original/pre-refinement artifacts are retained where useful for
demonstrating the effect of label refinement and tracing the evolution
of the experiments.

## Experimental Results

Following label refinement, the three evaluated models produced the
following test-set results:


| Model | Accuracy | Malicious Precision | Malicious Recall | Malicious F1 | PR-AUC | ROC-AUC | FP | FN |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| Logistic Regression | 0.968 | 0.823 | 1.000 | 0.903 | 0.988 | 0.998 | 55 | 0 |
| Random Forest | 0.990 | 0.947 | 0.988 | 0.967 | 0.994 | 0.999 | 14 | 3 |
| XGBoost | 0.994 | 0.977 | 0.984 | 0.980 | 0.998 | 0.9997 | 6 | 4 |



At the reported XGBoost operating point, 257 of the 1,715 test-set
alerts were surfaced for investigation, corresponding to an 85.01%
reduction in alert volume, while 4 malicious samples (approximately
1.57% of the malicious test samples) were missed.

These values correspond to the refined dataset and should not be assumed
to apply to other telemetry sources, ATT&CK tactics, SOC environments,
or deployment conditions.

## Reproducibility Notes

Reproducibility depends on preserving both the framework stages and the
configuration of the evaluated instantiation. When reproducing or
extending this work, record or retain:

-   Dataset version and label-refinement status
-   Train/test partitioning and random seeds
-   Preprocessing and masking rules
-   TF-IDF configuration
-   Model parameters
-   Decision threshold
-   Wazuh/Sysmon configuration
-   Software/library versions
-   Deployment and indexing configuration

Exact event-level reproduction of the locally generated dataset is not guaranteed, as telemetry may vary across repeated executions due to differences in system state and background activity. The repository therefore aims to reproduce the methodology, configuration, processing stages, and experimental procedure rather than guarantee an identical sequence of security events. 

Additionally, Manual label refinement involves analyst judgment and may therefore introduce inter-reviewer variability; the refinement criteria and procedure are documented to support transparency and repeatability.

The repository is intended to make these artifacts available wherever
licensing and redistribution terms permit.

## Extending the Framework

The framework is not restricted to Wazuh, Sysmon, Privilege Escalation,
or XGBoost. Future or independent instantiations may substitute:

-   Other SIEM platforms
-   Network or additional host telemetry
-   Additional ATT&CK tactics
-   Alternative feature representations
-   Other supervised learning models
-   Different decision policies and thresholds
-   Analyst-feedback and model-update mechanisms

Results from such extensions should be evaluated independently rather
than assumed to reproduce the performance of the proof-of-concept
implementation.

## License

This project is released under the repository’s open-source license.
Third-party datasets, software, rules, and other external artifacts
remain subject to their respective licenses and terms.

Mordor-derived material follows the licensing terms of the original
Mordor project. Atomic Red Team, Wazuh, SOCFORTRESS, and other
third-party components remain subject to their respective licenses.

## Citation

If you use this repository, framework, dataset-processing methodology,
or implementation in academic work, please cite the accompanying study.

Citation information will be updated when the manuscript is published.

    @article{imam_ml_alert_triage,
      title  = {A Reproducible Framework for Machine Learning-Assisted Alert Triage in Security Operations Centers},
      author = {Imam, Muhammad and Binbeshr, Farid},
      note   = {Manuscript under review / publication details to be added}
    }
