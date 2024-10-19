# EnhanceFX: Model Card

Version: 5.1.7

EnhanceFX V Evolution Prestige is a hybrid super-resolution model crafted in PyTorch. It utilises a combination of techniques found in DRCT and ESRGAN, alongside other optimisations to deliver sharp 16x upscaling, designed to extract both global and fine details effectively.

# Strengths
- 💡 Hybrid Super-Resolution Architecture: Combines Swin Transformers, RRDBs, and MSFE for great 16x upscaling.
- 🧠 Self-Attention Mechanisms: Transformer blocks help with contextual long-term detail preservation, aiding feature reconstruction.
- ⚡ Fast Fourier and Wavelet Frequency Blocks: Enhanced frequency-domain processing to mitigate noise artefacts.
- 🎯 1990/600 Million Parameters: Optimally tuned for both Generator (G) and Discriminator (D) power equality. (Parameter count can change significantly depending on # of RRDBs and Attention blocks, etc.)
- ✨ Distributed PixelShuffle upsampler.

# Model Architecture

**Hybrid Generator (G)**

---
**CORE**
---

Modified ESRGAN-style (5C):
- Residual in Residual Dense Blocks (RRDBs), each composed of 3 Residual Blocks.
- Seperate Dense Residual Blocks featuring deformable convolutions.

Modified DRCT-style:
- Split Dual Swin Transformer Trunks: RoPE-enabled, KAN-modified MLPs for deep feature generation and long-term consistency.

Additions:
- Custom Multi-Scale Feature Extraction (MSFE): Uses 3x3, 5x5, and 9x9 convolution kernels as initial layers.
- Dense connections between Transformer / RRDB trunks to preserve information.
- PixelShuffle (FBGEMM) 8 * 2x sparse staggered upampling blocks with feedback loop (2 between each trunk, step-by-step resolution enhancement).
- Globalised and transitive Skip Connections to link deep trunks.

---
**GLOBAL ADDITIONS**
---

Attention and Misc. Blocks include:
- Spectral
- Channel (from DRCT)
- CAB (from DRCT)
- Window (from DRCT)
- Colour
- Sharpening
- Axial
- Region
- Spatial
- Frequency (Fast Fourier (FFT) & Wavelet Transform)
- Squeeze-Excite

**DISCLAIMER: Attention and other blocks can be deactivated and reactivated on demand, by commenting out the respective blocks. Having all blocks activated is INCREDIBLY expensive and while not tested so far, it doesn't guarantee a superior model architecture!!!**

Activation Functions: LeakyReLU, GELU, and sparse SwiGLU for adaptive activation.

**Hybrid Discriminator (D)**

A mixture of (a):

- Pre-trained VGG-19 for perceptual adversarial loss.
- 24-layer U-Net for structural adversarial comparison.
- Same Axial Attention as the Generator.
- RaGAN + AdaIN.

# Training Process
- Gradual Resolution Increase: Teaches larger details first, progressing to finer details after ~30,000 epochs.
- Dataset: Custom 16K tiles and Nomos8K + zoom images, improving diversity and feature extraction opportunity.
- Colour Correction: Dynamic colour correction is applied to the high-resolution (HR) images during training to maintain colour accuracy. This adjustment helps the model generate outputs that are more visually natural and consistent with the original input.

# Current Progress
Based directly on the 8XOne architecture, this is the 5th iteration in development at King's College London.
Impressive results so far, with improvements still in progress:

- Issues to Address: Vanishing gradients, ISO-style noise artefacts from FFT, and colour leaks between transformers.
- Proposed Fixes: Recursive block interconnects (as a modification or addition to the existing dense ones) improve transformer coordination by promoting better gradient flow and effective communication between layers.

**Diffusion based post-processing**

A final diffusion based model can be used to bridge the gap between the output of EnhanceFX and the GT, given the outputs without post-processing are fairly high quality. Current systems use a single type of model, where using the expertise of an initial Conv/Xformer based model and the refining/style shifting abilities of a diffusion network would be more logical in tackling the shared goal of image restoration.

# Images
https://imgsli.com/MzAwNDM2

This architecture is actively being developed, and the images linked are early-stage results. Final performance and output quality may improve with ongoing research and optimisations.

# References & Motivation Source
- https://github.com/ming053l/DRCT,
- https://github.com/xinntao/ESRGAN,
- https://github.com/JingyunLiang/SwinIR,
- https://github.com/Fanghua-Yu/SUPIR (Diffusion bridger network)

# FAQ

(Based on questions I have received at university)

**Q: Why is there no source code?**

A: Unlike NeuralWorks and NeuralWorksCustom, I’ve chosen not to make the source code public for collaboration at this time. This decision is primarily for IP protection due to novelty, and I want to further research and refine the model before releasing anything. Academic publications are definitely on the horizon when time permits, but for now, I’m focused on ensuring that EnhanceFX reaches its full potential before making it publicly available.

**Q: Does the complexity of the architecture lead to diminishing returns in performance?**

A: Complex architectures like EnhanceFX can indeed lead to diminishing returns, especially when the overhead of training time and resource consumption outweighs the benefits gained from added layers or components. However, combining many exotic systems and experimenting with their interactions promotes deeper architectural research, which is always great to have.

**Q: This all seems too complicated and unrealistically impossible to train.**

A: This is a valid point/concern. In reality, I've tried to craft the architecture with a focus on modularity and efficiency. Each component is purpose-driven and can be selectively activated or deactivated to optimise computational resource usage in my forward()/backward() methods. The iterative addition of many blocks followed testing phases at multiple scales, ensuring that complexity is introduced thoughtfully (rather than impulsively in response to competition or desire to do so) and without negative repercussions. With the right setup and when the blocks work together, achieving high-quality results is not only feasible but has also been successfully demonstrated. (See **Images**)

Got any other questions? motions.07-busses@icloud.com

Thanks.

```python
Please note that this markdown is more detailed than any other reference to EFXVEP, such as LinkedIn.
```
