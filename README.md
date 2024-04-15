# Neural Inventory Control
Implementation of Hindsight Differentiable Policy Optimization, as described in the paper [Neural Inventory Control in Networks via Hindsight Differentiable Policy Optimization](https://arxiv.org/abs/2306.11246).

## Introduction

Inventory management offers unique opportunities for reliably evaluating and applying deep reinforcement learning (DRL). We introduce Hindsight Differentiable Policy Optimization (HDPO), facilitating direct optimization of a DRL policy's hindsight performance using stochastic gradient descent. HDPO leverages two key elements: (i) an ability to backtest any policy's performance on a sample of historical scenarios, and (ii) the differentiability of the total cost incurred in a given scenario. We assess this approach in four problem classes where we can benchmark performance against the true optimum. HDPO algorithms consistently achieve near-optimal performance across all these classes, even when dealing with up to 60-dimensional raw state vectors. Moreover, we propose a natural neural network architecture to address problems with weak (or aggregate) coupling constraints between locations in an inventory network. This architecture utilizes weight duplication for "sibling" locations and state summarization. We demonstrate empirically that this design significantly enhances sample efficiency and provide justification for it through an asymptotic performance guarantee. Lastly, we assess HDPO in a setting that incorporates real sales data from a retailer, demonstrating its superiority over generalized newsvendor strategies.

## Citation

You can cite our work using the following bibtex entry:

```
@article{alvo2023neural,
  title={Neural inventory control in networks via hindsight differentiable policy optimization},
  author={Alvo, Matias and Russo, Daniel and Kanoria, Yash},
  journal={arXiv preprint arXiv:2306.11246},
  year={2023}
}
```


## Installation

To set up the environment for this project, follow these steps:

1. Clone this repository to your local machine:

```
git clone git@github.com:MatiasAlvo/Neural_inventory_control.git
```

2. Navigate to the project directory:
```
cd Neural_inventory_control
```

3. Create a conda environment using the provided environment.yml file
```
conda env create -f environment.yml
```

4. Activate the conda environment:
```
conda activate neural_inventory_control
```

5. You can then run HDPO by running `main_run.py`. In that same file, you can define the config files to read from.

## Usage

The main functionalities for the code are in the following scripts:

1. `data_handling.py`: Defines how to create a Scenarios class (which is actually a collection of scenarios). This includes sampling parameters (such as demand or costs when they are obtained by sampling), and sampling the demand traces. It also defined the Dataset class.

2. `environment.py`: Defines the functionalities of the environment (simulator).

3. `main_run.py`: Defines which config files to read from (for the setting and the policy class to use) and runs HDPO.

4. `neural_networks.py`: Defines the neural network functionalities.

5. `trainer.py`: Defines the Trainer class, which is in charge of the training itself, including handling the interactions with the simulator and updating the neural network's weights.

Parameters for settings/instances and policies are to be defined in a config file under the **config_files/settings** and **config_files/policies_and_hyperparams**, respectively. Instructions on how to do this are detailed in a later section. 

## License

MIT License

Copyright 2024 Matias Alvo, Daniel Russo, Yashodhan Kanoria

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the “Software”), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED “AS IS”, WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

## Populating a config file

We provide config files for all the settings and all the policies described in our paper. A detailed description of its usage is as follows.


### Setting config file

#### `seeds` and `test_seeds`:
- `underage_cost`: Seed to sample underage costs, in case they are random.
- `holding_cost`: Seed to sample holding costs, in case they are random.
- `mean`: Seed to sample mean parameters, in case they are random.
- `coef_of_var`: Seed to sample the coefficient of variations (which defines the standard deviation), in case they are random.
- `lead_time`: Seed to sample lead times, in case they are random.
- `demand`: Seed to sample demand traces.
- `initial_inventory`: Seed to sample the value of initial inventories at the stores.

#### `sample_data_params`:
- `split_by_period`: Whether to split the dataset by period or by sample. If True (currently only used for setting with real data), train, dev and test sets
are generated by copying all values (costs and lead times), and splitting demand traces into disjoint sets of periods.

#### `problem_params`: 
- `n_stores`: The number of stores in the inventory network.
- `n_warehouses`: The number of warehouses in the inventory network.
- `n_extra_echelons`: The number of extra echelons in the inventory network.
- `lost_demand`: Whether to consider unmet demand to be lost or backlogged.
- `maximize_profit`: True if objective is to maximize profit. False if objective is to minimize cost.

#### `params_by_dataset`: one dictionary for each of train, dev and test, containing
- `n_samples`: The number of samples in the dataset.
- `batch_size`: The batch size for training.
- `periods`: The number of periods to simulate.
- `ignore_periods`: The number of initial periods to ignore for purposes of reporting costs.

#### `observation_params`: defines which information is containted in the observation
- `include_warehouse_inventory`: Whether to include warehouse inventory in observations.
- `include_static_features`: 
  - `holding_costs`: Whether to include holding costs in observations.
  - `underage_costs`: Whether to include underage costs in observations.
  - `lead_times`: Whether to include lead times in observations.
- `demand`: 
  - `past_periods`: The number of past periods to include in demand observations.
  - `period_shift`: The shift in demand periods. It shifts the beginning of the planning horizon from 0 to this value.
- `include_past_observations`:
  - `arrivals`: The number of past arrivals to include in observations.
  - `orders`: The number of past orders to include in observations.
- `time_features_file`: Path to the file containing time-related features.
- `time_features`: List of time-related features to include (e.g., 'days_from_christmas').
- `sample_features_file`: Path to the file containing scenario-related features (e.g., product type).
- `sample_features`: List of scenario-related features to include (e.g., 'store_nbr').

#### `store_params`:
- `demand`: 
  - `sample_across_stores`: Whether to sample the parameters for each store. Only works for normal distributions. If True, sample in the ranges specified in `mean_range` and `coef_of_var_range`.
  - `vary_across_samples`: Whether to sample the parameters for each different scenario. Only works for normal distributions. If True, sample in the ranges specified in `mean_range` and `coef_of_var_range`.
  - `expand`: Whether the copy demand parameters across all stores and scenarios by "expanding". If True, we copy the values of `mean` and `std`.
  - `mean_range`: The range from which to sample the mean of demands.
  - `coef_of_var_range`: The range of the coefficient of variation of the demand distribution from which to sample.
  - `mean`: Mean of the distribution, which is fixed across all stores and samples.
  - `std`: Standard deviation of the distribution, which is fixed across all stores and samples.
  - `distribution`: The distribution of demand. Can be normal, poisson or real.
  - `correlation`: The correlation coefficient for pairwise demands.
  - `clip`: Whether to clip demand values below from 0.
  - `decimals`: The number of decimals for demand values.
- `lead_time/holding_cost/underage_cost`: 
  - `sample_across_stores`: Whether to sample the parameters for each store.
  - `vary_across_samples`: Whether to sample the parameters for each different scenario. If True, sample in the range specified in `range`.
  - `expand`: Whether the copy demand parameters across all stores and scenarios by "expanding". If True, we copy the value `value`.
  - `range`: The range from which to sample the parameter values.
  - `value`: Value to be copied across all stores and scenarios
  - `file_location`: Location for directly reading the values of the parameters, instead of defining them manually.
- `initial_inventory`: 
  - `sample`: True if considering a random initial inventory, given as random_value*mean_demand_per_store.
  - `range_mult`: The range from which to sample random_value.
  - `inventory_periods`: How many periods to consider for the inventories state (which might be larger that lead time, if specified).

#### `warehouse_params`:
- `holding_cost`: The holding cost for the warehouse.
- `lead_time`: The lead time for the warehouse.

#### `echelon_params`:
  - `holding_cost`: A list specifying the holding cost for each echelon.
  - `lead_time`: A list specifying the lead time for each echelon.

### Config file for Hyperparameters and Neural Network Architectures

#### `trainer_params`:
- `epochs`: The number of epochs to train the neural network.
- `do_dev_every_n_epochs`: Perform evaluation on the dev dataset every `n` epochs.
- `print_results_every_n_epochs`: Print training results every `n` epochs.
- `save_model`: Whether to save the trained model.
- `epochs_between_save`: Number of epochs between model saving (and only if performance improved).
- `choose_best_model_on`: Performance on which to select the best model. Can be `dev_loss` or `train_loss`
- `load_previous_model`: Whether to load a previously trained model.
- `load_model_path`: Path to the previously trained model.

#### `optimizer_params`:
- `learning_rate`: The learning rate used by the optimizer.

#### `nn_params`:
- `name`: The name of the neural network architecture.
- `inner_layer_activations`: Activation functions for inner layers of each component/module of the neural network.
- `output_layer_activation`: Activation function for the output layer of each component/module of the neural network.
- `neurons_per_hidden_layer`: Number of neurons for each hidden layer of each component/module of the neural network.
- `output_sizes`: Size of the output layer for each component of the neural network.
- `initial_bias`: Initial bias for the last layer of each component/module of the neural network.
- `forecaster_location`: Location for loading quantile forecaster, if used within the policy.
- `warehouse_upper_bound_mult`: Multiplier to calculate the upper bound for warehouse outputs.
