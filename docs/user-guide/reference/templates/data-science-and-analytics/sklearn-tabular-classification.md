# scikit-learn Tabular Classifier

![scikit-learn Tabular Classifier Preview](/images/sklearn-notebook/sklearn-interactive-dashboard-hero.png "doc-image")

## Overview

Gradient Boosting Machine (GBM) is one of the most powerful and popular machine learning algorithms for tabular data (data organized in rows and columns). It is an ensemble method, which means it combines multiple individual models (typically decision trees) to make more accurate predictions than any single model could achieve on its own. GBM works by sequentially building trees, where each new tree tries to correct the errors made by the previous ones. This iterative process makes it particularly effective for complex datasets with non-linear relationships.

In this template, we utilize:
*   **Wine Quality Dataset:** A classic dataset containing chemical properties of wines.
*   **Gradient Boosting:** A powerful ensemble method for high-performance classification.
*   **Feature Interpretability:** Visualizing which factors (like alcohol or acidity) most impact wine quality.

## Why use this template?

Use this template as a production-ready foundation for any tabular classification task. It implements best practices for data splitting, model training, and performance metrics. Whether you are analyzing logistics, customer churn, or environmental data, this scikit-learn and Pandas foundation is designed to scale with your needs.

## Getting Started

1. Simply log in to Saturn Cloud.
2. Click on the **dashboard** in the top-left sidebar.
3. Scroll down to the **Templates & Starter Kits** section
4. Select the **Data Science** category
5. Click on **scikit-learn tabular classification**.

![Select scikit-learn Notebook from Dashboard](/images/sklearn-notebook/sklearn-dashboard-selection.png "doc-image")

6. In the **Create Resource from Template** modal, click **Submit**.

![Submit scikit-learn Template Resource](/images/sklearn-notebook/sklearn-template-submit.png "doc-image")

7. After clicking on the **Submit** button, you will be taken to the **Workspace** page where all the configuration about the resource is located. From there, you click on the **start** button to start the template.

![Start Template Resource](/images/sklearn-notebook/sklearn-start-template.png "doc-image")

8. The resource takes a few minutes to start and provision all the **necessary software**. The software **bar** will gradually fill up at the bottom of the resource information box. 

![Resource Provisioning](/images/sklearn-notebook/sklearn-resource-provisioning.png "doc-image")

9. Once the template resource is running, you can **click** on the **JupyterLab** tab to open the notebook and begin your analysis.

![Open JupyterLab](/images/sklearn-notebook/sklearn-open-jupyter.png "doc-image")

*Note: A new tab will open with the notebook, as seen in the image below.*

![JupyterLab Interface](/images/sklearn-notebook/sklearn-jupyterlab-interface.png "doc-image")

10. In JupyterLab, double-click the notebook file in the left sidebar to open it. Then, press **Run All** in the top menu to execute the code and train your classifier.

![Run All Notebook Cells](/images/sklearn-notebook/sklearn-run-all-cells.png "doc-image")

11. Once the execution completes, the feature importance plot and classification report will render directly in the notebook.

![Final scikit-learn Output](/images/sklearn-notebook/sklearn-final-output.png "doc-image")

## What to Expect in the Notebook

The included notebook is self-contained and guides you through a complete machine learning workflow:

1.  **Dependency Management:** Installing core libraries (`scikit-learn`, `pandas`, `matplotlib`, `seaborn`) into your active environment.
2.  **Data Loading & Preprocessing:** Loading the Wine Quality dataset and performing a Train/Test split to ensure model generalization.
3.  **Gradient Boosting Training:** Initializing and fitting a `GradientBoostingClassifier` with optimized hyperparameters.
4.  **Feature Importance Analysis:** Generating a horizontal bar chart to visualize the top variables driving the model's predictions.

## Troubleshooting & Best Practices

*   **Imbalanced Data:** If your target classes are not evenly distributed, consider using `class_weight='balanced'` or stratified splitting in `train_test_split`.
*   **Memory Management:** For extremely large tabular datasets (multi-gigabyte), consider using **Dask** or increasing the RAM on your Saturn Cloud resource.
*   **Hyperparameter Tuning:** This template uses sensible defaults. For higher accuracy, you can implement `GridSearchCV` or `RandomizedSearchCV` from `sklearn.model_selection`.

## Key Features

*   **End-to-End Pipeline:** Covers everything from raw data ingestion to final model evaluation.
*   **Interpretability Focus:** Built-in feature importance plotting to explain model decisions.
*   **Optimized Performance:** Leverages Gradient Boosting for state-of-the-art results on tabular data.
*   **Zero Setup Hassle:** Pre-configured to run on standard CPU resources within Saturn Cloud.

## Conclusion

This template demonstrates how to quickly build and interpret a high-performance classification model using scikit-learn. You can easily swap the Wine Quality dataset for your own business data and begin extracting insights immediately within Saturn Cloud.

**Resources and Support:**
*   **Platform:** [Saturn Cloud Dashboard](https://app.saturncloud.io/)
*   **Support:** [Saturn Cloud Documentation](/docs/)
*   **Library:** [scikit-learn Documentation](https://scikit-learn.org/)
*   **Library:** [Pandas Documentation](https://pandas.pydata.org/)
