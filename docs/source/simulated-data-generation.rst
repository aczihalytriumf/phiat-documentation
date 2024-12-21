Simulated Data Generation
========

Overview
-----

- **Language:** MATLAB
- **TIPS.m** creates simulated data files based on user-input cyclotron frequencies and accumulation times

Input
-----

- Various User Inputs (detailed below)

Output
-----

- .csv files containing simulated position, time-of-flight, and (fake) TDC trigger information
- One .csv file containing an address list of files, their accumulations times (Taccs), and reference files assignments

User Inputs
-----

- **tacc_list:** A list of accumulation times you want data simulated for. One file will be made for each Tacc
- **new_filename:** Directory (and filename) to be used
  - new_filename should be of the form: ~/path/to/directory/filename
  - ~/path/to/directory is the directory where files will be saved
  - filename will be the name of the FileList.csv
  - filenameTacc.csv will be the name of each simulated data file (where Tacc will be the respective Tacc number)
  - NOTE: When using simulated data with PhIAT.m, make sure Sim Ion Rate is checked and the trap center is set as (0,0)
  - Once user inputs are done, press the MATLAB run button to create your files!
- **spot_freq_list:** A list of cyclotron frequencies of spots you want in the simulated files
  - These can be taken from TITAN_spreadsheet.xlsx
- **R:** Average distance from trap center of the spots
- **pos_deviation:** Simulated Gaussian spread of X/Y positions
- **TOF:** Simulated ToF between ejection and detector of counts
- **tof_deviation:** Simulated Gaussian spread of ToFs
- **counts_per_spot:** Counts to be simulated for each spot (where spots are determined by cyclotron frequency list)
- **w_minus:** Magnetron frequency in trap
- **amp:** Amplitude of the sinusoid dependence (see Automatic Fitting for more explanation)
- **phase_const:** Phase constant of the sinusoid dependence
- **ref_phase:** Phase of the reference (i.e. Tacc = 0) spot
