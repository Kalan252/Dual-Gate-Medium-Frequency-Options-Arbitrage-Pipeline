\# High-Frequency Options Arbitrage Pipeline with Roll Microstructure \& Intraday HAR-RV Forecasting



A high-performance quantitative backtesting and execution engine designed to isolate intraday options pricing anomalies on SPY. The framework minimises computational overhead by using a dual-gate architecture that dynamically filters the options space using underlying asset microstructure boundaries before running a Numba-JIT accelerated Implicit Finite Difference Method (FDM) pricing kernel.



\## System Architecture \& Dual-Gate Design



To process massive high-frequency options data without experiencing kernel memory exhaustion, the pipeline implements a strict "Filter Before Price" engineering constraint.



\### Gate 1: Macro Stock Structure Boundary (Microstructure Bollinger Bands)

The system extracts the fundamental (clean) asset volatility directly from trade sequences by filtering out market microstructure noise (bid-ask bounce) using rolling return autocovariances. Bollinger Bands are calculated using this clean, annualised underlying volatility framework. A data chunk's rows are immediately dropped if the underlying asset price resides \*inside\* the bands.



\### Gate 2: Option Arbitrage Verification

For timestamps where a Bollinger Band breach occurs, the script extracts the relevant contracts and passes their parameters into a custom tridiagonal matrix solver accelerated via Numba JIT. If the FDM calculated fair value exceeds the executable market ask price, a buy signal is triggered and recorded to the trading ledger alongside the captured mathematical edge.



\###  Automated Visual Verification Panel

When the pipeline executes successfully, it generates and exports high-fidelity pipeline diagnostics:



!\[Filter Gate Verification Panels](gate\_1\_and\_2\_verification.png)



\---



\## Volatility Surface Forecasting



The framework incorporates an Intraday HAR-RV (Heterogeneous Autoregressive model of Realized Volatility) engine to construct volatility cascades across 5-minute, 1-hour, and 1-day horizons to project future volatility windows.



!\[HAR-RV Predictive Analysis](har\_rv\_predictive\_analysis.png)



\---

###  V2 Optimisation: Cold-Start Structural Gate Safeguard

In high-frequency volatility scanning, raw data processing at market open introduces transient anomalies due to unpopulated rolling windows. In V1, the rolling microstructure window lacked historical context during the initial morning ticks, causing the upper and lower Bollinger Bands to collapse onto a single identity line (BB_{upper} == BB_{lower}). This artificial compression triggered massive systemic false-positives, classifying standard market ticks as structural anomalies and bottlenecking the downstream finite difference method (FDM) pricing matrix.

**V2 Multi-Gate Hardening Upgrades**
* **Asymmetric Warm-Up Mask:** Implemented a vectorised state validator (`warmed_up_mask = df_m['BB_upper'] != df_m['BB_lower']`) to isolate and discard unpopulated cold-start lookback windows.
* **Filter Gate 1 Pre-Conditioning:** Shifted microstructural boundary constraints upstream, blocking raw data flows from reaching the computationally expensive tridiagonal bulk pricer until statistical variance stabilises.
* **Memory Asset Footprint Reduction:** Integrated aggressive garbage collection (`gc.collect()`) handles alongside deterministic Matplotlib caching (`plt.close('all')`) to handle massive parquet chunk streaming without memory leakage or thread locks.

\##  Project Structure



```text

├── HFT\_project.ipynb     # Interactive core pipeline (Data ingestion, HAR-RV, FDM engine)

├── .gitignore            # Protects remote repo from massive Parquet data files

├── README.md             # Production system documentation \& architecture layout

├── gate\_1\_and\_2\_verification.png  # Auto-generated verification graphics

└── har\_rv\_predictive\_analysis.png # Auto-generated forecasting diagnostics

