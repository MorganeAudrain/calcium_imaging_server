# Radboud Memory Dynamics - Calcium Imaging Analysis

This is the main calcium imaging analysis tool for the Radboud Memory Dynamics group.

The project structure is based on Cookie cutter Data Science by Driven Data. The analysis tools are provided by CaImAn by Flatiron Institute. 

This project is a joint work done by **Melisa Maidana Capitan** (m.maidanacapitan@donders.ru.nl),
**Casper ten Dam**, **Sebastian Tiesmeyer**, **Morgane Audrain** under the supervision of **Francesco Battaglia**. 

## General 

The calcium imaging program have two ways of working, you can choose to run:

* *Pipeline*
* *Pipeline_parameters_settings*

The pipeline code run the all pipeline with define parameters that you can find on the SQL database,
I will come back to the workability of this pipeline later.

The pipeline_parameters_settings is a pipeline where you can choose to run one or two trial of a specific mouse 
or a specific session for set up the more accurate parameters for this session or entire mouse.

The parameters settings pipeline plot for your different picture of each for you to choose the cropping region 
or the filter that you want to use for example.

You can run with this pipeline every step that you want but is not necessary and useful to run all the steps in row with this pipeline.
Once the parameters choose and upgrade on the database you can return to use the global pipeline.

Now that the difference between the two pipelines are explained, I will go more in details about the general use of the pipeline.

For a normal use of the pipeline you are not obligate to go through the organization of each steps, you can just run this pipeline. 

When you start the pipeline, you will be asked to complete some settings:
   
* **the mouse number**:  Identification number of the mouse that you want to analyse    
→ you can analyse just one mouse by one mouse, so you need to enter here just one number. For example, 32364

* **Sessions**: Sessions that you want to analyse for this specific mouse, you can choose to analyse one session or more. For multiple sessions you need to enter an array. For example, one session 1, multiple session [1 2 3]

* **Begin of trial**: the first trial where you want to start the analysis

* **Last trial**: Last trial you want to analyse

After setting those settings the pipeline will ask you which steps you want to run. The choice that you have it between:

* **Decoding**

* **Cropping**

* **Motion correction**

* **Alignment**

* **Equalization**

* **Source extraction**

* **Component evaluation**

* **Registration**

* **All**

 I will come back later the use and option of this different steps. After running the steps that you select the final question of the pipeline is if you want to run other steps with the same parameters meaning the same mouse, sessions, and trial. If you decide to do not run another step the pipeline stop every process that can still run and display the goodbye message.

### 1. Decoding

Decodes the raw files with Inscopix.

This function needs **the mouse number**, **the session**, and **trials** that you want to decode.

### 2. Cropping

Crop the region of interest (ROI) for later analysis. 

This step takes a decoded file and crop it according to specified cropping points.

This input is a tif file and the output is a tif file as well.
You do not need to specify here the decoding version because is by default version one. 
You will be ask here which cropping region you want to applicate, for visualization purpose you can choose to run first pipeline_parameters_settings, choose the parameters that you want and after applicate those parameters for the all session.
The spatial cropping points are referred to the selected interval. y1: start point in y axes, y2: end point in y axes. z1: start point in z axes, z2: end point in z axes. 
Changes in the size of the FOV of course will lead to changes in the initial condition, that can lead to huge differences in the extraction. Do not forget to verify before generalizing your parameters to a bigger FOV. 

### 3. Motion correction

Motion correction can be run both in the server and it the local machine, but it is supposed to be faster in the server. 

for each mouse,session,trial,is_rest parameters and version analysis. 

Once several parameters of motion_correction had been run, it is necessary to compare performance. Easier quality measurement of motion correction is crispness. And the end the script goes over all the analysed motion corrected versions (using function compare_crispness that is defined in analysis.metrics), and with plot_crispness_for_parameters (analysis.figures) creates a plot for cripness for the mean summary image and for the corr summary image. 


Also the script can run several rigid mode, pw_rigid modes, or other variations of the parameters. 

Strides size and overlap are relevant parameters if 'pw_rigid' mode is use. 

Optimal parameters correspond to --- values of crispness. The script prints in the screen the full dictionary corresponding to the optimal ones. 

### 4. Equalizer

This step should be run with to main objectives. 

1) Reducing the photobleching effect (as mentioned in https://imagej.net/Bleach_Correction )
2) Have an stationary signal over different days of recoding. 

The equalization procedure consists in mapping the histogram of pixel values of one image into the histogram of the other image (see '/home/sebastian/Documents/Melisa/calcium_imaging_analysis/data/interim/equalizer/meta/figures/MonaLisa_Vincent_equalization.png' for exemplification). 

It is known that bleaching diminishes the quality of the image turning it darker. The idea here is to use the histogram of the first recording videos as a template to match the following videos.


make_template_from_trial < - Refers to where the reference histogram is taking from
equalizer <- Sets the method to histogram matching. There are other methods like exponential fitting, or mean equalization that are not yet implemented. 
a part of the code also produces the histogram to verify equalization. h_step sets the sets the bin size


### 5.Alignment

It applies methods from the CaImAn package used originally in motion correction to do alignment.

### 6. Source extraction

Source extraction can be run both in the server and it the local machine, but again it is supposed to be faster in the server. 

Source extraction parameters as specified in a dictionary as follows : 

parameters for source extraction and deconvolution

p = 1 -> order of the autoregressive system

K = None -> upper bound on number of components per patch, in general None

gSig = (3, 3) -> gaussian width of a 2D gaussian kernel, which approximates a neuron

gSiz = (13, 13) -> average diameter of a neuron, in general 4*gSig+1

Ain = None -> possibility to seed with predetermined binary masks

merge_thr = .7 -> merging threshold, max correlation allowed

rf = 40 -> half-size of the patches in pixels. e.g., if rf=40, patches are 80x80

stride_cnmf = 20    # amount of overlap between the patches in pixels
                    (keep it at least large as gSiz, i.e 4 times the neuron size gSig)
		    
tsub = 2            # downsampling factor in time for initialization,
                     increase if you have memory problems
		     
ssub = 1            # downsampling factor in space for initialization,
                     increase if you have memory problems
                     you can pass them here as boolean vectors
		     
low_rank_background = None  # None leaves background of each patch intact,
                     True performs global low-rank approximation if gnb>0
		     
gnb = 0             # number of background components (rank) if positive,
                     else exact ring model with following settings
                         gnb= 0: Return background as b and W
                         gnb=-1: Return full rank background B
                         gnb<-1: Don't return background
			 
nb_patch = 0        # number of background components (rank) per patch if gnb>0,
                     else it is set automatically
		     
min_corr = .8       # min peak value from correlation image

min_pnr = 10        # min peak to noise ration from PNR image

ssub_B = 2          # additional downsampling factor in space for background

ring_size_factor = 1.4  # radius of ring is gSiz*ring_size_factor


The script runs different selections of gSig (gSiz = 4 * gSig + 1) and saves the resulting corr and pnr summary image (as well as the combination) in . From here exploration of the effect of different gSig on the summary images can be done.

Later exploration of min_corr and min_pnr can be done. The scripts creates a bunch of histograms to get an intuitive idea of the values of corr and pnr of a video. 

Selection a range or corr and pnr values, there is a script that computes source extraction and plots a figure with all the contour plots for the analyzed parameters. This plotting helps to understand how initial seeds change the final result of the extraction.  

Visual inspection of this figure can help to decide which values of pnr and corr are adequate (selects the higher number of neurons that are 'real neurons'). Parameter selection of source extraction can be done in combination with parameter selection of component evaluation in order to get a better solution.


### 7. Component Evaluation


For running component evaluation, a version of all previous states should be chosen. This step can be run in the local machine.

Component evaluation parameters are 3: minimal value of signal to noise ration in the extracted calcium traces, minimal pearson correlation coefficient between a template for the 'neuron footprint' extracted as a mean over all the images and the source extracted footprint, and a boolean variable that specifies whether the assessment will or will not use a CNN for classification as good/bad component.

Componente evaluation parameters as specified in a dictionary as follows : 


The script proposed runs for all source extraction versions selected different selection of component evaluation parameters and makes some plot where accepted and rejected components can be seen in the contour plot and also in  the traces ( plot_contours_evaluated and plot_traces_multiple_evaluated)


### 8.Registration


