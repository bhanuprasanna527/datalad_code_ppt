### DataLad Code-Along Demonstration

This README provides a guide for demonstrating how to use **DataLad** to manage a machine learning project. The steps will show how to set up a project, track changes, run scripts, and handle multiple experiments efficiently. We will use a Diabetes dataset as an example.

#### 1. Install DataLad and Verify Installation

Check that **DataLad** is installed and working:
```bash
datalad --version
```

#### 2. Initialize the Project Directory

Create a new project folder and initialize a **DataLad** repository:
```bash
mkdir diabetes
cd diabetes
datalad create -c yoda .
```

#### 3. Set Up the Project Structure

Create the necessary folder structure for code, data, and results:
```bash
mkdir -p code data/raw data/processed/train data/processed/test results params
```

#### 4. Configure Git Attributes

Configure Git and **Git-annex** settings:
```bash
echo 'results/**/* annex.largefiles=nothing' >> .gitattributes
echo 'params/**/* annex.largefiles=nothing' >> .gitattributes
datalad save -m "Initialize project structure and edit .gitattributes" .gitattributes
```

#### 5. Download Python Code Scripts

Download the required scripts using **DataLad**:
```bash
datalad run -m "Download download_data.py script" \
    --output code/download_data.py \
    "wget https://raw.githubusercontent.com/bhanuprasanna2001/datalad-demo/master/code/download_data.py -O code/download_data.py"

datalad run -m "Download evaluate.py script" \
    --output code/evaluate.py \
    "wget https://raw.githubusercontent.com/bhanuprasanna2001/datalad-demo/master/code/evaluate.py -O code/evaluate.py"

datalad run -m "Download process_data.py script" \
    --output code/process_data.py \
    "wget https://raw.githubusercontent.com/bhanuprasanna2001/datalad-demo/master/code/process_data.py -O code/process_data.py"

datalad run -m "Download train.py script" \
    --output code/train.py \
    "wget https://raw.githubusercontent.com/bhanuprasanna2001/datalad-demo/master/code/train.py -O code/train.py"
```

#### 6. Define Initial Model Parameters

Create the configuration file for model parameters:
```bash
datalad run -m "Create initial model configuration" \
    --output params/config.json \
    "echo '{
        \"model\": \"DecisionTree\",
        \"parameters\": {
            \"criterion\": \"gini\",
            \"splitter\": \"best\",
            \"max_depth\": 7,
            \"min_samples_split\": 2,
            \"min_samples_leaf\": 1,
            \"random_state\": 42
        }
    }' > params/config.json"
```

#### 7. Download Data

Run the script to download the Diabetes dataset:
```bash
datalad run -m "Download raw Diabetes dataset" \
    --output data/raw/diabetes_raw.csv \
    "python code/download_data.py"
```

#### 8. Process Raw Data into Training and Test Sets

Run the script to split the raw data into training and test sets:
```bash
datalad run -m "Process raw data into training and test sets" \
    --input data/raw/diabetes_raw.csv \
    --output data/processed/train/diabetes_train.csv \
    --output data/processed/test/diabetes_test.csv \
    "python code/process_data.py"
```

#### 9. Train the Model

Train the decision tree model using the current configuration:
```bash
EXPERIMENT_NAME="decision_tree_depth_7"

datalad run -m "Train Decision Tree model for ${EXPERIMENT_NAME}" \
    --input data/processed/train/diabetes_train.csv \
    --input params/config.json \
    --output results/${EXPERIMENT_NAME}/model.joblib \
    "python code/train.py ${EXPERIMENT_NAME}"
```

#### 10. Evaluate the Model

Evaluate the trained model and generate performance metrics:
```bash
datalad run -m "Evaluate Decision Tree model for ${EXPERIMENT_NAME}" \
    --input data/processed/test/diabetes_test.csv \
    --input results/${EXPERIMENT_NAME}/model.joblib \
    --output results/${EXPERIMENT_NAME}/metrics.json \
    --output results/${EXPERIMENT_NAME}/predictions.csv \
    --output results/${EXPERIMENT_NAME}/roc_curve.png \
    "python code/evaluate.py ${EXPERIMENT_NAME}"
```

#### 11. Modify Model Parameters

To experiment with different parameters, modify the configuration file:
```bash
datalad run -m "Update model configuration to max_depth=5" \
    --output params/config.json \
    "echo '{
        \"model\": \"DecisionTree\",
        \"parameters\": {
            \"criterion\": \"gini\",
            \"splitter\": \"best\",
            \"max_depth\": 5,
            \"min_samples_split\": 2,
            \"min_samples_leaf\": 1,
            \"random_state\": 42
        }
    }' > params/config.json"
```

#### 12. Train the Model with New Parameters

Train the model again with the updated parameters:
```bash
EXPERIMENT_NAME="decision_tree_depth_5"

datalad run -m "Train Decision Tree model for ${EXPERIMENT_NAME}" \
    --input data/processed/train/diabetes_train.csv \
    --input params/config.json \
    --output results/${EXPERIMENT_NAME}/model.joblib \
    "python code/train.py ${EXPERIMENT_NAME}"
```

#### 13. Evaluate the New Model

Evaluate the model with the new parameters and generate metrics:
```bash
datalad run -m "Evaluate Decision Tree model for ${EXPERIMENT_NAME}" \
    --input data/processed/test/diabetes_test.csv \
    --input results/${EXPERIMENT_NAME}/model.joblib \
    --output results/${EXPERIMENT_NAME}/metrics.json \
    --output results/${EXPERIMENT_NAME}/predictions.csv \
    --output results/${EXPERIMENT_NAME}/roc_curve.png \
    "python code/evaluate.py ${EXPERIMENT_NAME}"
```

#### 14. Compare Results Across Experiments

Finally, use `git diff` to compare the results from different experiments:
```bash
git diff branch_1 branch_2 -- results/decision_tree_depth_7/metrics.json params/config.json
```

This guide demonstrates how **DataLad** can help manage data, code, and experiments for reproducible machine learning workflows.