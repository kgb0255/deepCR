[![Build Status](https://travis-ci.com/profjsb/deepCR.svg?token=baKtC9yCzzwzzqM9ihAX&branch=master)](https://travis-ci.com/profjsb/deepCR) [![codecov](https://codecov.io/gh/profjsb/deepCR/branch/master/graph/badge.svg?token=SIwJFmKJqr)](https://codecov.io/gh/profjsb/deepCR)
[![Documentation Status](https://readthedocs.org/projects/deepcr/badge/?version=latest)](https://deepcr.readthedocs.io/en/latest/?badge=latest) [![DOI](https://joss.theoj.org/papers/10.21105/joss.01651/status.svg)](https://doi.org/10.21105/joss.01651) [![arXiv](https://img.shields.io/badge/astro--ph-1907.09500-blue)](https://arxiv.org/abs/1907.09500) 

## deepCR: Deep Learning Based Cosmic Ray Removal for Astronomical Images

Identify and remove cosmic rays from astronomical images using trained convolutional neural networks.

Documentation and tutorials: https://deepcr.readthedocs.io/

This is the installable package which implements the methods described in the paper: Zhang & Bloom (2019), ApJ accepted.
If you use this package, please cite Zhang & Bloom (2019): https://arxiv.org/abs/1907.09500 and consider including a
link to this repository.

Code to reproduce benchmarking results in the paper is at: https://github.com/kmzzhang/deepCR-paper



<img src="https://raw.githubusercontent.com/profjsb/deepCR/master/imgs/postage-sm.jpg" wdith="90%">

### New for v0.2.0

DECam deepCR model now available!
```python
from deepCR import deepCR
decam_model = deepCR(mask='decam', device='CPU')
acswfc_model = deepCR(mask='ACS-WFC-F606W-2-32', inpaint='ACS-WFC-F606W-2-32', device='GPU')
```

### Installation

```bash
pip install deepCR
```

Or you can install from source:

```bash
git clone https://github.com/profjsb/deepCR.git
cd deepCR/
python setup.py install
```

### Quick Start

Quick download of a HST ACS/WFC image

```bash
wget -O jdba2sooq_flc.fits https://mast.stsci.edu/api/v0.1/Download/file?uri=mast:HST/product/jdba2sooq_flc.fits
```

With Python >=3.5:

For smaller sized images
```python
from deepCR import deepCR
from astropy.io import fits
image = fits.getdata("jdba2sooq_flc.fits")[:512,:512]

# create an instance of deepCR with specified model configuration
mdl = deepCR(mask="ACS-WFC-F606W-2-32",
	     inpaint="ACS-WFC-F606W-2-32",
             device="CPU")

# apply to input image
mask, cleaned_image = mdl.clean(image, threshold = 0.5)
# best threshold is highest value that generate mask covering full extent of CR
# choose threshold by visualizing outputs.
# note that deepCR-inpaint would overestimate if mask does not fully cover CR.

# if you only need CR mask you may skip image inpainting for shorter runtime
mask = mdl.clean(image, threshold = 0.5, inpaint=False)

# if you want probabilistic cosmic ray mask instead of binary mask
prob_mask = mdl.clean(image, binary=False)
```

For WFC full size images (4k * 2k), you should specify **segment = True** to tell deepCR to segment the input image into 256*256 patches, and process one patch at a time.
Otherwise this would take up > 10gb memory. We recommended you use segment = True for images larger than 1k * 1k on CPU. GPU memory limits may be more strict.
```python
image = fits.getdata("jdba2sooq_flc.fits")
mask, cleaned_image = mdl.clean(image, threshold = 0.5, segment = True)
```

(CPU only) In place of **segment = True**, you can also specify **parallel = True** and invoke the multi-threaded version of segment mode. This will speed things up. You don't have to specify segment = True again.
```python
image = fits.getdata("jdba2sooq_flc.fits")
mask, cleaned_image = mdl.clean(image, threshold = 0.5, parallel = True, n_jobs=-1)
```
**n_jobs=-1** makes use of all your CPU cores.

Note that this won't speed things up if you're using GPU!

### Currently available models

```python
from deepCR import deepCR
decam_model = deepCR(mask='decam', device='CPU')
acswfc_model = deepCR(mask='ACS-WFC-F606W-2-32', inpaint='ACS-WFC-F606W-2-32', device='GPU')
```

**ACS/WFC model**: Input images for the ACS/WFC model should come from *_flc.fits* files which are in units of electrons.
The current ACS/WFC model is trained and benchmarked on HST ACS/WFC images in the F606W filter.
Visual inspection shows that these models also work well on filters from F435W to F814W. However, users should use a 
higher threshold for images of denser fields in short wavelength filters to minimize false detections, if any.

### Contributing

We are very interested in getting bug fixes, new functionality, and new trained models from the community (especially for ground-based imaging and spectroscopy). Please fork this repo and issue a PR with your changes. It will be especially helpful if you add some tests for your changes.
