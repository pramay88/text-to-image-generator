# AI Text-to-Image Generator

A simple college-lab demonstration that generates 512x512 images from natural-language prompts using Stable Diffusion 1.5, Hugging Face Diffusers, PyTorch, and Streamlit.

## Architecture

```text
Streamlit UI (app.py)
        |
        v
Generator service (generator.py)
        |
        v
StableDiffusionPipeline
  CLIP tokenizer/text encoder
  latent noise + U-Net denoising
  VAE decoder
        |
        v
PIL image -> preview, PNG download, outputs/generated_image.png
```

The model is downloaded from `stable-diffusion-v1-5/stable-diffusion-v1-5` on first use and cached by Streamlit. No API key, database, authentication, or paid service is required.

## Folder structure

```text
text-to-image/
├── app.py
├── generator.py
├── requirements.txt
├── README.md
├── outputs/
│   └── .gitkeep
└── colab/
    └── text_to_image.ipynb
```

## Windows PowerShell installation

For an RTX 3050, install the CUDA-enabled PyTorch wheel first:

```powershell
cd D:\Builds\dall-e
py -3.10 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install torch torchvision --index-url https://download.pytorch.org/whl/cu121
pip install -r requirements.txt
```

If the CUDA wheel is incompatible with the installed NVIDIA driver, use the
current command from the official PyTorch selector, or install CPU PyTorch:

```powershell
pip install torch torchvision
pip install -r requirements.txt
```

## Verify the GPU

```powershell
nvidia-smi
python -c "import torch; print('CUDA:', torch.cuda.is_available()); print(torch.cuda.get_device_name(0) if torch.cuda.is_available() else 'CPU fallback')"
```

## Run locally

```powershell
.\.venv\Scripts\Activate.ps1
streamlit run app.py
```

Open the displayed local URL, usually `http://localhost:8501`. The first
generation downloads several gigabytes of model files and can take time.
Subsequent generations reuse the cached model.

CUDA mode uses FP16, attention slicing, and Accelerate CPU offloading to reduce
VRAM use. CPU mode is automatic when CUDA is unavailable, but is much slower.

## Google Colab fallback

1. Open `colab/text_to_image.ipynb` in Google Colab.
2. Select **Runtime > Change runtime type > T4 GPU** (if available).
3. Run all cells.
4. The notebook installs packages, loads the same pretrained model, generates
   an image, displays it, and offers a PNG download.

Colab free GPU availability is variable and may be limited. The notebook does
not require a paid API or Hugging Face token.

## Test prompts

1. `A small wooden cabin beside a reflective alpine lake at sunrise`
2. `A friendly robot librarian in a magical library full of floating books`
3. `A futuristic Indian city street during monsoon, glowing neon reflections`

## Troubleshooting

- **CUDA is False:** Confirm `nvidia-smi`, install a matching CUDA PyTorch
  wheel, and restart the terminal. The app still runs using CPU.
- **Out of memory:** Close GPU applications, use 10-15 steps, or use CPU mode.
  The app already enables attention slicing and model CPU offloading.
- **Model download fails:** Check internet access, available disk space, and
  retry. Diffusion model files are large.
- **PowerShell activation is blocked:** Run
  `Set-ExecutionPolicy -Scope Process Bypass`, then activate the environment.
- **Generation is slow:** This is expected on CPU and during the first model
  download. A laptop RTX 3050 may still take tens of seconds per image.

## Viva questions and answers

1. **What is diffusion?** It learns to reverse a gradual noise process, starting
   from random latent noise and denoising it into an image.
2. **What does CLIP do?** CLIP's text encoder converts the prompt into an
   embedding that guides the U-Net during denoising.
3. **What is the U-Net?** It predicts noise at each denoising timestep.
4. **What does the VAE do?** It decodes the final latent representation into
   pixel-space image data.
5. **Why use FP16?** Half precision reduces VRAM usage and usually speeds up
   compatible NVIDIA GPU inference.
6. **What are inference steps?** The number of denoising iterations; more steps
   can improve quality but increase runtime.
7. **Why use a negative prompt?** It describes unwanted visual properties such
   as blur, distortion, or watermarks.
8. **Why cache the pipeline?** Streamlit reruns the script after interactions;
   resource caching prevents reloading the multi-gigabyte model every time.
