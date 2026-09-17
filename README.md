<h1 align="center">AI Portrait Generator</h1>

<h3 align="center">Photo-to-portrait generation using U²-Net and OpenCV</h3>

<p align="center">
  Transform your photos into pencil sketches or monochrome AI portraits, compare them with the originals, and download your results as PNG images.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Python-3776AB?style=flat&amp;logo=python&amp;logoColor=white" alt="Python" />
  <img src="https://img.shields.io/badge/Streamlit-FF4B4B?style=flat&amp;logo=streamlit&amp;logoColor=white" alt="Streamlit" />
  <img src="https://img.shields.io/badge/PyTorch-EE4C2C?style=flat&amp;logo=pytorch&amp;logoColor=white" alt="PyTorch" />
  <img src="https://img.shields.io/badge/OpenCV-5C3EE8?style=flat&amp;logo=opencv&amp;logoColor=white" alt="OpenCV" />
  <img src="https://img.shields.io/badge/Hugging%20Face-FFD21E?style=flat&amp;logo=huggingface&amp;logoColor=black" alt="Hugging Face" />
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-yellow?style=flat" alt="MIT License" /></a>
  <a href="https://portrait-generator.streamlit.app/"><img src="https://img.shields.io/badge/Live%20Demo-Streamlit-FF4B4B?style=flat&amp;logo=streamlit&amp;logoColor=white" alt="Open Streamlit app" /></a>
</p>

---

## Overview

**AI Portrait Generator** is an interactive image-processing application that uses **U²-Net and OpenCV** to turn uploaded photographs into stylized portraits. It combines neural network inference with image-processing techniques to offer two distinct styles, providing **side-by-side comparisons and downloadable PNG results** through a Streamlit interface.

The application processes images at **512 × 512 pixels** and runs a pretrained U²-Net model on the CPU. Its prediction is converted directly into a monochrome portrait for AI Draw, or used as a mask alongside **edge detection and dodge blending** to produce a pencil sketch with a white background.

---

## Live Demo

**[Try AI Portrait Generator on Streamlit](https://portrait-generator.streamlit.app/)**

Choose a style, upload a photo, and generate a portrait directly in your browser without installing the project locally.

---

## Features

| Feature | What it does |
| --- | --- |
| Pencil Sketch | Combines grayscale shading, detailed edges, and a model-derived mask to create a sketch effect. |
| AI Draw | Converts the inverted, normalized model prediction into a monochrome portrait. |
| Photo upload | Accepts JPG, JPEG, and PNG images through the browser. |
| Before-and-after comparison | Displays the original photograph and generated portrait side by side. |
| PNG download | Exports the generated image as `portrait.png`. |
| Cached model loading | Downloads weights through Hugging Face Hub and caches the loaded model with Streamlit. |
| CPU inference | Runs without requiring a GPU. |

---

## Tech Stack

| Concern | Technology | Purpose |
| --- | --- | --- |
| Language | Python | Application and model implementation |
| Interface | Streamlit | Uploads, style selection, previews, and downloads |
| Neural network | PyTorch | U²-Net architecture and pretrained inference |
| Image processing | OpenCV | Smoothing, shading, edge detection, and mask processing |
| Image handling | Pillow | Image conversion, resizing, enhancement, and PNG export |
| Numerical processing | NumPy | Array operations and image compositing |
| Preprocessing | scikit-image | Resizing images for model input |
| Model distribution | Hugging Face Hub | Downloading the portrait checkpoint |

Dependencies are listed in [requirements.txt](requirements.txt). Versions are currently unpinned.

---

## Project Structure

```text
portrait-generator/
├── app.py             # Streamlit interface, preprocessing, and portrait styles
├── u2net_model.py     # U²-Net blocks, encoder, decoder, and output fusion
├── requirements.txt   # Python dependencies
├── .gitignore         # Excludes environments, caches, and model weights
├── LICENSE            # MIT license
└── README.md          # Project overview and setup guide
```

Model weights are downloaded at runtime and are not committed to the repository.

---

## Quick Start

### Prerequisites

- Python and pip installed; the repository does not currently specify a minimum Python version.
- Git to clone the repository.
- Internet access to install dependencies and download the model on first use.

### 1. Clone the repository

```bash
git clone https://github.com/Maxwelltebi/portrait-generator.git
cd portrait-generator
```

### 2. Create and activate a virtual environment

**Windows PowerShell:**

```powershell
python -m venv venv
.\venv\Scripts\Activate.ps1
```

**macOS / Linux:**

```bash
python3 -m venv venv
source venv/bin/activate
```

### 3. Install dependencies

```bash
python -m pip install -r requirements.txt
```

### 4. Launch the application

```bash
python -m streamlit run app.py
```

Open the local URL printed in the terminal. The application displays a style selector and photo uploader. The model loads when you first select **Generate Portrait**, so the first generation also includes the checkpoint download and initialization.

---

## How It Works

### Image preparation

The uploaded image is converted to RGB and resized to 512 × 512 pixels. For model input, the application scales pixel values by the image maximum, applies per-channel normalization, and converts the result to a PyTorch tensor. A separate resized RGB copy is used for pencil-sketch processing.

### U²-Net inference

The network in [u2net_model.py](u2net_model.py) uses nested residual U-blocks in an encoder-decoder structure. Six side outputs are resized and fused into a single prediction map through a convolution and sigmoid activation.

The application loads the pretrained checkpoint on the CPU, switches the model to evaluation mode, and runs inference with gradient tracking disabled. This repository contains the inference application, not a training pipeline.

### Portrait styling

| Style | Processing pipeline |
| --- | --- |
| Pencil Sketch | Bilateral smoothing → grayscale dodge blending → Canny edges → shading/edge multiplication → model-mask compositing against white → contrast and sharpness enhancement |
| AI Draw | Prediction normalization → inversion → normalization → conversion to an RGB image with monochrome content |

Both styles produce a 512 × 512 image for preview and PNG download.

---

## Results and Limitations

- **Output resolution:** Both styles use a fixed 512 × 512 canvas. Non-square photographs are resized to a square, which can distort proportions.
- **Style behavior:** Results depend on the input photograph and the pretrained model's prediction. Pencil Sketch also depends on fixed edge and mask thresholds.
- **Performance:** Inference runs on the CPU. No generation-time benchmarks or image-quality evaluation results are included in the repository.
- **First-use dependency:** The application requires access to its configured Hugging Face checkpoint when it is not already cached.
- **Input edge cases:** The normalization functions do not guard against zero denominators for all-black inputs or constant prediction maps.
- **Reproducibility:** Dependency versions and the downloaded checkpoint revision are not pinned.

---

## Usage

1. Choose **Pencil Sketch** or **AI Draw**.
2. Upload a JPG, JPEG, or PNG photograph.
3. Select **Generate Portrait** and wait for processing to finish.
4. Compare the generated portrait with the original.
5. Select **Download Portrait** to save `portrait.png`.

To try the other style, change the selection and generate the portrait again.

---

## Configuration

The current application defines model settings directly in [app.py](app.py); it does not provide an application-specific environment configuration file.

| Setting | Current value | Purpose |
| --- | --- | --- |
| `HF_REPO_ID` | `Maxwelltebi/u2net-portrait` | Hugging Face repository used for checkpoint downloads |
| `HF_FILENAME` | `u2net_portrait.pth` | Checkpoint filename |
| Processing size | `512 × 512` | Model input and portrait output dimensions |
| Inference device | CPU | Checkpoint loading and tensor processing |

The checkpoint path comes from `hf_hub_download`. Placing a weights file in the project root does not make the current application load that file instead.

---

## Development

Run the interface from the repository root:

```bash
python -m streamlit run app.py
```

Edit [app.py](app.py) to adjust the interface or portrait effects, and [u2net_model.py](u2net_model.py) to inspect the network architecture. Architecture changes must remain compatible with the loaded checkpoint.

For a manual check, generate both styles from a photograph, verify the comparison display, and open the downloaded PNG. The repository currently has no automated test suite or deployment configuration.

---

## Acknowledgements

- U²-Net for the neural network architecture used in portrait inference.
- The portrait checkpoint distributed through `Maxwelltebi/u2net-portrait` on Hugging Face.
- Streamlit, PyTorch, OpenCV, Pillow, NumPy, and scikit-image for the interface and image-processing tools.

## License

This project is licensed under the [MIT License](LICENSE).

Third-party libraries and pretrained model weights remain subject to their respective licenses.
