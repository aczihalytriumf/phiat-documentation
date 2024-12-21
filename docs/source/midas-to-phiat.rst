midas_to_phiat.py Overview
###########

- **Language:** Python 3
- **midas_to_phiat.py** Converts MIDAS files from MPET DAQ to PhIAT-compatible format
- **Input:** directory to MIDAS files (.mid.lz4) 
  - Example: C:\Users\aczihaly\Documents\PIICR_Analysis\Data
- **Output:**
  - .csv files containing position, time-of-flight, and TDC trigger information
  - .csv files containing approximate count rates
  - One .csv file containing an address list of converted MIDAS files, their accumulation times (Taccs), and reference files assignments

Background
-----


Four pieces of information are important and relevent for PI-ICR analysis:

- x position (mm)
- y position (mm)
- time of flight (s)
  - from extraction to detector
- trigger (integer) or timestamp (s)
  - CAEN: trigger
    - Trigger number an ion hit came during (starting from 0 at file beginning)
  - VT2: timestamp
    - Timestamp of event relative to initial trigger time

Initialization
-----

- Download titan_data: https://bitbucket.org/ttriumfdaq/titan_data/src/master/
- On Line 20: Change the address in sys.path.append from '/Users/wsporter/Documents/Physics_Research/TITAN/PIICR_Analysis/titan_data' to the local address of titan_data on your computer
  - Example: sys.path.append('/Users/aczihaly/Documents/PIICR_Analysis/titan_data')
- all_one_file: 
  - Default is all_one_file = TRUE for PI-ICR analysis 
- CAEN/ VT2: Set TRUE whichever TDC is being used to read out data (indicated in the MPET DAQ under 'Scan')
  - Default is CAEN = TRUE
  - CAEN and VT2 cannot both be true - this will trigger an error
- testing: If you are using test data from MIDAS that DOES NOT include a reference file (i.e. a Tacc of 0), set to TRUE (automatically assigns the last file a Tacc of 0, as needed to run through the script)
    - Otherwise, set to FALSE (the default for normal data)

Following the steps, you are now ready to use midas_to_phiat.py automatically as part of PhIAT (see PhIAT documentation for further detail) or separately in a command line.

Data -- CAEN 25ps TDC
-----

- **X/Y Position:** Determined via timing pulses in delay line anode
  - Correspond to ``caen_tdc_parsed.pos_x_mm`` (or ``pos_y_mm``) in ``../titan_data/mpet/__init__.py``
- **ToF:** Time between trap ejection and ion contact with MCP
  - Corresponds to ``caen_tdc_parsed.mcp_tof_secs``
- **Trigger:** Trigger number at which an ion event occurred
  - Corresponds to ``caen_tdc_parsed.trigger_count``

Data -- VT2 TDC
-----

- **X/Y Position:** Determined via LeCroy 1190 Position Analyzer
  - Correspond to ``pos_data.x`` (or ``y``) in ``../titan_data/mpet/__init__.py``
  - **NOTE:** The LC1190 DOES NOT give reliable position data...
- **ToF:** Time between trap ejection and ion contact with MCP
  - Corresponds to ``vt2_data.tof_secs``
  - **NOTE:** Since this is raw data (i.e. each ion hit does not yet correspond to one event),
    need to specify a ``channel_ids`` to get just one trigger value per ion hit
- **Timestamp:** Timestamp of event relative to initial trigger time
  - Corresponds to ``vt2_data.time_secs``
  - **NOTE:** Since this is raw data (i.e. each ion hit does not yet correspond to one event),
    need to specify a ``channel_ids`` to get just one trigger value per ion hit

Ion-Ion Interaction Data Cuts
-----

- We want to minimize any ion-ion interaction effects in the trap on results,
  so we discard any data where > N total ions were in the trap together
  - N corresponds to ``ion_ion_cut_min``
- In practice, we throw away data with > N events in the same trigger
  window
- ONLY works with CAEN 25ps TDC Data

Determining Ion Rates
-----

- We want to determine the rate at which ion events are occurring
- We determine a gating time t (``gating_rate`` [ms]) and determine the number
  of events that occur within each gating time window across the file
- i.e. events between 0-t, events between t-2t, etc.
- We approximate the global event time for each event as the total trap time
  (``trap_time``) times the trigger number of that event
- We record the approximate global gate time, the number of events, and the
  rate of those events (``num_events/t``) for each gating time window

Pairing Final and Reference Files
-----

- For each final file (``Tacc > 0``), we need to pair it with the reference file (``Tacc = 0``) closest to it in time

- We determine the midpoint time of each file:
  - i.e. ``file_start + (file_end - file_start)/2``
  - ``file_start`` is ``event_time[0]`` and ``file_end`` is ``event_time[-1]`` from ``titan_data/mpet/__init__.py``

- We find the difference between the midpoint time of the final file and all reference files, and pair the actual file with the reference file of smallest difference


Output: Ion Rate .csv
-----

- A .csv file is printed with the recorded ion rate data:
  - Column 1: Approximate Global Gate Time [ms] (gating window number)*t
  - Column 2: Number of Events in each Window [int]
  - Column 3: Rate of Incoming Events in each Window [events/ms] (``num_events/t``)
  - Saved as ``original_MIDAS_filename.mid_ion_rate.csv``

Output:  Main Data .csv
-----

- A .csv is printed with the data described on slides 7-9:
  - Column 1: X Position [mm]
  - Column 2: Y Position [mm]
  - Column 3: Time-of-Flight [s]
  - Column 4: Trigger Number [int] (CAEN) OR Timestamp [s] (VT2)
  - Saved as ``original_MIDAS_filename.mid_.csv``

- Main Data and Ion Rate CSVs are created for each MIDAS file in the input directory 

Output:  File List .csv
-----

- A .csv is printed with:
  - Column 1: Addresses of Main File CSVs
  - Column 2: Accumulation Times of each File [s]
  - Column 3: Reference File Assignment for each Final File
    - If file is a reference file, this is blank
    - Numbering of associated references corresponds to ordering of references in first column
  - Saved as ``inputDirectoryName_firstTaccFile_List.csv``
    - i.e. ``Test_CAENOffset_0.03File_List.csv``
  - Saved OUTSIDE the input directory
