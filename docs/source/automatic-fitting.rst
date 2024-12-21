Automatic Fitting Overview
========

- Language: MATLAB (R2016a or newer)

- Input: A directory address (i.e. address to a folder) of MIDAS Files (ex: ``/Users/wsporter/Documents/Physics_Research/TITAN/PIICR_Analysis/Testing/Test_CAENOffset``) OR a File List.csv (see midas_to_phiat Documentation for more)

- Output:
  - A .csv containing relevant info and potential cyclotron frequencies of spots in all files (if Create Freq ID checked)
  - A .xlsx containing relevant data fitting results (if Finish Analysis checked)
  - A .png of the sinusoidal fit of cyclotron frequency data (if Finish Analysis and Sine Curve Fitting are checked)
  - A .png of the sinusoidal fit of radius data (if Finish Analysis and Sine Curve Fitting are checked)

Usage
-----

- Open PhIAT.m in MATLAB and at the top of the MATLAB window, click **Run** to open the GUI
- Click the red **Select file(s) for analysis** button in the top left of the GUI
- Navigate to the location of the files for analysis, select relevent runs you wish to analyze, and click **Open**
  - If you are using true data, select multiple .mid.lz4 files
  - If you are using simulated data (created with PI_ICR_simulated_data.m), select the FileList.csv file 
- Click the **Load File Button** in the top center of the GUI to load the first time
- Perform a TOF cut
  - Using the distribution seen in the ToF plot, determine bounds that cut out data outside of the counts majority
  - It’s expected the distribution is pseudo-Gaussian
  - Put these bounds in the **Lower Bound** and **Upper Bound** boxes next to the ToF plot
  - These will be taken into account when **Automatic Fit** is then pressed
- Click the green **Automatic Fit** button 
  - If you are using simulated data (created with PI_ICR_simulated_data.m), check the **Sim Ion Rate** box before clicking **Automatic Fit**
- Enter the **Cyclotron Frequency Guess** in the top left of the GUI 
- Enter **N Range** in the top right of the GUI (details below on how to pick a vlaue for this)
- Select which species you’d like to finish analysis for by designating that spot’s angle in Spot Angle (in degrees)
  - From -180 to 180 (w.r.t x-axis)
- Optional: Enter Isotope, Time in Trap, and Date in the top left of the GUI to save this information in the file name

Finish Analysis Overview
-----

Checking **Finish Analysis** will determine the final cyclotron frequency of one of the species present in the file.

Outputs:
- **Data.xlsx:** Contains all relevant fitting information

Create Freq ID Overview
-----

Checking **Create Freq ID** will create ``ID_Frequencies.csv``, which contains the following data about every spot in every ‘final’ file:

- Tacc [s]
- Spot Color
- Spot Angle [-180 to 180 deg] (with respect to x-axis)
- Ion Rate [counts/s]: Rate at which counts in a spot come in
- Ion Percentage: Percentage of total counts (unnormalized)
- Cyc Freq: Cyclotron frequencies ⍵c,N

Sine Curve Fitting Overview
-----

Checking **Sine Curve Fitting** will determine the final cyclotron frequency of one of the species present in the file by fitting a sinusoid to a spot’s cyclotron frequency at each ``Tacc``.

Outputs:
- **SineFit.png:** An image of the ⍵c vs. ``Tacc`` sinusoid fit
- **RadiusSineFit.png:** An image of the Radius vs. ``Tacc`` sinusoid fit

Output: Data.csv
-----

- ``Data.csv`` is saved, containing tabs of relevant data:
  1. **Cyc. Freq.:** The final cyclotron frequency, uncertainty, and reduced Chi^2
  2. **Cyc. Freq. Data:** The data input to sinusoid cyclotron frequency fit
  3. **Cyc. Freq. Fit Parameters:** Resulting sinusoid fit parameters and uncertainties for cyclotron frequency 

If **Sine Curve fitting** is checked, ``Data.csv`` will also contain  
  4. **Radius:** The final radius, uncertainty, and reduced Chi^2
  5. **Radius Fit Parameters:** Resulting sinusoid fit parameters and uncertainties for radius


Mean Shift Clustering
-----

Once **Automatic Fit** is pressed, the X/Y position data of each file is clustered into groups via a Mean Shift algorithm:

- Determines what cluster centers would have the highest density of nearest neighbors, and then groups those neighbors into a cluster

The total number of clusters is NOT predetermined and depends on two user choices:

- **Spot Bandwidth:** The effective “radius” of the cluster
  - A smaller value means smaller/finer clusters

- **Points per Cluster:** Any resulting clusters that have fewer points than this value will instead belong to no cluster
  - You will likely need to play around with these parameters to get clusters that represent what you would like/expect

Fitting Clustered Data
-----

- The center of each cluster is determined by a 1D Gaussian fit in X/Y
  - via the Maximum Likelihood method
- The resulting fit parameters (mean and square root of the variance) are taken as the spot’s center and standard deviation
- Fits are done for every cluster in a file, and then resulting clusters (and unclustered points) and fit centers are plotted
- PhIAT then cycles to the next file and does the same until all files have been fit
- If you have not checked any other boxes in the Automatic Fitting section, this is where the process stops

Reference and Final Spots
-----

- Each “final” file (Nonzero Tacc) has been paired with a “reference” file (Zero Tacc) in the FileList.csv
- Based on ordering in FileList.csv, “final” files were fit first, and “reference” files second
- This pairing is used when determining the phase difference (ɸc)


Determining Phase -- Trap Center
-----

- The Trap Center [mm] is a user-defined input; make sure it reflects the most recent trap center determination to a given set of data
- The Trap Center uncertainty [mm] must be changed in the script itself (lines 556-557)
- **NOTE:** If using simulated data from `PI_ICR_simulated_data.m`, Trap Center is (0,0)

Finding Number of Turns -- How It Works
-----

- To determine ⍵c, we still need to know the number of turns (i.e. full revolutions) the ion made in the trap
  - N(tacc)

- We can’t determine this experimentally, so we need to infer it
  - This is done using the guess cyclotron frequency (⍵c, guess)
    - We determine the number of turns (Nguess) a spot would have given ⍵c, guess and ɸc = 0

- We then generate a range of N values, where n_range is user-specified

- We then generate a cyclotron frequency (⍵c,N) for every N value

Selecting ⍵c from ⍵c,N -- How It Works
-----

- We need to select one ⍵c from the array of ⍵c,N for each spot
- The ⍵c closest to ⍵c, guess is selected, and thus is dependent on the user-input for Cyclotron Frequency Guess


Radius Sinusoid Fitting -- How It Works
-----

- The radii of the spots should also follow a similar sinusoid
- A 4-parameter sinusoid is fit to the radius data
- This is only used to confirm the data is as expected
- If the fit is poor, this can be a signal that something is wrong
- A .png image of both the ⍵c and Radius sinusoid fits is saved (SineFit.png and RadiusSineFit.png)

Systematics -- Phase Cutoff
-----

- To minimize effects of magnetic field misalignment, a phase cutoff can be applied to cut away any data points with a reference/final phase difference greater than a user-set value (in deg). 
  - i.e. any data points with final/reference phase > 10 deg here are not incorporated into sinusoid fit.
  - Only implemented when box is checked.
