# Datalad Diabetes Project

This README provides a step-by-step guide for setting up and running a Datalad project focused on a diabetes dataset. You'll learn how to manage data and code using Datalad, run scripts, and track changes in a reproducible manner.

## Prerequisites

- Install [Datalad](https://www.datalad.org/) and ensure it's available in your PATH.
- Have Python installed to run the scripts.
- Install the `tree` command for directory visualization (optional).

Verify your Datalad installation:

```bash
datalad
```

Check Datalad version:

```bash
datalad --version
```

## Setting Up the Project

Create a new directory for the project and initialize it with Datalad using the YODA (YODA's Organized Data Approach) data management practices:

```bash
mkdir diabetes
cd diabetes
datalad create -c yoda .
```

View the directory structure:

```bash
tree
```

## Initializing the Project Structure

Create the necessary directories:

```bash
mkdir -p code data/raw data/processed/train data/processed/test results params
```

Configure Git attributes to track certain files with Git instead of Git-annex:

```bash
echo 'results/**/* annex.largefiles=nothing' >> .gitattributes
echo 'params/**/* annex.largefiles=nothing' >> .gitattributes
```

Save the changes to `.gitattributes`:

```bash
datalad save -m "Initialize project structure and edit .gitattributes" .gitattributes
```

View the updated directory structure:

```bash
tree
```

## Downloading Code Files

Download the necessary Python scripts into the `code` directory:

```bash
datalad download-url -O code/ https://raw.githubusercontent.com/bhanuprasanna2001/datalad-demo/master/code/download_data.py -m "Downloaded download_data.py"
datalad download-url -O code/ https://raw.githubusercontent.com/bhanuprasanna2001/datalad-demo/master/code/evaluate.py -m "Downloaded evaluate.py"
datalad download-url -O code/ https://raw.githubusercontent.com/bhanuprasanna2001/datalad-demo/master/code/process_data.py -m "Downloaded process_data.py"
datalad download-url -O code/ https://raw.githubusercontent.com/bhanuprasanna2001/datalad-demo/master/code/train.py -m "Downloaded train.py"
```

View the directory structure with the downloaded code:

```bash
tree
```

Check the Git commit log:

```bash
git log
```

## Adding Parameter Configurations

Create a `config.json` file with model parameters:

```bash
echo '{
    "model": "DecisionTree",
    "parameters": {
        "criterion": "gini",
        "splitter": "best",
        "max_depth": 7,
        "min_samples_split": 2,
        "min_samples_leaf": 1,
        "random_state": 42
    }
}' > params/config.json
```

## Running the Data Download Script

Attempt to run the data download script:

```bash
datalad run -m "Run Download Data Script. Add Raw Diabetes Dataset." \
    --output "data/raw/diabetes_raw.csv" \
    "python code/download_data.py"
```

**Note:** The above command may fail because there are untracked changes in the `params` folder (`config.json`).

Save the `config.json` file to track it with Git:

```bash
datalad save -m "Add parameter configurations" params/config.json
```

Now, run the data download script again:

```bash
datalad run -m "Run Download Data Script. Add Raw Diabetes Dataset." \
    --output "data/raw/diabetes_raw.csv" \
    "python code/download_data.py"
```

View the directory structure:

```bash
tree
```

## Processing the Data

Process the raw data into training and test sets:

```bash
datalad run -m "Process raw data into training and test sets" \
    --input data/raw/diabetes_raw.csv \
    --output data/processed/train/diabetes_train.csv \
    --output data/processed/test/diabetes_test.csv \
    "python code/process_data.py"
```

View the directory structure:

```bash
tree
```

Check the Git commit log:

```bash
git log
```

## Training and Evaluating Models

Set an experiment name:

```bash
EXPERIMENT_NAME="decision_tree"
```

Train the Decision Tree model:

```bash
datalad run -m "Train Decision Tree model for ${EXPERIMENT_NAME}" \
    --input data/processed/train/diabetes_train.csv \
    --input params/config.json \
    --output results/${EXPERIMENT_NAME}/model.joblib \
    "python code/train.py ${EXPERIMENT_NAME}"
```

Evaluate the model and generate metrics and plots:

```bash
datalad run -m "Evaluate model for ${EXPERIMENT_NAME} with ROC curve plot" \
    --input data/processed/test/diabetes_test.csv \
    --input results/${EXPERIMENT_NAME}/model.joblib \
    --output results/${EXPERIMENT_NAME}/metrics.json \
    --output results/${EXPERIMENT_NAME}/predictions.csv \
    --output results/${EXPERIMENT_NAME}/roc_curve.png \
    "python code/evaluate.py ${EXPERIMENT_NAME}"
```

## Modifying Parameters and Re-running

Change the `max_depth` parameter in `config.json`:

```bash
echo '{
    "model": "DecisionTree",
    "parameters": {
        "criterion": "gini",
        "splitter": "best",
        "max_depth": 5,
        "min_samples_split": 2,
        "min_samples_leaf": 1,
        "random_state": 42
    }
}' > params/config.json
```

View the directory structure:

```bash
tree
```

Save the changes to `config.json`:

```bash
datalad save -m "Change max_depth in parameters ${EXPERIMENT_NAME}" params/config.json
```

Re-train the model with the new parameters:

```bash
datalad run -m "Train Decision Tree model for ${EXPERIMENT_NAME}" \
    --input data/processed/train/diabetes_train.csv \
    --input params/config.json \
    --output results/${EXPERIMENT_NAME}/model.joblib \
    "python code/train.py ${EXPERIMENT_NAME}"
```

Re-evaluate the model:

```bash
datalad run -m "Evaluate model for ${EXPERIMENT_NAME} with ROC curve plot" \
    --input data/processed/test/diabetes_test.csv \
    --input results/${EXPERIMENT_NAME}/model.joblib \
    --output results/${EXPERIMENT_NAME}/metrics.json \
    --output results/${EXPERIMENT_NAME}/predictions.csv \
    --output results/${EXPERIMENT_NAME}/roc_curve.png \
    "python code/evaluate.py ${EXPERIMENT_NAME}"
```

## Comparing Results

Compare the metrics and parameters between the two branches:

```bash
git diff branch_1 branch_2 -- results/${EXPERIMENT_NAME}/metrics.json params/config.json
```

This command will show the differences in the model metrics and parameter configurations between the two experiments.

---

Follow these steps to replicate the project and understand how Datalad can help manage data and code in a reproducible way. Each command is provided in its own code block for clarity and ease of use during your code-along demonstration.
