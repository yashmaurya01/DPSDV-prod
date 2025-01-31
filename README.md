# DP-SDV

## Performance:

|name              |PRIV_METRIC_NumericalLR|PRIV_METRIC_NumericalMLP|PRIV_METRIC_NumericalSVR|
|------------------|-----------------------|------------------------|------------------------|
|FAST_ML-DP        |0.083478628            |**0.1780978**               |0.071120968             |
|FAST_ML           |0.087402661            |0.176534839             |0.074326679             |
|Gaussian Copula-DP|**0.093651126**            |**0.177785047**             |**0.189530896**             |
|Gaussian Copula   |0.066141002            |0.176947755             |0.073740679             |
|CT-GAN-DP         |**0.166319716**            |**0.178170664**             |**0.189561336**             |
|CT-GAN            |0.072312111            |0.173496572             |0.078755983             |
|Copula-GAN-DP     |**0.162198436**            |**0.179451562**             |**0.18955882**              |
|Copula-GAN        |0.084989631            |0.177600009             |0.07810617              |
|TVAE-DP           |**0.073633603**            |**0.176959062**             |0.071933513             |
|TVAE              |0.053901697            |0.175550734             |0.075317994             |


## Install

**Using `pip`:**

```bash
pip install DPSDV
```

## Quickstart

In this short tutorial we will guide you through a series of steps that will help you
getting started using **SDV**.

### 1. Model the dataset using SDV

To model a multi table, relational dataset, we follow two steps. In the first step, we will load
the data and configures the meta data. In the second step, we will use the sdv API to fit and
save a hierarchical model. We will cover these two steps in this section using an example dataset.

#### Step 1: Load example data

**SDV** comes with a toy dataset to play with, which can be loaded using the `sdv.load_demo`
function:

```python3
from DPSDV import load_demo

metadata, tables = load_demo(metadata=True)
```

This will return two objects:

1. A `Metadata` object with all the information that **SDV** needs to know about the dataset.

For more details about how to build the `Metadata` for your own dataset, please refer to the
[Working with Metadata](https://sdv.dev/SDV/user_guides/relational/relational_metadata.html)
tutorial.

2. A dictionary containing three `pandas.DataFrames` with the tables described in the
metadata object.

The returned objects contain the following information:

```
{
    'users':
            user_id country gender  age
          0        0     USA      M   34
          1        1      UK      F   23
          2        2      ES   None   44
          3        3      UK      M   22
          4        4     USA      F   54
          5        5      DE      M   57
          6        6      BG      F   45
          7        7      ES   None   41
          8        8      FR      F   23
          9        9      UK   None   30,
  'sessions':
          session_id  user_id  device       os
          0           0        0  mobile  android
          1           1        1  tablet      ios
          2           2        1  tablet  android
          3           3        2  mobile  android
          4           4        4  mobile      ios
          5           5        5  mobile  android
          6           6        6  mobile      ios
          7           7        6  tablet      ios
          8           8        6  mobile      ios
          9           9        8  tablet      ios,
  'transactions':
          transaction_id  session_id           timestamp  amount  approved
          0               0           0 2019-01-01 12:34:32   100.0      True
          1               1           0 2019-01-01 12:42:21    55.3      True
          2               2           1 2019-01-07 17:23:11    79.5      True
          3               3           3 2019-01-10 11:08:57   112.1     False
          4               4           5 2019-01-10 21:54:08   110.0     False
          5               5           5 2019-01-11 11:21:20    76.3      True
          6               6           7 2019-01-22 14:44:10    89.5      True
          7               7           8 2019-01-23 10:14:09   132.1     False
          8               8           9 2019-01-27 16:09:17    68.0      True
          9               9           9 2019-01-29 12:10:48    99.9      True
}
```

#### Step 2: Fit a model using the SDV API.

First, we build a hierarchical statistical model of the data using **SDV**. For this we will
create an instance of the `sdv.SDV` class and use its `fit` method.

During this process, **SDV** will traverse across all the tables in your dataset following the
primary key-foreign key relationships and learn the probability distributions of the values in
the columns.

```python3
from DPSDV.relational import HMA1

model = HMA1(metadata)
model.fit(tables)
```

OR

```python3
from DPSDV.relational import HMA1

model = HMA1(metadata)
model.fit(tables, eps=1e2)
```

to add differential privacy epsilon through argument `eps=1e2`

Once the modeling has finished, you can save your fitted `model` instance for later usage
using the `save` method of your instance.

```python3
model.save('sdv.pkl')
```

The generated `pkl` file will not include any of the original data in it, so it can be
safely sent to where the synthetic data will be generated without any privacy concerns.

### 2. Sample data from the fitted model

In order to sample data from the fitted model, we will first need to load it from its
`pkl` file. Note that you can skip this step if you are running all the steps sequentially
within the same python session.

```python3
model = HMA1.load('sdv.pkl')
```

After loading the instance, we can sample synthetic data by calling its `sample` method.

```python3
samples = model.sample()
```

The output will be a dictionary with the same structure as the original `tables` dict,
but filled with synthetic data instead of the real one.

## Implementations

1. Tabular Preset

So, adding noise based on the Wishart Mechanism for Differentially Private Principal Components Analysis paper's algorithm 1. Lap(0, 2d/ne), in this d is the number of columns in the covariance matrix taken from `model.get_parameters()`. Now, taking sensitivity=1. We modify the covariance matrix.

2. GaussianCopula Model

So, adding noise based on the Wishart Mechanism for Differentially Private Principal Components Analysis paper's algorithm 1. Lap(0, 2d/ne), in this d is the number of columns in the covariance matrix taken from `model.get_parameters()`. Now, taking sensitivity=1. We modify the covariance matrix.

3. CTGAN Model

Added DP-SGD

4. CopulaGAN Model

Added DP-SGD

5. TVAE Model

Added DP-SGD

6. MWEM Model

Added DP privacy

