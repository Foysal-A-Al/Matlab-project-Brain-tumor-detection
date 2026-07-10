# Brain MRI Tumor Segmentation and Classification

A MATLAB-based brain MRI analysis project for **tumor segmentation, handcrafted feature extraction, dimensionality reduction, and SVM classification**.

The workflow combines:

- MRI image preprocessing
- Otsu thresholding and binary segmentation
- color-space-based clustering
- multi-level Discrete Wavelet Transform
- Principal Component Analysis
- texture and statistical feature extraction
- Support Vector Machine classification
- MATLAB GUIDE graphical interface
- repeated holdout and cross-validation experiments

The repository is intended as an educational and research prototype for exploring classical machine-learning methods in brain MRI analysis.

> **Important:** This project is not a medical device and must not be used for clinical diagnosis or treatment decisions.

---

## Project Overview

The system processes a brain MRI image through a classical image-analysis pipeline:

```text
MRI image
   ↓
Resize and grayscale conversion
   ↓
Thresholding / segmentation
   ↓
Wavelet decomposition
   ↓
PCA transformation
   ↓
Texture and statistical feature extraction
   ↓
SVM classification
   ↓
Benign / malignant prediction
```

The project includes both:

1. a command-line MATLAB script for experimentation and model evaluation
2. a MATLAB GUIDE interface for image selection, segmentation, feature display, classification, and kernel comparison

---

## Main Capabilities

- Load common MRI image formats
- Resize images to a fixed `200 × 200` resolution
- Convert RGB images to grayscale
- Apply Otsu thresholding
- Perform binary tumor-region segmentation
- Apply K-means-based color clustering
- Extract three levels of 2D wavelet coefficients using `db4`
- Reduce transformed features using PCA
- Compute GLCM texture descriptors
- Compute statistical image descriptors
- Train and evaluate SVM classifiers
- Compare linear, RBF, polynomial, and quadratic kernels
- Run repeated holdout validation
- Run cross-validation experiments
- Display classification results through a MATLAB GUI

---

## Feature Extraction Pipeline

The project extracts a 13-dimensional handcrafted feature vector:

| Feature | Description |
|---|---|
| Contrast | Local intensity variation from the GLCM |
| Correlation | Linear dependency between neighbouring intensities |
| Energy | Uniformity of the GLCM |
| Homogeneity | Closeness of GLCM elements to the diagonal |
| Mean | Average PCA-transformed intensity |
| Standard deviation | Intensity dispersion |
| Entropy | Information complexity |
| RMS | Root-mean-square intensity |
| Variance | Average feature variance |
| Smoothness | Intensity smoothness estimate |
| Kurtosis | Distribution tail weight |
| Skewness | Distribution asymmetry |
| IDM | Inverse Difference Moment |

The final feature vector has the form:

```matlab
feat = [
    Contrast,
    Correlation,
    Energy,
    Homogeneity,
    Mean,
    Standard_Deviation,
    Entropy,
    RMS,
    Variance,
    Smoothness,
    Kurtosis,
    Skewness,
    IDM
];
```

---

## Wavelet Decomposition

The segmented image is decomposed over three levels using the Daubechies-4 wavelet:

```matlab
[cA1,cH1,cV1,cD1] = dwt2(signal1,'db4');
[cA2,cH2,cV2,cD2] = dwt2(cA1,'db4');
[cA3,cH3,cV3,cD3] = dwt2(cA2,'db4');
```

The third-level approximation and detail coefficients are concatenated:

```matlab
DWT_feat = [cA3,cH3,cV3,cD3];
```

PCA is then applied:

```matlab
G = pca(DWT_feat);
```

The resulting matrix is used for GLCM and statistical feature extraction.

---

## Classification

The project trains SVM models using features stored in `Trainset.mat`.

Expected variables:

```matlab
meas
label
```

Where:

- `meas` contains the training feature matrix
- `label` contains the corresponding class labels

The main classifier uses a linear SVM:

```matlab
svmStruct = svmtrain(meas, label, ...
    'kernel_function', 'linear');

species = svmclassify( ...
    svmStruct, ...
    feat, ...
    'showplot', false);
```

The expected prediction classes are:

```text
BENIGN
MALIGNANT
```

---

## Supported SVM Kernels

The code evaluates the following kernels:

| Kernel | MATLAB option |
|---|---|
| Linear | `'kernel_function', 'linear'` |
| Radial Basis Function | `'kernel_function', 'rbf'` |
| Polynomial | `'Kernel_Function', 'polynomial'` |
| Quadratic | `'Kernel_Function', 'quadratic'` |

Repeated holdout experiments estimate performance for each kernel.

---

## Repository Structure

A recommended repository layout is:

```text
Brain-MRI-Classification/
├── BrainMRI_GUI.m
├── BrainMRI_GUI.fig
├── BrainMRI_Classification.m
├── Trainset.mat
├── Normalized_Features.mat
├── sample_images/
│   ├── benign/
│   └── malignant/
├── screenshots/
│   ├── main_gui.png
│   ├── segmentation.png
│   └── classification_result.png
├── README.md
└── LICENSE
```

### Required project files

| File | Purpose |
|---|---|
| `BrainMRI_GUI.m` | MATLAB GUIDE callback logic |
| `BrainMRI_GUI.fig` | GUI layout created with GUIDE |
| `BrainMRI_Classification.m` | Main processing and evaluation script |
| `Trainset.mat` | Training features and labels |
| `Normalized_Features.mat` | Normalized features for validation experiments |

The GUI will not open correctly without the matching `.fig` file.

---

## Requirements

### MATLAB

This project was written using legacy MATLAB APIs and GUIDE-style GUI code.

Recommended:

- MATLAB R2015a to R2018b for the closest legacy compatibility
- later MATLAB versions may require code migration

### Required toolboxes

- Image Processing Toolbox
- Statistics and Machine Learning Toolbox
- Wavelet Toolbox

The original implementation uses several older functions:

```matlab
im2bw
makecform
applycform
svmtrain
svmclassify
classperf
crossvalind
```

Some of these functions are deprecated or removed in newer MATLAB releases.

---

## Installation

Clone the repository:

```bash
git clone https://github.com/your-username/brain-mri-classification.git
cd brain-mri-classification
```

Open MATLAB and set the repository as the current folder.

Alternatively:

```matlab
addpath(genpath('path/to/brain-mri-classification'));
```

Confirm that these files are available:

```text
Trainset.mat
Normalized_Features.mat
BrainMRI_GUI.fig
BrainMRI_GUI.m
```

---

## Running the Main Script

In MATLAB:

```matlab
BrainMRI_Classification
```

The script opens a file-selection dialog:

```matlab
[filename, pathname] = uigetfile(...);
```

Select an MRI image.

The script then:

1. displays the original image
2. resizes it to `200 × 200`
3. converts it to grayscale
4. applies thresholding
5. performs segmentation
6. extracts DWT and PCA features
7. computes texture and statistical features
8. loads the training set
9. predicts the MRI class
10. evaluates several SVM kernels

---

## Running the GUI

Run:

```matlab
BrainMRI_GUI
```

The GUI supports:

- MRI image loading
- image preview
- segmented image preview
- feature extraction
- benign or malignant classification
- display of individual feature values
- repeated SVM accuracy evaluation
- comparison of multiple kernels

---

## GUI Workflow

### 1. Load MRI image

The first button opens an image-selection dialog:

```matlab
uigetfile('*.jpg;*.png;*.bmp', ...
          'Pick an MRI Image');
```

The selected image is resized and displayed.

### 2. Segment and classify

The processing callback:

- converts the image to grayscale
- applies binary thresholding
- removes small regions
- computes DWT coefficients
- applies PCA
- extracts 13 features
- loads the training set
- predicts the class
- displays the result

### 3. Evaluate kernels

Separate GUI buttons run repeated holdout validation for:

- RBF SVM
- linear SVM
- polynomial SVM
- quadratic SVM

The maximum observed accuracy is shown in the corresponding GUI field.

---

## Example Usage

```matlab
I = imread('sample_images/example_mri.png');
I = imresize(I, [200, 200]);

if size(I,3) == 3
    gray = rgb2gray(I);
else
    gray = I;
end

level = graythresh(gray);
segmented = imbinarize(gray, level);

[cA1,cH1,cV1,cD1] = dwt2(segmented,'db4');
[cA2,cH2,cV2,cD2] = dwt2(cA1,'db4');
[cA3,cH3,cV3,cD3] = dwt2(cA2,'db4');

DWT_feat = [cA3,cH3,cV3,cD3];
G = pca(double(DWT_feat));
```

This example uses `imbinarize`, which is recommended for modern MATLAB versions.

---

## Modern MATLAB Compatibility

The original project relies on legacy MATLAB functions. Recommended replacements are:

| Legacy function | Recommended replacement |
|---|---|
| `im2bw` | `imbinarize` |
| `svmtrain` | `fitcsvm` |
| `svmclassify` | `predict` |
| `crossvalind` | `cvpartition` |
| `classperf` | confusion matrix and custom metrics |
| `makecform` / `applycform` | `rgb2lab` |
| GUIDE | App Designer |

### Example modern SVM code

```matlab
model = fitcsvm( ...
    xdata, ...
    group, ...
    'KernelFunction', 'linear', ...
    'Standardize', true);

species = predict(model, feat);
```

For an RBF model:

```matlab
model = fitcsvm( ...
    xdata, ...
    group, ...
    'KernelFunction', 'rbf', ...
    'Standardize', true);
```

---

## Data Format

### `Trainset.mat`

Expected contents:

```matlab
meas   % N × 13 feature matrix
label  % N × 1 class labels
```

Example:

```matlab
size(meas)
% N × 13
```

The extracted test feature vector must have the same number and order of features:

```matlab
size(feat)
% 1 × 13
```

### `Normalized_Features.mat`

Expected contents:

```matlab
norm_feat
norm_label
```

These are used in the normalization and cross-validation section.

---

## Validation Methods

The code contains several evaluation approaches.

### Repeated holdout

```matlab
[train,test] = crossvalind('HoldOut',groups);
```

This is repeated multiple times and the maximum accuracy is reported.

### Kernel comparison

The same split is used to compare:

- linear
- RBF
- polynomial
- quadratic

### Five-fold cross-validation

The script also attempts K-fold validation using normalized features.

---

## Important Evaluation Warning

The current project reports the **maximum accuracy across repeated random holdout runs**.

This can produce an overly optimistic result because the best split is selected after many trials.

A more reliable evaluation should report:

- mean accuracy
- standard deviation
- sensitivity
- specificity
- precision
- recall
- F1-score
- ROC-AUC
- confusion matrix
- confidence intervals

Example:

```matlab
meanAccuracy = mean(Accuracy_Percent);
stdAccuracy = std(Accuracy_Percent);
```

Do not present the maximum holdout accuracy alone as the expected generalization performance.

---

## Known Issues in the Current Code

### Grayscale input handling

The code directly calls:

```matlab
gray = rgb2gray(I);
```

This may fail when the input image is already grayscale.

Safer version:

```matlab
if size(I,3) == 3
    gray = rgb2gray(I);
else
    gray = I;
end
```

### Otsu threshold calculated from the wrong variable

The code uses:

```matlab
level = graythresh(I);
```

It should normally use the grayscale image:

```matlab
level = graythresh(gray);
```

### Threshold not consistently applied

In the GUI, the Otsu threshold is calculated but the segmentation uses:

```matlab
img = im2bw(I, .6);
```

This uses a fixed threshold rather than the computed Otsu threshold.

### K-means configured for one cluster

The script sets:

```matlab
nColors = 1;
```

One cluster cannot separate foreground from background or distinguish tumor tissue.

A meaningful clustering setup would usually require at least two clusters:

```matlab
nColors = 2;
```

or more, depending on the image and tissue classes.

### Segmented image cell size mismatch

The code creates:

```matlab
segmented_images = cell(1,3);
```

but uses `nColors = 1`.

The cell size should be based on the selected number of clusters:

```matlab
segmented_images = cell(1,nColors);
```

### Global name conflict

The variable name:

```matlab
gray
```

is acceptable locally, but care should be taken not to shadow functions or reuse ambiguous names across scripts.

### PCA usage

MATLAB's `pca` expects observations in rows and variables in columns. Applying it directly to a wavelet coefficient matrix may not represent the intended sample-feature organization.

The feature design should explicitly define:

- what constitutes one observation
- what constitutes one feature
- whether PCA is trained only on training data
- how the same PCA transformation is applied to unseen images

### Potential data leakage

If PCA or normalization is fitted using the complete dataset before cross-validation, information from the test folds may leak into training.

PCA and normalization should be fitted inside each training fold.

### Duplicate group assignment

The code includes:

```matlab
groups = ismember(label,'BENIGN   ');
groups = ismember(label,'MALIGNANT');
```

The first assignment is immediately overwritten.

Only one definition should be used.

### Five-fold loop is incorrect

The code uses:

```matlab
indicies = crossvalind('Kfold',label,5);

for i = 1:length(label)
    test = (indicies == i);
```

For five-fold validation, the loop should run from `1` to `5`, not over the number of labels:

```matlab
for i = 1:5
    test = (indicies == i);
    train = ~test;
end
```

### Reusing `classperf`

The same performance object may be reused across multiple kernel evaluations, which can mix results.

Create a new evaluation object for every independent model.

### Duplicate local function names

The supplied script defines `crossfun` more than once.

MATLAB does not allow multiple local functions with the same name in one file.

### Incomplete optimization function

The optimization section references undefined variables:

```matlab
cdata
grp
```

It also contains an incomplete first function block.

This section will not run without restructuring and defining all required inputs.

### Deprecated GUI technology

The GUI was built with MATLAB GUIDE, which is no longer the recommended GUI framework.

For long-term maintenance, migrate the interface to App Designer.

---

## Recommended Evaluation Design

For more scientifically valid results:

1. divide data at the patient level
2. preserve class balance using stratification
3. fit preprocessing only on training folds
4. fit PCA only on training folds
5. tune SVM hyperparameters using nested cross-validation
6. evaluate once on an untouched test set
7. report all clinically relevant metrics
8. include confidence intervals
9. avoid selecting the best result from repeated random splits
10. document the dataset source and class distribution

---

## Suggested Improvements

Future development could include:

- migration from GUIDE to App Designer
- replacement of legacy SVM functions
- automatic grayscale handling
- adaptive tumor segmentation
- connected-component analysis
- morphological tumor-mask cleanup
- train-only normalization and PCA
- nested cross-validation
- confusion matrix visualization
- ROC and precision-recall curves
- model persistence
- reproducible random seeds
- patient-level train/test splitting
- batch processing of MRI datasets
- NIfTI and DICOM support
- comparison with CNN and transfer-learning models
- Grad-CAM or interpretable image-based explanations

---

## Reproducibility

For repeatable holdout experiments, set the MATLAB random seed:

```matlab
rng(42);
```

Store:

- MATLAB version
- toolbox versions
- random seed
- dataset split
- class distribution
- image preprocessing parameters
- feature order
- trained model settings

---

## Troubleshooting

### `Undefined function 'svmtrain'`

The project uses a legacy MATLAB SVM API.

Use an older compatible MATLAB release or migrate to:

```matlab
fitcsvm
predict
```

### `Undefined function 'im2bw'`

Replace:

```matlab
im2bw(I, level)
```

with:

```matlab
imbinarize(gray, level)
```

### GUI opens without controls

Confirm that `BrainMRI_GUI.fig` is in the same directory as `BrainMRI_GUI.m`.

### Feature dimension mismatch

Check:

```matlab
size(meas,2)
size(feat,2)
```

Both must be equal.

### `Trainset.mat` cannot be found

Place `Trainset.mat` in the current MATLAB folder or load it using an explicit path.

### Classification labels contain spaces

Legacy character-array labels may contain padding spaces.

Consider converting labels to categorical:

```matlab
label = categorical(strtrim(cellstr(label)));
```

---

## Clinical and Research Disclaimer

This repository is a technical prototype for education and research.

It has not been validated for:

- clinical diagnosis
- treatment planning
- radiology workflow integration
- regulatory compliance
- patient-level risk assessment
- real-world deployment

MRI tumor assessment requires expert clinical interpretation, validated datasets, rigorous external testing, and regulatory review.

---

## Citation

When using this repository in academic work, cite:

- the repository
- the dataset source
- the original wavelet and SVM methods
- any related publication or technical report

A formal citation can be added using a `CITATION.cff` file.

Example:

```yaml
cff-version: 1.2.0
message: "If you use this software, please cite it."
title: "Brain MRI Tumor Segmentation and Classification"
type: software
authors:
  - family-names: "Your Family Name"
    given-names: "Your Given Name"
version: 1.0.0
```

---

## Contributing

Contributions are welcome, especially for:

- modern MATLAB compatibility
- code modularization
- evaluation corrections
- App Designer migration
- reproducible experiments
- better documentation
- additional tests

Suggested workflow:

```bash
git checkout -b feature/improve-segmentation
git add.
git commit -m "Improve MRI segmentation pipeline"
git push origin feature/improve-segmentation
```

Then open a pull request with a clear description and validation results.

---

## License
- MIT License
---

## Acknowledgements

This project uses MATLAB functionality from:

- Image Processing Toolbox
- Statistics and Machine Learning Toolbox
- Wavelet Toolbox

The original GUI code identifies the project as a brain tumor segmentation and classification application implemented with MATLAB GUIDE.
