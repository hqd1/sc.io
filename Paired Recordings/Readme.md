# Introduction

This is the companion repository to our preprint where we describe experiments recording from the same cortical neuron *in vivo* using [Neuropixel probes](https://www.nature.com/articles/nature24636) and patch-clamp:

**Marques-Smith, A., Neto, J.P., Lopes, G., Nogueira, J., Calcaterra, L., Frazão, J., Kim, D., Phillips, M., Dimitriadis, G., Kampff, A.R.** (2018). *Recording from the same neuron with high-density CMOS probes and patch-clamp: a ground-truth dataset and an experiment in collaboration.*  
bioRxiv 370080; doi: https://doi.org/10.1101/370080



In this repository you will find:
- The [code used to preprocess data, run analysis and generate the figures presented in Marques-Smith et al., 2018](https://github.com/kampff-lab/sc.io/tree/master/Paired%20Recordings/Preprint%20Code);
- A list of [proposed collaborative follow-up projects](https://github.com/kampff-lab/sc.io/tree/master/Paired%20Recordings/Projects) based on that same dataset and details on how to contribute;
- Basic instructions on loading recordings on your machine.

## Conditions of use
Code in this repository is shared under the [MIT licence](https://opensource.org/licenses/mit-license.php).

All the data we shared is free for you to use under the licensing conditions of [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/).    We ask that you cite both:
- **The original publication describing the dataset:** Marques-Smith, A., Neto, J.P., Lopes, G., Nogueira, J., Calcaterra, L., Frazão, J., Kim, D., Phillips, M., Dimitriadis, G., Kampff, A.R. (2018). *Recording from the same neuron with high-density CMOS probes and patch-clamp: a ground-truth dataset and an experiment in collaboration.* bioRxiv 370080; doi: https://doi.org/10.1101/370080
- **The dataset itself:** André Marques-Smith, Joana P. Neto, Gonçalo Lopes, Joana Nogueira, Lorenza Calcaterra, João Frazão, Danbee Kim, Matthew G. Phillips, George Dimitriadis and Adam R. Kampff (2018); Simultaneous patch-clamp and dense CMOS probe extracellular recordings from the same cortical neuron in anaesthetized rats. CRCNS.org
http://dx.doi.org/10.6080/K0J67F4T

## Guide to contributions
Please see [here](https://github.com/kampff-lab/sc.io/blob/master/Paired%20Recordings/Projects/readme.md#how-do-i-contribute) for a detailed how-to.

## Dataset and Metadata
The full dataset is freely-available for download [from our Google Drive](https://drive.google.com/open?id=13GCOuWN4QMW6vQmlNIolUrxPy-4Wv1BC) and [CRCNS.org](http://dx.doi.org/10.6080/K0J67F4T). For the moment we are still supporting Google Drive downloads, but given the data is now hosted at CRCNS.org, we will eventually discontinue this. We recommend you use the CRCNS.org link, as this enables easier batch downloading of files, as well as a smaller download size (~276 GB), since the folders are compressed in tar.gz.

The directory contains:
- A folder for each paired recording with data;
- Data Summary.xls - relevant metadata for each paired-recording;
- chanMap.mat - map of channel layout;
- Recording Catalogue.pdf - a series of stereotyped figures summarizing patch-clamp and extracellular recordings for every cell.

The dataset is large. To unpack it in its entirety, you will need **approximately 750 GB hard drive space**. If you do not need the whole dataset, we advise you to use **'Data Summary.xls. and 'Recording Catalogue.pdf'** to choose a subset of recordings that fit your requirements. Those two documents combined with the preprint should contain all the information you need. 

### Dataset organisation
Each folder is titled 'cxx' (cell xx), corresponds to a paired-recording and contains the following files:

- cxx_expt_meta.csv  
*Dimensions that Neuropixel and patch-clamp recording files should be reshaped to and their data type (int, float), when importing into Python or Matlab.*

- cxx_npx_raw.bin  
*Neuropixel recording (384 channels), 1D binary file. The shared data has only been offset-subtracted; no CAR or filtering has been performed on it, as different individuals have distinct preprocessing routines they prefer to use. Please note that this is the action potential band; during acquisition it was high-pass filtered at 300 Hz (first-order). We do have the LFP files, which are available upon any request - see below.*

- cxx_npx_sync.bin  
*Neuropixel sync channel, binary file.*

- cxx_patch_ch1.bin  
*Patch-clamp current or voltage, depending if cell was recorded in voltage- or current-clamp. No preprocessing has been performed on this shared data.*

- cxx_patch_sync.bin  
*Patch-clamp sync channel.*

- cxx_wc_spike_samples.npy  
*Patch-clamp recording samples corresponding to patched cell's spike peaks, numpy 1D array.*

- cxx_extracellular_spikes.npy   
*Only for the 21 cells where an extracellular spike waveform could be detected. Details Neuropixel recording samples corresponding to the peak of extracellular spikes in the channel closest to patched cell's soma, numpy 1D array.*

### How to load data
**Neuropixel**  
Neuropixel recordings were saved into a 1D binary vector, from 2D array organised as 384 rows (channels) x n columns (samples). The data was written to file from this matrix in column-major (F) order, ie, the first sample in the recording was written to file for every channel, then the second sample was written for every channel, etc, etc.

The data are ready to be sorted using your favourite spike-sorting algorithm. If you run into trouble, [drop Andre a line](mailto:andrefmsmith@gmail.com).  

To load the data correctly for other analysis you might want to carry out, you have to tell your software how it's organised and how to unpack it.

In Python, you could do:  
```python 
import numpy as np

npx_path = 'full path to the downloaded cxx_npx_raw.bin file'
npx_channels = 384

npx_recording = np.memmap( npx_path, mode = 'r', dtype=np.int16, order = 'C')

npx_samples = int(len(npx_recording)/npx_channels)

npx_recording = npx_recording.reshape((npx_channels, npx_samples), order = 'F')
```
which will generate a 2D numpy array where channels are rows (384) and columns are samples. Pay attention to the ['mode' parameter](https://docs.scipy.org/doc/numpy-1.13.0/reference/generated/numpy.memmap.html) in the memmap function, as whis will determine whether you're loading the file as read-only or if you're writing changes to disk.

**Neuropixel raw data is provided as an int16 binary. [Neuropixel ADCs](https://github.com/cortex-lab/neuropixels/wiki/Gain_settings) are 10 bits, with a range of -0.6 to 0.6 V, and acquisition was at 500x gain, yielding a resolution of 2.34 µV/bit.  
To obtain readings in µV, you should  multiply the int16 values by 2.34.** 


**Patch-clamp**  
Patch-clamp data are already provided in float64 type and current (pA) or voltage (mV) units, depending if the recording was performed in voltage- or current-clamp.

To load a patch-clamp recording in Python, do:  
```python
import numpy as np

patch_path = 'full path to the downloaded cxx_patch_ch1.bin file'
patch_recording = np.fromfile(patch_path, dtype='float64')
```

### Time-base and sampling rate
The Neuropixel and patch-clamp recordings provided are temporally-aligned. This was done by generating periodic digital pulses, which were recorded on the Neuropixel and patch-clamp sync channels (respectively, cxx_npx_sync.bin and cxx_patch_sync.bin; see Marques-Smith et al., 2018, for detail). Neuropixels signals were acquired at 30 kHz and patch signals at 50.023 kHz. Due to software issues there was some minor variation in exact sampling rate (see [Data Summary](https://github.com/kampff-lab/sc.io/blob/master/Paired%20Recordings/Preprint%20Code/Data%20Summary.xlsx)) between recordings for the patch clamp signal.

To relate patch-clamp and extracellular events, you should obtain the conversion factor (m) that allows you to work out which sample number in Neuropixel corresponds to a sample number in patch-clamp and vice versa. As the two data streams are temporally-aligned, this is simply the ratio between the length (in samples) of the two recordings. In Python, you could do  

```python  
m = len(patch_recording)/float( len(npx_recording[0]))

neuropixel_event = int(patch_sample / m)
patch_event = int(neuropixel_sample * m)
```  

Importantly, you should obtain this conversion factor **for each patch recording** you look at due to the issue described above.

### I'd like to look at LFPs.
Awesome! We have the LFP band (below 300 Hz) recordings for every cell, but these are not yet uploaded. [Drop Andre a line](mailto:andrefmsmith@gmail.com) and he'll send you a download link. Total size for the LFP dataset is ~60 GB and LFP sampling rate is 2.5 KHz. Eventually this will also be added to CRCNS.org.

### "I'm more of a Matlab person"
We focused on Python here, as that's what we use in the lab. All the files we shared are in open formats, and as such usable in Matlab. If you're an experienced Matlab user, we'd really appreciate if you got in touch/sent a pull request with Matlab code to load our Neuropixel and patch-clamp binary data, so that we can put it here instead of this placeholder.

# Help!
If you run into trouble, if something is unclear, you spot a mistake, or if there is information you think would be helpful to add that we've missed out on, please get in touch with Andre Marques-Smith, at andrefmsmith@gmail.com. We're open to suggestions and very eager to help and improve the usability of this repository.

### Good luck! We look forwards to collaborating with you.
