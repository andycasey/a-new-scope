*sick*, the spectroscopic inference crank
------

[![Build Status](http://img.shields.io/travis/andycasey/sick.svg)](https://travis-ci.org/andycasey/sick) [![PyPi download count image](http://img.shields.io/pypi/dm/sick.svg)](https://pypi.python.org/pypi/sick/) [![Coverage Status](https://img.shields.io/coveralls/andycasey/sick.svg)](https://coveralls.io/r/andycasey/sick)


**Idea**

All other attempts to determine stellar parameters from a grid of
pre-computed spectra separate out the normalisation, radial velocity, and synthetic
smoothing components. In reality these phenomena are all linked, and the reliability
of stellar parameter inference will be affected by any uncertainty in these components.
You should have a single mathematical model that can incorporate all of these convolutions, as well as anything else that can affect the spectra. That's what this code does. It's flexible enough for use on any type of spectra.

**Installation**

``pip install sick`` (or [if you must](https://stackoverflow.com/questions/3220404/why-use-pip-over-easy-install), use ``easy_install sick``)


**Usage**

Running *sick* is as easy as:

``sick model.yaml my_spectrum.fits``

If you have multiple spectra in different channels (e.g., multiplexing) then you can analyse all spectra just as easily:

``sick model.yaml --multi-beam blue_channel.fits red_channel.fits``

Or you can analyse the data in a Python script:

````
import sick

# A single spectrum:
posteriors, sampler, info = sick.solve("sun.ms.fits", "model.yaml")

# Multiple spectra of the same star:
# (As long as your model apertures are defined, the order of input spectra does not matter..)
posteriors, sampler, info = sick.solve(["red.fits", "blue.fits"], "model.yaml")

# Or you can load the model first and use it for many stars:
model = sick.models.Model("model.yaml")
sun_posteriors, sun_sampler, sun_info = sick.solve("sun.ms.fits", model)

arcturus_posteriors, arcturus_sampler, arcturus_info = sick.solve("arcturus.fits", model)
````


**Model Example**

In the usage example above, the ``model.yaml`` file contains all the model information required. This file can be a YAML or JSON-style format. Below is an example of what ``model.yaml`` might look like, with comments:

````
solver:
  optimise: yes
  walkers: 200
  burnin: 500
  samples: 100
  threads: 8

models:
  dispersion_filenames:
    # Here we are going to decide on the names of each 'aperture', or each portion of spectrum.
    # If you only have one complete spectrum, just call it 'optical' or something similar.
    blue: /media/wd/ges/synthetic-spectra/GES_UVESRed580/GES_UVESRed580_Lambda.fits
    red:  /media/wd/ges/synthetic-spectra/GES_HR21/GES_HR21_Lambda.fits

  flux_filenames:
    # Here all of our flux files are stored in a folder for each aperture we just named.
    # The parameters we want to infer are in the flux filenames. We can call these parameters
    # whatever we want, and we use a regular expression to find all the relevant files.
    blue:
      folder: "/media/wd/ges/synthetic-spectra/GES_UVESRed580/GES_UVESRed580_deltaAlphaFe+0.0.fits/"
      re_match: '[s|p](?P<teff>[0-9]+):g\+(?P<logg>[0-9.]+).+z(?P<feh>[0-9.+-]+).+fits'
    red: 
      folder: "/media/wd/ges/synthetic-spectra/GES_HR21/GES_HR21_deltaAlphaFe+0.0.fits/"
      re_match: '[s|p](?P<teff>[0-9]+):g\+(?P<logg>[0-9.]+).+z(?P<feh>[0-9.+-]+).+fits'

# Allow for normalisation in the observed spectra
normalise:
  blue:
    method: polynomial
    order: 2
  red:
    method: polynomial
    order: 3

# Allow for individual doppler shifts in each aperture
doppler_shift:
  blue: yes
  red: yes

# Allow for each aperture to have different resolutions
convolve:
  blue: yes
  red: yes
````

**License**

Copyright 2014 Andy Casey 

This is open source software available under the MIT License. For details see the ``LICENSE`` file.
