# openrecon-ants

This OpenRecon was modified by exchanging the SynthStrip brain masking with antspynet brain masking. The original repo can be found at https://github.com/benoitberanger/openrecon-ants.
The following steps are required to run this recon with Siemens FIRE:

1. Necessary files at the scanner: 
    - `%CustomerIceProgs%/fire/fire_i2i.ini`
        - Contains network configuration for reco server including the port number
    - `%CustomerIceProgs%/IceProgramFireImageAddin_ants.xml`
        - Set "IniFile" to "%CustomerIceProgs%\fire\fire_i2i.ini"
        - Set "Config" to "or_ants"
        - Set "Anchor" in the "ImageEmitter" to the last ICE functor before imafinish. This is often the distortion correction (e.g. "DistorCor3D"), but can vary in different sequence. To inspect the ICE chain, export the raw data from Twix (or just the raw data header) and open it with Xbuilder. Then select PIPE in the IRIS section.
        - Optional: Set "PassOnData" to true "within" the ImageEmitter to pass data also along the original ICE pipeline.
    - `%CustomerIceProgs%/IceProgramFireImageAddin_ants.ipr`
2. Build the docker container as described in the "Build" section below with the command `python build.py`.
3. Run the docker container with a command like: `docker run -d -t --rm -p 9023:9002 --gpus all openrecon_icm_ants:v3.0.0`. The port (here 9023) has to be the same as in "fire_i2i.ini".
4. Configure a DotAddin (e.g. Generic Views) to set `IceProgramFireImageAddin_ants.ipr` as an additional IceProgram via Ice Configuration. This is described in the FIRE manual.
5. Add the DotAddin to a protocol and run the sequence.
6. The log file can be found inside the container at /tmp/share/debug/python-ismrmrd-server.log.

Note: If the FIRE image addin is used, the image data gets upscaled for some reason by a factor ~16 (tested for MPRAGE), which can cause clipping due to DICOM limits. Therefore, an appropriate image scaling factor has to be set in the sequence

The Docker image is archived at https://hub.docker.com/repository/docker/mavel101/fire_ants. This archive can also be used to create a singularity image with the command `singularity pull docker://mavel101/fire_ants:latest`.

# Original README

## Screenshots from the MR Host
Siemens 7T Terra.X in XA60A.  
Also tested on Siemens 3T Cima.X on XA61(-SP01)  


## Brainmasking, Debias, Denoise
![ANTs V2 in XA60A on a 7T Terra.X](doc/OpenRecon_ANTS_V2_FLAIR-CP_FLAIR-UP_DIR-UP_SAG_blur.png)  
_Sequence_: Non-selective 3D SPACE.  
_From left to right_: Original, brain mask (SynthStrip), N4BiasFieldCorrection (ANTs) in brain mask, DenoiseImage (ANTs) in brain mask after N4BiasFieldCorrection.  
_From top to bottom_: 3D FLAIR with Circular Polarization (CP), 3D FLAIR with Universal Pulses (UP), 3D DIR with UP.  

[ANTs](https://github.com/ANTsX/ANTs) using [ANTsPy](https://github.com/ANTsX/ANTsPy) in OpenRecon.  
Brain masking is performed by [SynthStrip](https://surfer.nmr.mgh.harvard.edu/docs/synthstrip/)


## SkullStripping

![ANTs V3 in XA60 on a 7T Terra.x](doc/V3_TOF_SkullStripping_MIP_blur.PNG)
_Sequence_: Angio TOF  
_From left to right_: Original, SkullStripped using brainmask from Synthstrip, N4BiasFieldCorrection, DenoiseImage  
_From top to bottom_: SAG MIP, COR MIP, TRA MIP


# Features

This OR performs ANTs image operations : 
- N4BiasFieldCorrection
- DenoiseImage
- N4BiasFieldCorrection then DenoiseImage (default)
- DenoiseImage then N4BiasFieldCorrection
- None (only for brain masking / skull stripping)

Brain mask usage :
- Apply ANTs in brainmask (keep outside the mask intact)
- Skull stripping then ANTs (keep only in mask)
- None (only ANTs, no masking)

There is an option, a checkbox, to **save original images** and intermediate images. (default is _True_)

Based on https://github.com/benoitberanger/openrecon-template


# Build

Requirements for building :
- python **3.12**
- jsonschema

 Python environment manager is **strongly** recomanded :
```bash
conda create --name openrecon-ants
conda install python=3.12
pip install jsonschema
```

Build with :
```bash
python build.py
```


# Offline test and dev

Python modules :
- ismrmrd
- pydicom
- pynetdicom
- antspyx
- torch
- surfa

``` bash
pip install ismrmrd pydicom pynetdicom antspyx torch surfa
```
Follow guidelines in https://github.com/benoitberanger/openrecon-template

# TODO

- Add fields in the UI to tune `N4BiasFieldCorrection` and `DenoiseImage`
- Add fields in the UI to tune the mask (erode ? delate ? ...)

