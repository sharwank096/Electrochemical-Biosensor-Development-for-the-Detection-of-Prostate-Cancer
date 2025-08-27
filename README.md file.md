**Galectin‑3 Electrochemical Data Analysis**

**What this project is?**
I analysed electrochemical data from three screen-printed electrodes (SPE-12, SPE-14, SPE-18) using purely sinusoidal voltammetry. After basic cleaning of each file (convert to numeric, remove NaNs, keep a steady 2–12 s window, filter obvious spikes with IQR), I saw that time-domain plots varied a lot between electrodes, so they weren’t reliable for calibration. 

I transformed the current-time signals with FFT to get amplitudes at the drive frequency around 9.5 Hz and its harmonics such as 19.1 and 28.6 Hz, and used these as a feature matrix. Then I applied PCA to understand patterns: unscaled features mainly showed electrode differences, while scaled features also revealed clear concentration trends within each electrode. Simple linear fits worked inside a single electrode but did not transfer reliably across different electrodes.

---

**Data I worked with**
- Source folders in Google Drive under Data prostate cancer 2
- Each electrode has its own folder: SPE 12, SPE 14, and SPE 18
- For each concentration code there are two files per run:
  - PSV_XX_cv_current (current vs time)
  - PSV_XX_cv_voltage (potential vs time)
  - XX = two-digit concentration code
- Concentration codes mapped to values in µg/mL:
  - 00 → 0, 01 → 0.001, 02 → 0.01, 03 → 0.1, 04 → 1, 05 → 10

I loaded these into a nested dictionary shaped like filtered_data[electrode][concentration]['current'/'voltage'] so I could access any run cleanly.

---

**What I did and why?**

1) Pre‑processing (make signals reliable)
- Converted all columns to numeric and removed missing values.
- Kept a 2–12 s slice from every trace to avoid start‑up transients and end effects.
- Removed obvious spikes using the IQR rule so only realistic values remain.


2) Time‑domain checks (quality control)
- Plotted current vs time and current vs potential for sanity checks.
- I noticed big electrode‑to‑electrode differences and strong capacitive background.
- I did not try to build a calibration from these plots.

Why: These views confirm the data are sensible, but they are not stable enough to estimate concentration across electrodes.

3) FFT feature extraction (the core signal information)
- Used FFT to move to the frequency domain and read amplitudes at:
  - Fundamental ≈9.5 Hz
  - Harmonics ≈19.1 Hz, ≈28.6 Hz, ≈38.1 Hz, ≈47.7 Hz
- Where needed, I used peak finding to lock onto the exact bins.
- Built a feature matrix: each row is a run; each column is a harmonic amplitude.

Why: Harmonic amplitudes are more reproducible than raw time‑domain shapes and are the natural language for PSV.

4) PCA on FFT features (see structure clearly)
- Ran PCA twice:
  - One Unscaled features: PC1 is dominated by low‑frequency amplitude and separates electrodes strongly.
  - Two Z‑scored features: variance spreads across harmonics and concentration gradients become visible within each electrode.
- I also plotted a heatmap of the FFT feature matrix to see blocks by electrode and gradients by concentration.

Why: PCA reduces complexity and shows the big picture

5) Simple regression
- Fitted linear models between selected harmonic amplitudes and concentration  single electrodes.
- Reported R² but treated results as exploratory.
- Did not pool electrodes for one calibration line because baselines differ a lot.

Why: I wanted to know if a single harmonic can estimate concentration locally, but I avoided making claims that won’t transfer to another electrode.

---

**What came out of the analysis?**

- Time‑domain plots are useful for checks but not for calibration.
- FFT harmonics (especially ≈9.5 Hz and ≈19.1 Hz) are stable and reproducible descriptors.
- PCA shows strong electrode clustering. After scaling, concentration trends appear inside each electrode.
- Regression works inside one electrode, but fails across electrodes because each device has its own baseline and scale.
- Overall, frequency‑domain analysis explains the data better than raw plots, but electrode variability is the main limitation for universal calibration.

---

**How to run this (Colab)?**
1. Open the notebook Electrochemical_Data.ipynb in Google Colab.
2. Mount Google Drive and set the base path to your data folder.
3. Run the preprocessing cells to build filtered_data.
4. Run the FFT cells to create the feature matrix.
5. Run the PCA and plotting cells to reproduce the figures.

---

File map in this project
- Electrochemical_Data.ipynb – full analysis: cleaning, FFT, PCA, regression, and plotting.

---

Notes and decisions I took
- I fixed the analysis window to 2–12 s for all runs for fairness.
- I kept per‑electrode regressions to avoid over‑claiming a global calibration.

---

**conclusion**
I analysed PSV datasets from SPE-12, SPE-14, and SPE-18 across coded concentrations 00–05 (0, 0.001, 0.01, 0.1, 1, 10 µg/mL) using a consistent 2–12 s window after basic cleaning (numeric checks, remove NaNs, IQR outliers). Time-domain plots varied a lot between electrodes, so I moved to the frequency domain. Using FFT, the harmonic amplitudes (especially around ~9.5 Hz and ~19.1 Hz) gave me stable, comparable descriptors.

 PCA on these harmonic features made the structure clear: strong electrode differences and clear concentration steps inside each electrode. Simple linear fits were meaningful only within a single electrode, so one global calibration across electrodes is not reliable. My takeaway is to treat concentration estimation per electrode and strengthen it with replicate runs and basic metadata when available.
