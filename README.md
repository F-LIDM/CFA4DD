[![License:
MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![PyPI version](https://badge.fury.io/py/cfanalysis.svg)](https://badge.fury.io/py/cfanalysis)
[![Downloads](https://static.pepy.tech/badge/cfanalysis)](https://www.pepy.tech/projects/cfanalysis)
[![Downloads](https://static.pepy.tech/badge/cfanalysis/month)](https://www.pepy.tech/projects/cfanalysis)

# cfanalysis
The repository provides code and data for the paper "Enhancing ADMET Property Models Performance through Combinatorial Fusion Analysis", categorized by drug encoding schemes: Morgan Circular Fingerprints, RDKit 2D descriptors, and MCFP. 

# Python package: cfanalysis

[Library to perform Combinatorial Fusion Analysis](https://pypi.org/project/cfanalysis/)

## :warning: Introduction

A Combinatorial Fusion Analysis (CFA) to enhance Absorption, Distribution, Metabolism, Excretion, and Toxicity (ADMET) performance of ML models. CFA models show superior performance compared to most of the individual ML models. The CFA model architecture and the performance of CFA models on TDC and other internal datasets is presented in the paper. Significant enhancement suggests that CFA is a viable tool for improving ADMET ML model performance, offering promise for faster and more cost-effective drug development pipelines. 

## :woman_technologist: Installation

You can install this package using `pip`. Run the following commands to install it and then import it:

```python
pip install cfanalysis==0.1.9
import cfanalysis
from cfanalysis import cfafunctions
```

## :zap: Usage

An example using [Therapeutic Data Common's](https://tdcommons.ai/) (TDC) `Clearance_Microsome_AZ` dataset. 
### Import the required libraries

```python
import numpy as np
import pandas as pd

from tdc.single_pred import ADME
from tdc.benchmark_group import admet_group
from tdc import BenchmarkGroup
```

### Import the TDC dataset and input our predictions: :warning: you may use your own dataset and predictions
`Clearance_Microsome_AZ` has a continuous target variable (regression) 

```python
group = admet_group(path = 'data/')
name = 'Clearance_Microsome_AZ' #you need to change dataset name to get the model fusion result
benchmark = group.get(name)
train_val, test = benchmark['train_val'], benchmark['test']
y_test = np.array(test.Y) # store the target variable for the test set

y_valid = {}  # create a dictionary to store target variable for the validation set for each seed

for seed in [1, 2, 3, 4, 5]:
    train, valid = group.get_train_valid_split(benchmark=name, split_type='default', seed=seed)
    y_valid[seed] = valid.Y # store the target variable for the validation set

# read in the dataframes with predicted values for the validation sets and the test sets, these datasets are available in the data dir
predictions_val_xgb = pd.read_csv('%s_predictions_val_xgb.csv' % name)
predictions_val_rf = pd.read_csv('%s_predictions_val_rf.csv' % name)
predictions_val_svm = pd.read_csv('%s_predictions_val_svm.csv' % name)


predictions_test_xgb = pd.read_csv('%s_predictions_test_xgb.csv' % name)
predictions_test_rf = pd.read_csv('%s_predictions_test_rf.csv' % name)
predictions_test_svm = pd.read_csv('%s_predictions_test_svm.csv' % name)
```

### :memo: Input model data 
```python
model_names = ['xgb', 'rf', 'svm'] # mention model names 
preds = cfafunctions.model_predictions(
    len(model_names),
    model_names,
    val_dfs_list=[predictions_val_xgb, predictions_val_rf, predictions_val_svm],
    test_dfs_list=[predictions_test_xgb, predictions_test_rf, predictions_test_svm]
)
```

`cfafunctions.model_predictions` takes 4 arguments: 

`len(model_names)`: Number of models to perfrom combinatorial fusion analysis

`model_names`: Vector of model names for identification

`val_dfs_list`: Predictions from the validation set for each ML model

`test_dfs_list`: Predictions from the test set for each ML model

:warning: Predictions, `val_dfs_list` and `test_dfs_list`, need to be probabilities for the positive class for classification ML models and predictions from 5 seeds are required for each model. 

### :white_check_mark: Obtain the results from the CFA analysis: 
```bash
cfafunctions.calculate_spearman_corr(model_names, preds, y_test, y_valid) # use Spearman-Rank correlation (regression) 
```
Other options for metrics for CFA
```python
>> cfafunctions.calculate_MAE(model_names, preds, y_test, y_valid) # use Mean Absolute Error (regression) 
>> cfafunctions.calculate_auroc(model_names, preds, y_test, y_valid) # use Area Under the Receiver Operating Characteristics (classification)
>> cfafunctions.calculate_auprc(model_names, preds, y_test, y_valid) # use Area Under the Precision-Recall Curve (classification)
```

### :chart_with_upwards_trend: Interpreting results 
```
modelcombination_ds: implies diversity strength
modelcombination_r: implies rank combination
modelcombination_ps: implies performance strength
modelcombination: implies average score strength
modelname: implies raw model metric
```

## :paperclip: Citation 
Jiang, Nan, et al. "Enhancing ADMET Property Models Performance through Combinatorial Fusion Analysis." arXiv preprint arXiv:#### (2023).
```bib
@article{Jiang2023cfa,
      title={Enhancing ADMET Property Models Performance through Combinatorial Fusion Analysis}, 
      author={Nan Jiang and Mohammed Quazi and Christina Schweikert and D. Frank Hsu and Tudor I. Oprea and Suman Sirimulla},
      year={2023},
      eprint={####},
      archivePrefix={arXiv},
      primaryClass={####},
      publisher={arXiv}
}
```
