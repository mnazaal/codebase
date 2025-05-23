
<h1>Multi-Agent Reinforcement Learning:
Foundations and Modern Approaches</h1>
  
<h2>
  Book Codebase: www.marl-book.com
</h2>

Cite the book using:
```latex
@book{marl-book,
    author = {Stefano V. Albrecht and Filippos Christianos and Lukas Sch\"afer},
    title = {Multi-Agent Reinforcement Learning: Foundations and Modern Approaches},
    publisher = {MIT Press},
    year = {2024},
    url = {https://www.marl-book.com}
}
```

This codebase is part of the [MARL book](http://www.marl-book.com) and provides access to basic and easy-to-understand MARL ideas. 
The algorithms are self-contained and the implementations are focusing on simplicity.
Implementation tricks, while necessary for some algorithms, are sparse as not to make the code very complicated. As a result, some performance has been sacrificed.

All algorithms are implemented in [_PyTorch_](https://pytorch.org/) and use the [_Gymnasium_](https://gymnasium.farama.org/) interface.

<h1>Table of Contents</h1>

- [Getting Started](#getting-started)
  - [Installation](#installation)
  - [Running an algorithm](#running-an-algorithm)
    - [(Optional) Use Hydra's tab completion](#optional-use-hydras-tab-completion)
  - [Running a hyperparameter search](#running-a-hyperparameter-search)
    - [An advanced hyperparameter search using `search.py`](#an-advanced-hyperparameter-search-using-searchpy)
  - [Logging](#logging)
    - [File System Logger](#file-system-logger)
    - [WandB Logger](#wandb-logger)
- [Implementing your own algorithm/ideas](#implementing-your-own-algorithmideas)
- [Interpreting your results](#interpreting-your-results)
- [Implemented Algorithms](#implemented-algorithms)
  - [Parameter Sharing](#parameter-sharing)
  - [Value Decomposition](#value-decomposition)
- [Contact](#contact)


# Getting Started

## Installation

We *strongly* suggest you use a virtual environment for the instructions below. A good starting point is [Miniconda](https://docs.conda.io/en/latest/miniconda.html), with which you would do:

```sh
conda create -n marlbase python=3.10
conda activate marlbase
```

Then, clone and install the repository using: 

```sh
git clone https://github.com/marl-book/codebase.git
cd codebase
pip install -r requirements.txt
pip install -e .
```
Do not forget to install PyTorch in your environment. Instructions for your system/setup can be found here: https://pytorch.org/get-started/locally/

## Running an algorithm
This project uses [Hydra](https://hydra.cc/) to structure its configuration. Algorithm implementations can be found under `marlbase/`. The respective configs are found in `marlbase/configs/algorithms/`.

You would first need an environment that is registered in Gymnasium. This repository uses the Gymnasium API (with the only difference being that the rewards are a tuple or list - one for each agent). 

A good starting point would be [Level-based Foraging](https://github.com/uoe-agents/lb-foraging) and [RWARE](https://github.com/uoe-agents/robotic-warehouse). You can install both using:
```sh
pip install -U lbforaging rware
```

Then, running an algorithm (e.g. IA2C) looks like:

```sh
cd marlbase
python run.py +algorithm=ia2c env.name="lbforaging:Foraging-8x8-2p-3f-v3" env.time_limit=25
```

Similarly, running IDQN can be done using:
```sh
python run.py +algorithm=idqn env.name="lbforaging:Foraging-8x8-2p-3f-v3" env.time_limit=25
```

Overriding hyperparameters is easy and can be done in the command line. An example of overriding the `batch_size` in IDQN:
```sh
python run.py +algorithm=idqn env.name="lbforaging:Foraging-8x8-2p-3f-v3" env.time_limit=25 algorithm.batch_size=256
```

Find other hyperparameters in the files under `marlbase/configs/algorithm`.

### (Optional) Use Hydra's tab completion
Hydra also supports tab completion for filling in the hyperparameters. See [here](https://hydra.cc/docs/tutorials/basic/running_your_app/tab_completion), and install it with:
```sh
eval "$(python run.py -sc install=bash)"
```
## Running a hyperparameter search

Can be easily done using [Hydra's multirun](https://hydra.cc/docs/tutorials/basic/running_your_app/multi-run) option. An example of sweeping over batch sizes is:

```sh
python run.py -m +algorithm=idqn env.name="lbforaging:Foraging-8x8-2p-3f-v3" env.time_limit=25 algorithm.batch_size=32,64,128
```

### An advanced hyperparameter search using `search.py`
*This section might get deprecated in the future if Hydra implements this feature.*

We include a script named `search.py` which reads a search configuration file (e.g. the included `configs/sweeps/sample.yaml`) and runs a hyperparameter search in one or more tasks. The script can be run using
```sh
python search.py run --config configs/sweeps/sample.yaml --seeds 5 locally
```
In a cluster environment where one run should go to a single process, it can also be called in a batch script like:
```sh
python search.py run --config configs/sweeps/sample.yaml --seeds 5 single $TASK_ID
```
Where `$TASK_ID` is an index for the experiment (i.e. 1...#number of experiments).

## Logging
We implement two loggers: FileSystem Logger and WandB Logger.

### File System Logger
The default logger is the FileSystemLogger which saves experiment results in a `results.csv` file. You can find that file, the configuration that has been used & more under `outputs/{env_name}/{alg_name}/{random_hash}` or `multirun/{date}/{time}/{experiment_id}` for multiruns.
### WandB Logger
By appending `+logger=wandb` in the command line you can get support for WandB. Do not forget to `wandb login` first.

Example:

```sh
python run.py +algorithm=idqn env.name="lbforaging:Foraging-8x8-2p-3f-v3" env.time_limit=25 logger=wandb
```
You can override the project name using:

```sh
python run.py +algorithm=idqn env.name="lbforaging:Foraging-8x8-2p-3f-v3" env.time_limit=25 logger=wandb logger.project_name="my-project-name"
```

# Implementing your own algorithm/ideas

The fastest way would be to create a new folder starting from the algorithm of your choice e.g.
```sh
cp -R ac ac_new_idea
```
and create a new configuration file:
```sh
cp configs/algorithm/ia2c.yaml configs/algorithm/ac_new_idea.yaml
```

with the editor of your choice, open `ac_new_idea.yaml` and change
```yaml
...
algorithm:
  _target_: ac.train.main
  name: "ac"
  model:
    _target_: ac.model.A2CNetwork
...
```
to 
```yaml
...
algorithm:
  _target_: ac_new_idea.train.main
  name: "ac_new_idea"
  model:
    _target_: ac_new_idea.model.NewNetwork
...
```
Make any changes you want to the files under `ac_new_idea/` and run it using:

```sh
python run.py +algorithm=ac_new_idea env.name="lbforaging:Foraging-8x8-2p-3f-v3" env.time_limit=25
```
You can now add new hyperparameters, change the training procedure, or anything else you want and keep the old implementations for easy comparison. We hope that the way we have implemented these algorithms makes it easy to change any part of the algorithm without the hustle of reading through large code-bases and huge unnecessary layers of abstraction. RL research benefits from iterating over ideas quickly to see how they perform!

# Interpreting your results

We have multiple tools to analyze the outputs of FileSystemLogger (for WandBLogger, just login to their webpage).

You can easily find the best hyperparameter configuration per environment using: 
```sh
python utils/postprocessing/find_best_hyperparams.py  --source <PATH/TO/SOURCE/DIR>
```
By default, this script will determine the best hyperparameters based on the average total returns across all evaluations and seeds. To use a different metric, you can specify the desired metric (from the `results.csv` files) with the `--metric` argument.

Similarly, you can plot the stored runs (average/std across seeds) using:
```sh
python utils/postprocessing/plot_runs.py --source <PATH/TO/SOURCE/DIR>
```
By default, this will visualise the mean and std across seeds of the `mean_episode_returns` metric. You can specify the metric to plot using the `--metric` argument. You can also provide the additional `--save_path` argument to save the plot as a `.pdf` file.

We also provide a script to export the data of multiple runs as a pandas dataframe using:
```sh
python utils/postprocessing/export_multirun.py --folder folder/containing/results --export-file myfile.hd5
```
The file will contain two pandas DataFrames: `df` which contains all `mean_episode_returns` (by default summed across all agents), and `config` which contains information about the tested hyperparameters.  
You can load both through Python using:
```python
import pandas as pd
df = pd.read_hdf("myfile.hd5", "df")
configs = pd.read_hdf("myfile.hd5", "configs")
```
The imported DataFrames look like the ones below. `df` has a multi-index column indexing the environment name, the algorithm name, a hash unique to the parameter search, and a seed. `configs` maps the hash to the full configuration of the run.

```ipython
In [1]: df
Out[2]: 
                       Foraging-20x20-9p-6f-v3             ...                       
                                       Algo1               ...     Algo2             
                                   f7c2ecb3ddf1            ... 5284ad99ce02          
                                         seed=0    seed=1  ...       seed=0    seed=1
environment_steps                                          ...                       
0                                      0.178373  0.000000  ...     0.089167  0.054286
100000                                 0.026786  0.066667  ...     0.054545  0.033333
200000                                 0.130278  0.084650  ...     0.043333  0.055833
300000                                 0.086111  0.109975  ...     0.182626  0.116768
...

In [3]: configs
Out[4]: 
             algorithm.name  algorithm.lr  algorithm.batch_size
f7c2ecb3ddf1       DQN-FuPS        0.0001                   256
ecaf120f572e       DQN-SePS        0.0001                   128
5a80fe220cfc       DQN-SePS        0.0003                   128
d16939a558b6       DQN-FuPS        0.0003                   256
...
```

Finally you can use [HiPlot](https://github.com/facebookresearch/hiplot) to interactively visualize the performance of various hyperparameter configurations using:
```sh
pip install -U hiplot
hiplot marlbase.utils.postprocessing.hiplot_fetcher.experiment_fetcher
```
You will have to enter `exp://myfile.hd5/env_name/alg_name` in the browser's textbox.


# Implemented Algorithms

|                             | IA2C                | MA-A2C             | IPPO               | MA-PPO             | DQN (Double Q)     | VDN                 | QMIX                |
|-----------------------------|---------------------|--------------------|--------------------|-------------------|--------------------|---------------------|---------------------|
| Parameter Sharing           | :heavy_check_mark:  | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark:  | :heavy_check_mark:  | 
| Selective Parameter Sharing | :heavy_check_mark:  | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark:  | :heavy_check_mark:  | 
| Return Standardisation      | :heavy_check_mark:  | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark:  | :heavy_check_mark:  | 
| Reward Standardisation      | :heavy_check_mark:  | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark:  | :heavy_check_mark:  | 
| Target Networks             | :heavy_check_mark:  | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark:  | :heavy_check_mark:  | 


## Parameter Sharing

Parameter sharing across agents is optional and being done behind the scenes in the torch model.
There are three types of parameter sharing:
- No Parameter Sharing (default)
- Full Parameter Sharing
- Selective Parameter Sharing ([Christianos et al.](https://arxiv.org/pdf/2102.07475.pdf))

For example, for IDQN you can enable either of these using:
```sh
python run.py +algorithm=dqn env.name="lbforaging:Foraging-8x8-4p-3f-v3" env.time_limit=25 algorithm.model.parameter_sharing=False
python run.py +algorithm=dqn env.name="lbforaging:Foraging-8x8-4p-3f-v3" env.time_limit=25 algorithm.model.parameter_sharing=True
python run.py +algorithm=dqn env.name="lbforaging:Foraging-8x8-4p-3f-v3" env.time_limit=25 "algorithm.model.parameter_sharing=[0,0,1,1]"
```
for each of the methods respectively. For Selective Parameter Sharing, you need to supply a list of indices pointing to the network that is going to be used for each agent. Example: `[0,0,1,1]` as above makes the agents `0` and `1` share network `0` and agents `2` and `3` share the network `1`. Similarly `[0,1,1,1]` would make the first agent not share parameters with anyone, and the other three would share parameters.

In actor-critic methods you would need to separately define parameter sharing for the actor and the critic. The respective config is `algorithm.model.actor.parameter_sharing=...` and `algorithm.model.critic.parameter_sharing=...`

## Value Decomposition

We have implemented VDN and QMIX on top of the DQN algorithm. To use load the respective algorithm config with:

```sh
python run.py +algorithm=vdn env.name="lbforaging:Foraging-8x8-4p-3f-v3" env.time_limit=25
```

Note that for this to work we use the `CooperativeReward` wrapper that _sums_ the rewards of all agents before feeding them to the training algorithm. If you have an environment that already has a cooperative reward, you still need it to return a *list of rewards* (e.g. `reward = n_agents * [reward/n_agents]`).


# Contact
- Filippos Christianos - filippos {dot} christianos {at} gmail {dot} com
- Lukas Schäfer - luki {dot} schaefer96 {at} gmail {dot} com

Based on: https://github.com/semitable/fast-marl (by Filippos Christianos)

