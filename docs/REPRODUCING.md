# Reproduction guide

[Back to the project overview](../README.md)

This guide covers the code in [AnsariNavid/mixed-integer-ink-selection](https://github.com/AnsariNavid/mixed-integer-ink-selection), the complete data archive, and the corresponding [paper](https://aidam.mpi-inf.mpg.de/data/pdfs/miisfsr.pdf) and [supplement](https://aidam.mpi-inf.mpg.de/data/pdfs/miisfsr_SM.pdf). The original release is preserved in the [GitLab repository](https://gitlab.mpi-klsb.mpg.de/nansari/mixed-integer-ink-selection-code-and-dataset).

## 1. Extract and organize the files

The GitHub repository contains the MATLAB and Python code, five pretrained checkpoints, and the small ink-selection inputs. The large data files are hosted separately on Google Drive because of their size. Download the linked archive in addition to the GitHub checkout to obtain the neural training arrays, full painting spectra, paint-measurement arrays, and calibration channel patterns. GitHub's **Download ZIP** contains only the files hosted in this repository.

The [Google Drive archive](https://drive.google.com/file/d/1TwPk96DutU-MFO03uSR_5ThjqFBJIOHH/view?usp=sharing) is named `mixed-integer-ink-selection-code-and-dataset-master.zip`. It contains one top-level folder, `mixed-integer-ink-selection-code-and-dataset-master/`, with 110 files from the original release.

### Prepare a GitHub checkout

Clone this repository, or download and extract its GitHub ZIP. The directory containing `README.md`, `Dataset/`, `Ink_selection/`, and `Spectral seperation neural networks/` is the project root.

```bash
git clone https://github.com/AnsariNavid/mixed-integer-ink-selection.git
cd mixed-integer-ink-selection
```

Download the Google Drive archive, then extract it into a separate temporary folder. Merge the extracted `Dataset/` and `Spectral seperation neural networks/` folders into the matching folders in the checkout, copying missing files and retaining existing files. Do not nest the entire archive folder inside the checkout or flatten the data directories.

For example, on Linux or macOS, run the following **from the GitHub project root**. Replace the archive path with its actual location:

```bash
unzip "/path/to/mixed-integer-ink-selection-code-and-dataset-master.zip" -d ../ink-selection-data
```

The following Python command merges the two folders without replacing existing files. Run it from the same project root after extraction:

```bash
python - <<'PY'
from pathlib import Path
import shutil

source = Path('../ink-selection-data/mixed-integer-ink-selection-code-and-dataset-master')
target = Path.cwd()
assert (source / 'Ink_selection').is_dir(), 'Check the archive extraction path.'
assert (target / 'Ink_selection').is_dir(), 'Run this from the GitHub project root.'

copied = 0
for folder in ('Dataset', 'Spectral seperation neural networks'):
    for src in (source / folder).rglob('*'):
        if src.is_file():
            dst = target / src.relative_to(source)
            if not dst.exists():
                dst.parent.mkdir(parents=True, exist_ok=True)
                shutil.copy2(src, dst)
                copied += 1
print(f'Copied {copied} missing files.')
PY
```

On Windows, use a ZIP extractor and merge the same folders, choosing **skip existing files** when prompted. The reviewed GitHub revision needs 58 additional files. A later revision or an already prepared checkout may need fewer.

The compressed archive is 6,306,246,522 bytes and the total uncompressed file contents are 6,459,992,450 bytes. Keeping both requires about 12.77 GB before filesystem overhead and generated outputs. The temporary extraction adds another copy of the restored files while merging; remove that temporary copy after confirming the files are in the checkout. A complete standalone extraction can also run the examples, but use the GitHub checkout to retain the current documentation and any subsequent code updates.

### Check file locations

| Destination relative to the project root | Required contents |
| --- | --- |
| `Ink_selection/Test data/` | `ink_names.mat`, `transmitance_completelibrary.mat`, and `coresettrans_*.mat`. |
| `Spectral seperation neural networks/` | `dataset_approx_spec.mat`, `dataset_halftone.mat`, `flower_painting_approx_spec.mat`, model-definition files, and the required `.ckpt` checkpoints. |
| `Spectral seperation neural networks/` | Canon example inputs: `canondataset_approx_spec.mat` and `halftone_canon.mat`. |
| `Dataset/Water color paint/` | `reflectance_watercolor.mat` and the reference image. |
| `Dataset/Oil paint/` | `Reflectance_oilcolor.mat` and the reference image. |
| `Dataset/Spectral paintings/` | Painting arrays and `Image dimensions.txt`. |
| `Dataset/Dataset_ 300samples/` | Reference reflectances, transmittances, RGB arrays, and `halftoning pattern/`. |
| `Dataset/Dataset_ 33000samples/` | `halftone_code.mat`, `all.png`, and channel-pattern folders `p1/` through `p4/`. |

Spaces, capitalization, and original spellings are significant. In particular, retain `Spectral seperation neural networks`, `Dataset_ 300samples`, and `Dataset_ 33000samples` exactly.

For pretrained Flower inference, add `flower_painting_approx_spec.mat` from the Google Drive archive. Retraining also needs `dataset_approx_spec.mat` and `dataset_halftone.mat`. Put these files directly inside `Spectral seperation neural networks/`, beside the Python scripts and checkpoints. The MATLAB demo's `Ink_selection/Test data/` inputs are already present in the GitHub checkout.

### Data reference

Spectra use 31 wavelength bands, `400:10:700` nm. Keep spectral rows aligned with the corresponding ink-coverage rows. The arrays use a fractional reflectance/transmittance scale rather than percentages. Some reconstructed reflectances fall outside the nominal 0–1 range; document any clipping instead of silently modifying the supplied data.

| File | MATLAB variable | Shape |
| --- | --- | --- |
| `Ink_selection/Test data/transmitance_completelibrary.mat` | `transmitance_completelibrary` | `44 × 31` |
| `Ink_selection/Test data/ink_names.mat` | `names` | `1 × 44` cell array |
| `Ink_selection/Test data/coresettrans_cat.mat` | `coresettrans_cat` | `200 × 31` |
| `Spectral seperation neural networks/dataset_approx_spec.mat` | `dataset_approx_spec` | `33,945 × 31` |
| `Spectral seperation neural networks/dataset_halftone.mat` | `dataset_halftone` | `33,945 × 8` |
| `Spectral seperation neural networks/flower_painting_approx_spec.mat` | `flower_painting_approx_spec` | `4,612,992 × 31` |
| `Spectral seperation neural networks/canondataset_approx_spec.mat` | `canondataset_approx_spec` | `9,912 × 31` |
| `Spectral seperation neural networks/halftone_canon.mat` | `halftone_reshape` | `9,912 × 4` |
| `Dataset/Water color paint/reflectance_watercolor.mat` | `spec` | `423 × 31` |
| `Dataset/Oil paint/Reflectance_oilcolor.mat` | `spec` | `289 × 31` |

Several painting variables do not match their filenames. Under `Dataset/Spectral paintings/`, use:

| File | MATLAB variable | Shape |
| --- | --- | --- |
| `cat_painting_approx_spec.mat` | `cat_painting_approx_spec` | `4,248,090 × 31` |
| `couple_painting_approx_spec.mat` | `men_painting_approx_spec` | `6,095,040 × 31` |
| `lady_painting_approx_spec.mat` | `women_painting_approx_spec` | `5,304,348 × 31` |
| `painting_duetone_approx_spec.mat` | `painting_duetone_approx_spec` | `3,044,615 × 31` |
| `sunset_approx_spec_LDR.mat` | `painting9_approx_spec_LDR` | `3,114,990 × 31` |

Use `whos('-file', filename)` in MATLAB or `scipy.io.whosmat(filename)` in Python when checking a file. For reshaping images, consult `Image dimensions.txt` and preserve the exporter's MATLAB pixel-order convention.

The neural training spectra were reconstructed from camera measurements of printed patches. They should not be described as 33,945 independent spectrophotometer measurements; see Section 6.2 of the paper.

## 2. Run ink selection

Install MATLAB, [YALMIP](https://yalmip.github.io/tutorial/installation/), and [Gurobi's MATLAB interface](https://docs.gurobi.com/projects/optimizer/en/current/reference/matlab/setup.html). Configure the interface with `gurobi_setup` from your Gurobi installation's `matlab/` directory. The default demo has 15,044 decision variables before presolve, so the Gurobi license must support that model size.

Check interface visibility in MATLAB:

```matlab
which sdpvar
which binvar
which optimize
which gurobi
```

These checks do not validate the solver license. The paper used Gurobi 9.0.1; record the MATLAB, YALMIP, and solver versions used for a new experiment.

The entry point is [`absorbtance_optimization_demo.m`](../Ink_selection/absorbtance_optimization_demo.m). Its load statements use Windows separators. For portable paths, replace the three loading statements with:

```matlab
load(fullfile('Test data', 'ink_names.mat'))
load(fullfile('Test data', 'transmitance_completelibrary.mat'))
coresettrans = struct2cell(load(fullfile('Test data', 'coresettrans_cat.mat')));
```

Keep the following line, `coresettrans = coresettrans{1};`. From the project root, run:

```matlab
cd('Ink_selection')
absorbtance_optimization_demo
```

The demo uses 44 library entries, the 200-spectrum Cat coreset, a maximum of eight selected inks, and a maximum thickness of four per candidate ink and sample. It minimizes the sum of absolute absorbance errors. The transform is `A = -log(T)`, so input transmittances must be strictly positive.

On `sol.problem == 0`, the script assigns `selected_inks` and `thickness` and displays the objective and selected names. It does not save a results file automatically. Save those outputs and the solver status if you need to compare runs.

The thickness variables belong to the absorbance mixing model. They are not neural area-coverages or printer-ready bitmaps. To change the target, select another coreset file. Preserve filenames such as `coresettrans_sunet.mat`; some internal coreset keys also differ from the filenames, which the script's `struct2cell` loading handles.

## 3. Run neural spectral separation

Install Python 3, NumPy, SciPy, and a suitable [PyTorch build](https://pytorch.org/get-started/locally/). Run all Python commands in `Spectral seperation neural networks/`. The release has no pinned environment file, so record the package versions used.

The bundled checkpoints contain tensors saved on `cuda:0`. The supplied loaders do not use `map_location`, and `generate_halftone.py` explicitly creates CUDA input tensors. Use a CUDA-enabled environment for the unmodified inference script. A CPU port needs consistent device placement and checkpoint loading changes; the printed device fallback alone is insufficient.

### Use the supplied Flower checkpoints

Required files in the neural directory:

- `Forward_model.ckpt`
- `model_backward_flower.ckpt`
- `flower_painting_approx_spec.mat`
- `layer_config_forward.py` and `layer_config_backward.py`

From the project root:

```bash
cd "Spectral seperation neural networks"
python -c "from pathlib import Path; Path('painting_halftone').mkdir(exist_ok=True)"
python generate_halftone.py
```

For the supplied Flower array, the code writes `painting0.npy` through `painting9.npy`: nine arrays of `500,000 × 8` and a final array of `112,992 × 8`. These are continuous ink-coverages. The printed error compares the target with the forward model's prediction, not with a newly measured print.

### Retrain and adapt

Preserve the supplied checkpoints before training because the scripts overwrite their existing names. From the neural directory, run:

```bash
python HalftoneToSpec.py
python "Forward+Backward.py"
python "Adaptive learning.py"
python -c "from pathlib import Path; Path('painting_halftone').mkdir(exist_ok=True)"
python generate_halftone.py
```

| Stage | Inputs | Saved checkpoint |
| --- | --- | --- |
| Forward training | `dataset_approx_spec.mat` and `dataset_halftone.mat` | `Forward_model.ckpt` |
| General inverse training | Training spectra and the forward checkpoint | `model_backward.ckpt` |
| Flower adaptation | Flower spectra and the forward/general inverse checkpoints | `model_backward_flower.ckpt` |

The forward model is frozen during both inverse training stages. The inverse loss combines spectral reconstruction error with an ink-coverage penalty.

For another painting, change the MAT filename and key in adaptation and inference. Replace the hard-coded sample range `4612992` in `Adaptive learning.py` with the actual row count, then align the checkpoint name, output location, chunk count, and image dimensions.

For another ink set, prepare paired calibration spectra and coverages with the correct channel order and train compatible models. The eight-ink Epson network is not automatically reconfigured by the MATLAB optimizer's selection.

## 4. Export coverage maps

Open [`coverage2halftone.m`](../Spectral%20seperation%20neural%20networks/Prepare%20the%20area%20coverage%20to%20print/coverage2halftone.m) under `Spectral seperation neural networks/Prepare the area coverage to print/`.

The released exporter expects `image/test_im0.npy` through `test_im9.npy`, whereas Python writes `painting_halftone/painting0.npy` through `painting9.npy`. Replace the initial loading loop with:

```matlab
addpath('npy_read')
completehalftone = [];
input_dir = fullfile('..', 'painting_halftone');
for i = 1:10
    file_name = sprintf('painting%d.npy', i-1);
    approx_halftone = readNPY(fullfile(input_dir, file_name));
    completehalftone = [completehalftone; approx_halftone];
end
```

Keep the remaining reshape and export code. Run it with the exporter's directory as MATLAB's current folder. For Flower, it reshapes to `[2344 1968 8]` and then permutes to `[1968 2344 8]`. Reassemble chunks in numeric order and preserve this ordering convention when porting to Python.

The eight channels are **Yellow, LightBlack, LightMagenta, LightCyan, LightLightBlack, Magenta, Cyan, Black**, in that order. The exporter clips coverages above one and writes inverted channel TIFFs. Review clipping and channel mapping when preparing a different printing experiment.

The adjacent `convertnpy2mat.m` converts a single NPY file to a MAT file containing `matfile`. It is optional because the coverage exporter reads NPY directly.

## 5. Reproduction notes

### Match the released settings to the intended experiment

| Item | Released example | Paper |
| --- | --- | --- |
| Ink-library size | 44 entries in the supplied array | 43-ink physical library |
| Forward hidden widths | `150, 150, 150` | `300, 300, 200, 100` in Section 6.4 |
| Inverse hidden widths | `300, 300, 200, 100` | `160, 160, 160, 160` in Section 6.4 |
| Forward epochs / initial learning rate | 19 / `0.001` | 20 / `0.005` |
| Inverse epochs / initial learning rate | 20 / `0.005` | 20 / `0.005` |
| Split / actual batch size / learning-rate decay | 80:20 / 31 / `0.95` | 80:20 / 31 / `0.95` |

The reason for the extra library entry has not been established. Do not remove an entry without checking its identity. The inverse constructor uses only the first four hidden widths even though the calling code lists five. Checkpoint shapes and model definitions must agree.

The paper reports a `10^-4` duality-gap target; the demo does not set a tolerance explicitly. Record the configured relative and absolute gap settings and the achieved solver status. Solver optimality concerns the formulated absorbance problem, not an unconditional guarantee of print accuracy.

### Details that affect execution or evaluation

| Source behavior | Implication |
| --- | --- |
| Training validation arrays are initialized with one zero-error row per batch before real samples are appended. | The displayed mean is too low. Recompute metrics over real samples only. For complete 31-sample batches, the legacy mean is `31/32` of the unpadded value. |
| Inference averages chunk errors without weighting by pixel count. | Weight chunk means by the number of pixels when computing a full-image mean. |
| Inference iterates to `floor(N/500000) + 1`. | If `N` is exactly divisible by 500,000, avoid the resulting empty final chunk. |
| Forward trainers reshape batches to exactly 31 rows. | Use a batch-size-independent reshape for datasets with partial final batches. |
| `Canon_Net.py` uses `.mat.mat` filenames and mismatched MATLAB keys. | Use `canondataset_approx_spec.mat` / `canondataset_approx_spec` and `halftone_canon.mat` / `halftone_reshape`. The 9,912-row dataset also requires handling partial batches. |
| `Extract_model_w_b.py` misspells an import as `layer_confog_forward` and supplies an incompatible two-width configuration. | Align the import, constructor, and checkpoint before using that optional utility. |
| Compact Epson and Canon forward models differ from the main forward architecture. | Their checkpoints are not drop-in replacements for the main inverse workflow. |

Initialization, splitting, and painting subsampling are not seeded in the released scripts. Record seeds and package versions when comparing trained models. These observations concern the released code; they do not establish how the paper's reported tables were computed.

### Scope of reproduction

The combined GitHub checkout and data archive include computational examples, calibration data, painting spectra, and checkpoints. The original release's file inventory contains no standalone R2T training/inference, RGB-to-spectrum regression, coreset-generation, genetic-algorithm benchmark, or custom printer-driver implementation. The Google Drive archive matches the original GitLab release; the GitHub repository contains the code and a subset of those assets.

Figure 8's large-library experiment uses Munsell spectra as candidate surrogates. Section 6.8 evaluates adaptive separation through forward-model simulations. Physical reproduction additionally requires the matching inks, substrate, calibration, halftoning, and printing hardware described in the paper. The TIFF-export utility alone does not supply that complete physical pipeline.

For a reproducible comparison, record the target paper figure or table, data and ink set, code revision, model architecture and checkpoint, environment, random seeds, solver settings, and metric definition. Distinguish model-predicted errors from errors measured on physical prints.

## Verification record

The source, paper, supplement, and 24 MATLAB data-file headers were reviewed during preparation of this documentation. The Google Drive ZIP directory's commit comment and 110-file inventory match original GitLab commit `03509a0e5b04e6c2bd914d481723cbb80aa4db38`. Selected source, checkpoint, and small data files were extracted from the archive and checked against its CRCs and GitLab blob hashes.

The GitHub repository was checked at commit `1c3bbd8033f2a9e1c47e43bdc3c2b9047bc78edd`: 52 files match the original release by Git blob hash, `.gitattributes` is additional, and 58 original data and metadata files are absent. The missing files belong to the two folders covered by the preparation instructions above. This comparison describes the repository before this documentation update.

The multi-gigabyte painting arrays were not downloaded again for this revision. Optimization, neural training, and physical printing were not rerun for this documentation update.
