# IVIM-DKI Analysis app (IDTV-AD)

Desktop application for hybrid IVIM-DKI parameter mapping with the IDTV-AD method. It loads a 4D diffusion-weighted
NIfTI (x, y, z, b-value), fits the diffusion coefficient D, pseudo-diffusion coefficient D*, perfusion fraction f and
kurtosis k in every voxel, and saves the parameter maps.

The fit is the published IDTV-AD method: a bounded, damped Gauss-Newton solver with a backtracking line search, a free
voxel-wise S0 estimated in closed form, and parameter-specific total-variation regularization on the logarithm of each
parameter (computed in voxel units) inside a fitting mask.

## Install (Windows, no Python needed)

1. Go to the **Releases** page of this repository (right-hand side of the GitHub page) and download
   `IVIM-DKI-Windows-x64.zip` (about 165 MB).
2. Right-click the downloaded file → **Extract All…** and extract it to a folder you can write to, for example
   `Documents` or `Desktop` (not `Program Files`). Do not run the app from inside the zip.
3. Open the extracted `IVIM-DKI` folder and double-click **`IVIM-DKI.exe`**. The first start takes a few seconds.
4. The first time, Windows may show *"Windows protected your PC"* because the app is not code-signed: click
   **More info → Run anyway**.

Results are written to an `Analysis_Results` folder inside the `IVIM-DKI` folder. To try it out, load one of the files in
`IVIM-DKI\test_data` (see *Test data* below). The app runs on the CPU: a 100×100×5 phantom takes about 35 s, a patient
volume several minutes.

To uninstall, delete the `IVIM-DKI` folder.

## Running from source (Windows, macOS or Linux, Python 3.10+)

```
pip install -r requirements.txt
python src/app.py
```

## Using it

1. **Input tab** → *Select IVIM-DKI data* and choose a 4D NIfTI file. If a `<name>.bval` or `<name>.txt` file with the
   b-values sits next to it, the b-value field is filled in automatically; otherwise type them (first b-value must be 0).
2. Choose the **tissue preset** (sets the initial values) and the **bounds preset**:
   *Clinical* uses a D* floor of 0.008 mm²/s, *Simulation* a floor of 0.0001 mm²/s.
3. Choose the **fitting mask**: *Automatic body mask* for patient data, *Whole volume* for simulations, or a mask file
   on the same grid. Optionally restrict the fit to some slices (e.g. `5:14`).
4. The TV weights (0.01, 0.0035, 0.01, 0.0017 for D, D*, f, k), 400 iterations, voxel-unit TV and single precision are
   the published settings. Press **Run**; the fit runs in the background and can be stopped.
5. Results are saved automatically to `Analysis_Results/<input>_IDTV-AD_.../`: one NIfTI per parameter (same geometry
   as the input, 0 outside the fitting mask), the fitting mask, a composite PNG of the middle slice and
   `analysis_info.txt` with all settings. The **Output** tab shows the maps; the **Statistics** tab computes ROI
   statistics from a parameter map and an ROI mask.

## Test data

`test_data/` contains the four digital phantoms of the paper at SNR 60 (Simulation-I to IV: D, D*, f or k varied in five
concentric rings, the other three parameters fixed): the noisy 4D input (`*_SNR60.nii.gz`, S0 = 1000, Gaussian noise
σ = 16.66, 13 b-values), the b-values (`*.bval`) and the ground-truth maps (`*_ground_truth.nii.gz`, volumes D, D*, f, k).
Load one with the *Simulation* presets and *Whole volume* mask. A full phantom takes about 40 s on a typical desktop CPU.

`IVIM-DKI.exe --selftest` (or `python src/app.py --selftest`) fits all four phantoms without the window and writes the
error of the varied parameter to `selftest_result.txt`. Expected RMSE at SNR 60: D ≈ 2.7%, D* ≈ 6.6%, f ≈ 10.5%,
k ≈ 3.4%.

## Building the Windows app

```
powershell -ExecutionPolicy Bypass -File build.ps1
```

creates `dist/IVIM-DKI/` (PyInstaller, one folder) with the executable, the test data and this README.

## Files

| File | Content |
|---|---|
| `src/app.py` | Graphical interface (Tkinter) |
| `src/engine.py` | IDTV-AD fit with the published settings, automatic body mask |
| `src/idtv_revised_pp.py`, `src/fixed_checkpoint_backend_pp.py` | IDTV-AD optimizer (JAX) |
| `src/otsu_mask.py`, `src/otsu_log.py`, `src/body_envelope_mask.py`, `src/mask_io.py` | Body-mask construction |
| `src/selftest.py` | Headless accuracy check on the test data |

## License

MIT (see `LICENSE`). Developed by Hisham Hanif and Himansu Maurya.
