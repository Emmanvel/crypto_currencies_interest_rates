# `README.md`

# Cryptocurrencies and Interest Rates: Inferring Yield Curves in a Bondless Market

<!-- PROJECT SHIELDS -->
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![Python Version](https://img.shields.io/badge/python-3.9%2B-blue.svg)](https://www.python.org/downloads/)
[![Code style: black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)
[![Imports: isort](https://img.shields.io/badge/%20imports-isort-%231674b1?style=flat&labelColor=ef8336)](https://pycqa.github.io/isort/)
[![Type Checking: mypy](https://img.shields.io/badge/type_checking-mypy-blue)](http://mypy-lang.org/)
[![Jupyter](https://img.shields.io/badge/Jupyter-%23F37626.svg?style=flat&logo=Jupyter&logoColor=white)](https://jupyter.org/)
[![arXiv](https://img.shields.io/badge/arXiv-2509.03964-b31b1b.svg)](https://arxiv.org/abs/2509.03964)
[![Year](https://img.shields.io/badge/Year-2025-purple)](https://github.com/chirindaopensource/crypto_currencies_interest_rates)
[![Discipline](https://img.shields.io/badge/Discipline-Mathematical%20Finance-blue)](https://github.com/chirindaopensource/crypto_currencies_interest_rates)
[![Methodology](https://img.shields.io/badge/Methodology-Robust%20Statistics%20%26%20Derivatives%20Pricing-orange)](https://github.com/chirindaopensource/crypto_currencies_interest_rates)
[![Data Source](https://img.shields.io/badge/Data-Deribit%20%7C%20Aave-lightgrey)](https://www.deribit.com/)
[![Pandas](https://img.shields.io/badge/pandas-%23150458.svg?style=flat&logo=pandas&logoColor=white)](https://pandas.pydata.org/)
[![NumPy](https://img.shields.io/badge/numpy-%23013243.svg?style=flat&logo=numpy&logoColor=white)](https://numpy.org/)
[![Statsmodels](https://img.shields.io/badge/Statsmodels-15588D.svg?style=flat)](https://www.statsmodels.org/stable/index.html)
[![PyYAML](https://img.shields.io/badge/PyYAML-4B5F6E.svg?style=flat)](https://pyyaml.org/)

--

**Repository:** `https://github.com/chirindaopensource/crypto_currencies_interest_rates`

**Owner:** 2025 Craig Chirinda (Open Source Projects)

This repository contains an **independent**, professional-grade Python implementation of the research methodology from the 2025 paper entitled **"Cryptocurrencies and Interest Rates: Inferring Yield Curves in a Bondless Market"** by:

*   Philippe Bergault
*   Sébastien Bieber
*   Olivier Guéant
*   Wenkai Zhang

The project provides a complete, end-to-end computational framework for constructing cryptocurrency yield curves from derivatives market data. It delivers a modular, auditable, and extensible pipeline that replicates the paper's entire workflow: from rigorous data validation and cleansing, through robust outlier detection via RANSAC, to the closed-form analytical estimation of zero-coupon bond prices and their conversion to a continuous, daily time series of interest rates.

## Table of Contents

- [Introduction](#introduction)
- [Theoretical Background](#theoretical-background)
- [Features](#features)
- [Methodology Implemented](#methodology-implemented)
- [Core Components (Notebook Structure)](#core-components-notebook-structure)
- [Key Callable: run_full_analysis](#key-callable-run_full_analysis)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Input Data Structure](#input-data-structure)
- [Usage](#usage)
- [Output Structure](#output-structure)
- [Project Structure](#project-structure)
- [Customization](#customization)
- [Contributing](#contributing)
- [Recommended Extensions](#recommended-extensions)
- [License](#license)
- [Citation](#citation)
- [Acknowledgments](#acknowledgments)

## Introduction

This project provides a Python implementation of the methodologies presented in the 2025 paper "Cryptocurrencies and Interest Rates: Inferring Yield Curves in a Bondless Market." The core of this repository is the iPython Notebook `crypto_currencies_interest_rates_draft.ipynb`, which contains a comprehensive suite of functions to replicate the paper's findings, from initial data validation to the final generation and analysis of yield curve time series.

The paper addresses a fundamental challenge in decentralized finance: how to determine a term structure of interest rates for a currency that lacks a traditional, liquid bond market. This codebase operationalizes the paper's derivatives-based approach, allowing users to:
-   Rigorously validate and cleanse high-frequency options market data.
-   Apply the Random Sample Consensus (RANSAC) algorithm to robustly filter outliers from option price data based on a modified put-call parity relationship.
-   Solve a weighted least-squares problem with a closed-form analytical solution to jointly estimate the prices of synthetic zero-coupon bonds in both the cryptocurrency and a reference currency (USD).
-   Convert these bond prices into annualized, continuously compounded interest rates.
-   Aggregate and interpolate these granular estimates to construct a continuous, daily time series of the yield curve for standardized tenors.
-   Perform a full suite of robustness checks, including parameter sensitivity analysis and bootstrap confidence intervals, to validate the stability and precision of the estimates.
-   Conduct economic validation by comparing the derived rates to external benchmarks and performing formal econometric tests.

## Theoretical Background

The implemented methods are grounded in Arbitrage Pricing Theory, Financial Engineering, and Robust Statistics.

**1. Modified Put-Call Parity for Inverse Options:**
The cornerstone of the methodology is a static replication argument. A specific portfolio of an inverse call, an inverse put, and an inverse future on a cryptocurrency `C` creates a deterministic payoff in a reference currency (USD). Under the no-arbitrage principle, the cost of this portfolio today must equal the discounted value of its future payoff. This leads to the central pricing relationship, a modified form of put-call parity:
$$ C_t(K, T) - P_t(K, T) = \frac{F_t(T) - K}{S_t} \cdot ZC_t^{ref}(T) $$
where $ZC_t^{ref}(T)$ is the unknown price of a zero-coupon bond in the reference currency.

**2. Robust Regression with RANSAC:**
Market data is noisy and contains outliers. To robustly identify the underlying relationship in the data before estimation, the paper uses the Random Sample Consensus (RANSAC) algorithm. This iterative method fits a model to minimal subsets of the data to find the largest "consensus set" of inliers, effectively isolating and ignoring outliers. This is applied to a linear relationship derived from put-call parity for bid-ask prices.

**3. Joint Weighted Least-Squares Estimation:**
The prices of the crypto zero-coupon bond ($ZC_t^C(T) = \alpha_t$) and the reference currency zero-coupon bond ($ZC_t^{ref}(T) = \beta_t$) are estimated by jointly minimizing the errors in two parity relationships across all inlier strike prices for a given expiry. This is formulated as the minimization of the objective function:
$$ \arg\min_{\alpha_t, \beta_t} \sum_{i=1}^{n_T} \left(y_t^i - \left(\alpha_t - \frac{K_i}{S_t} \beta_t\right)\right)^2 + \lambda_n^T \left(\alpha_t - \frac{F_t}{S_t} \beta_t\right)^2 $$
The paper provides a closed-form analytical solution to this problem, which is implemented directly.

## Features

The provided iPython Notebook (`crypto_currencies_interest_rates_draft.ipynb`) implements the full research pipeline, including:

-   **Modular, Task-Based Architecture:** The entire pipeline is broken down into 22 distinct, modular tasks, from data validation to economic analysis.
-   **Professional-Grade Data Validation:** A comprehensive validation suite ensures all inputs (data and configurations) conform to the required schema before execution.
-   **Auditable Data Filtering:** A multi-stage filtering pipeline for maturity and liquidity, returning detailed diagnostic reports at each step.
-   **From-Scratch RANSAC Implementation:** A numerically stable, from-scratch implementation of the RANSAC algorithm for maximum control and fidelity to the paper's methodology.
-   **High-Fidelity Analytical Solver:** A direct and numerically robust implementation of the closed-form solution for the zero-coupon bond prices.
-   **Complete Term Structure Construction:** A full suite of functions for temporal aggregation, maturity interpolation, and time-series gap-filling to construct a continuous daily yield curve panel.
-   **Advanced Robustness Toolkit:**
    -   A parallelized framework for conducting **parameter sensitivity analysis** across a grid of hyperparameter values.
    -   A parallelized framework for constructing **bootstrap confidence intervals** to quantify the statistical uncertainty of the estimates.
-   **Econometric Validation Suite:** Functions to perform economic plausibility checks, compare results to external benchmarks, and conduct formal statistical tests (ADF, Cointegration).

## Methodology Implemented

The core analytical steps directly implement the methodology from the paper:

1.  **Data Validation and Filtering (Tasks 1-6):** Ingests and rigorously validates the raw data and `config.yaml` file, then filters the data based on maturity, liquidity, and the existence of complete call-put pairs.
2.  **RANSAC Outlier Rejection (Tasks 7-9):** Prepares the data for RANSAC, executes the algorithm for each timestamp-expiry group, and validates the results to find a clean set of inlier options.
3.  **Closed-Form Estimation (Tasks 10-12):** Constructs feature vectors from the inlier data and applies the analytical solution to estimate the zero-coupon bond prices (`α*`, `β*`).
4.  **Rate Conversion and Validation (Tasks 13-14):** Converts the bond prices to continuously compounded interest rates and performs final economic plausibility checks.
5.  **Time Series Construction (Tasks 15-18):** Aggregates the granular estimates into a daily panel, interpolates across maturities to standardized tenors, and fills temporal gaps to create a continuous time series.
6.  **Robustness and Economic Analysis (Tasks 20-22):** Executes the optional, advanced analyses for parameter sensitivity, bootstrap CIs, and economic validation.

## Core Components (Notebook Structure)

The `crypto_currencies_interest_rates_draft.ipynb` notebook is structured as a logical pipeline with modular orchestrator functions for each of the major tasks. All functions are self-contained, fully documented with type hints and docstrings, and designed for professional-grade execution.

## Key Callable: run_full_analysis

The central function in this project is `run_full_analysis`. It orchestrates the entire analytical workflow, providing a single entry point for running the baseline study replication and the advanced robustness checks.

```python
def run_full_analysis(
    df_master: pd.DataFrame,
    fused_input_state: Dict[str, Any],
    parameter_grid: Optional[Dict[str, List[Any]]] = None,
    bootstrap_group: Optional[Tuple] = None,
    n_bootstrap: int = 1000,
    benchmark_df: Optional[pd.DataFrame] = None,
    verbose: bool = False
) -> Dict[str, Any]:
    """
    Executes the entire end-to-end yield curve analysis.
    """
    # ... (implementation is in the notebook)
```

## Prerequisites

-   Python 3.9+
-   Core dependencies: `pandas`, `numpy`, `statsmodels`, `pyyaml`.

## Installation

1.  **Clone the repository:**
    ```sh
    git clone https://github.com/chirindaopensource/crypto_currencies_interest_rates.git
    cd crypto_currencies_interest_rates
    ```

2.  **Create and activate a virtual environment (recommended):**
    ```sh
    python -m venv venv
    source venv/bin/activate  # On Windows, use `venv\Scripts\activate`
    ```

3.  **Install Python dependencies:**
    ```sh
    pip install pandas numpy statsmodels pyyaml
    ```

## Input Data Structure

The pipeline requires two primary inputs:
1.  **`df_master`:** A `pandas.DataFrame` containing high-frequency options market data. It **must** have a `MultiIndex` with the levels `['timestamp', 'asset', 'expiry_date', 'strike_price', 'option_type']` and the required data columns.
2.  **`fused_input_state`:** A Python dictionary loaded from the `config.yaml` file, which controls all hyperparameters of the pipeline.

A mock data generation code block is provided in the main notebook to create a valid example `df_master` for testing the pipeline.

## Usage

The `crypto_currencies_interest_rates_draft.ipynb` notebook provides a complete, step-by-step guide. The core workflow is:

1.  **Prepare Inputs:** Load your `df_master` DataFrame. Ensure the `config.yaml` file is present and configured.
2.  **Execute Pipeline:** Call the grand orchestrator function.

    ```python
    # This single call runs the entire project.
    full_report = run_full_analysis(
        df_master=my_market_data_df,
        fused_input_state=my_config_dict,
        verbose=True
    )
    ```
3.  **Inspect Outputs:** Programmatically access any result from the returned dictionary. For example, to view the final yield curve time series:
    ```python
    final_yield_curves = full_report['core_pipeline_results']['yield_curves_df']
    print(final_yield_curves.head())
    ```

## Output Structure

The `run_full_analysis` function returns a single, comprehensive dictionary containing all generated artifacts, including:
-   `core_pipeline_results`: A dictionary containing the final `yield_curves_df` and a nested dictionary of diagnostics from every step of the main pipeline run.
-   `sensitivity_analysis`: (Optional) A dictionary containing summary DataFrames for rate and coverage stability.
-   `bootstrap_analysis`: (Optional) A dictionary with the point estimate and confidence intervals for the targeted group.
-   `economic_validation`: (Optional) A dictionary with the results of the theoretical, benchmark, and statistical tests.

## Project Structure

```
crypto_currencies_interest_rates/
│
├── crypto_currencies_interest_rates_draft.ipynb   # Main implementation notebook
├── config.yaml                                    # Master configuration file
├── requirements.txt                               # Python package dependencies
├── LICENSE                                        # MIT license file
└── README.md                                      # This documentation file
```

## Customization

The pipeline is highly customizable via the `config.yaml` file. Users can easily modify all hyperparameters, such as filter thresholds, RANSAC parameters, and target tenors, without altering the core Python code.

## Contributing

Contributions are welcome. Please fork the repository, create a feature branch, and submit a pull request with a clear description of your changes. Adherence to PEP 8, type hinting, and comprehensive docstrings is required.

## Recommended Extensions

Future extensions could include:

-   **Alternative Interpolation Methods:** Implementing more advanced term structure interpolation methods, such as cubic splines or the Nelson-Siegel model, as alternatives to linear interpolation.
-   **Generalization to Other Derivatives:** Extending the framework to incorporate data from other relevant derivatives, such as perpetual futures or different option structures.
-   **Automated Reporting:** Creating a function that takes the final `full_analysis_report` dictionary and generates a full PDF or HTML report summarizing the findings with plots and tables.
-   **Live Data Integration:** Building a data ingestion module to connect the pipeline to a live market data feed from an exchange API for real-time yield curve monitoring.

## License

This project is licensed under the MIT License. See the `LICENSE` file for details.

## Citation

If you use this code or the methodology in your research, please cite the original paper:

```bibtex
@article{bergault2025cryptocurrencies,
  title={{Cryptocurrencies and Interest Rates: Inferring Yield Curves in a Bondless Market}},
  author={Bergault, Philippe and Bieber, S{\'e}bastien and Gu{\'e}ant, Olivier and Zhang, Wenkai},
  journal={arXiv preprint arXiv:2509.03964},
  year={2025}
}
```

For the implementation itself, you may cite this repository:
```
Chirinda, C. (2025). A Python Implementation for Inferring Cryptocurrency Yield Curves.
GitHub repository: https://github.com/chirindaopensource/crypto_currencies_interest_rates
```

## Acknowledgments

-   Credit to **Philippe Bergault, Sébastien Bieber, Olivier Guéant, and Wenkai Zhang** for their foundational research, which forms the entire basis for this computational replication.
-   This project is built upon the exceptional tools provided by the open-source community. Sincere thanks to the developers of the scientific Python ecosystem, including **Pandas, NumPy, and Statsmodels**, whose work makes complex computational finance accessible and robust.

--

*This README was generated based on the structure and content of `crypto_currencies_interest_rates_draft.ipynb` and follows best practices for research software documentation.*
