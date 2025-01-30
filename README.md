# Scripts for data processing for “Enhancing Transcranial Ultrasound Stimulation Planning with MRI-Derived Skull Masks: A Comparative Analysis with CT-based processing”

This collection includes the scripts and steps required to reproduce the analyses presented in the publication Zadeh *et al.* 2024, *Enhancing Transcranial Ultrasound Stimulation Planning with MRI-Derived Skull Masks: A Comparative Analysis with CT-based processing*, [DOI:10.1088/1741-2552/adab22](https://doi.org/10.1088/1741-2552/adab22). Below is a detailed description of each file and the procedure to execute the analyses.

## Directory overview:
*	The repository contains the following scripts to reproduce the results.
*	`0_eeg_normals.py`: Generates EEG normal trajectories for each location.
*	`1_BabelBrain_All_Conditions.ipynb`: Runs ultrasound simulations for MRI-based and CT-based processing.
*	`2_TemplateCalcStats.ipynb`: Calculates pressure metrics and area statistics.
*	`3_Dice_Coeff.ipynb`: Computes Dice coefficients for simulations.
*	`SELECTED_EEG.csv`: Contains selected EEG locations for analysis.
Note: Specific locations (e.g., Fp1, Fp2, Fpz) were excluded due to complications such as passage through air-filled structures, proximity to the ear, or defacing issues, leaving 54 locations for analysis.
*	MRI and CT data: Defaced data can be downloaded from https://doi.org/10.5281/zenodo.7894431, please take note of the paths.
* The directory [MRI_Custom_Settings](MRI_Custom_Settings) contains the improved atlas and settings for a better skull segmentation with CHARM.

# Step-by-Step Guide
## 1. Check requirements
Verify you have the minimum hardware requirements to run [BabelBrain](https://proteusmrighifu.github.io/BabelBrain/installation.html)
## 2. Install SimNIBS 4.1.0
Use installer for [v4.1.0](https://github.com/simnibs/simnibs/releases/tag/v4.1.0) version. Take note of SimNIBS path; for example,/Users/john/Applications//SimNIBS-4.1
## 3. Clone BabelBrain source code
Clone source code from https://github.com/ProteusMRIgHIFU/BabelBrain. While the latest version should be compatible with the scripts, we recommend using version 0.3.5 to ensure the reproducibility of the results in the manuscript. You can do a checkout of the tag `0.3.5` to select the version.
## 4. Create a conda environment with the recommended YAML file
Create a conda environment using the most appropriate YAML file for your computer type; for example, use `environment_mac_arm64-39.yml` for macOS with an Apple Silicon processor.
## 5. Prepare MRI and CT Data 
After downloading the MRI and CT data, delete the included directories of previous calculations with the CHARM tool as they were produced with version 4.0.0 of the tool. The directories to delete have the prefix `m2m`, for example `m2m_SDR0p55`.  Process the T1- and T2-weighted scans using the CHARM tool in SimNIBS 4.1 to generate the m2m folder for each participant. Follow these steps:
### 5.a. Default Settings
Run the following command:

`charm <id> T1.nii.gz T2.nii.gz –forceqform`
### 5.b. Custom CHARM FAT Atlas Integration. 
The custom CHARM FAT atlas files are in the directory [MRI_Custom_Settings](MRI_Custom_Settings) 
### 5.d. Setup:
#### 5.d.1. Place the custom atlas in:
`<path_to_simnibs>/simnibs/segmentation/atlases/`

Download `settings_fat.ini` and `shared_gmm_fat.txt` and place them together in a directory.
#### 5.d.2.	Run with Custom Atlas:
`charm <subid> T1.nii.gz T2.nii.gz --usesettings <path_to_settings_fat.ini> --noneck –forceqform`
## 6. Generate EEG Normals
Run `0_eeg_normals.py` script with a conda environment based on SimNIBS specifying the paths to the SimNIBS output for the dataset. For example
    
`conda activate <path_to_simnibs>/simnibs_env/`

`python 0_eeg_normals.py ~/Documents/TempForSim/SDR_0p42/m2m_SDR_0p42/SDR_0p42.msh ~/Documents/TempForSim/SDR_0p42/m2m_SDR_0p42/eeg_positions/EEG10-10_Neuroelectrics.csv SDR_0p42_eeg_normals.csv`
    
This generates normal trajectories for each EEG location and participant. Run this for each of the 5 datasets.

## 7. Run Ultrasound Simulations
Open `1_BabelBrain_All_Conditions.ipynb`:
* Update the “Additions to path” section with the path to the BabelBrain v0.3.5 folder.
* Run the notebook to perform simulations for MRI-based and CT-based processing at frequencies of 250, 500, and 750 kHz.

## 8. Calculate Metrics
### First metrics
In `2_TemplateCalcStats.ipynb`:
1.	Update the following parameters:
    * `inputcsv`: Path to the trajectory file (e.g., SDR_0p55_eeg_normals.csv).
    * `BasePath`: Path to the m2m folder generated using CHARM.
    * `T1W`: Path to the T1-weighted image.
    * `Frequency` and `PPW` (e.g., 500 kHz, PPW=6).
1. Run the notebook to calculate:
    *   Maximum Pressure
    *   Area Size
    *   Centroid coordinates (0, 1, 2)
    *   Outputs are saved in an Excel file
### Second metrics - Dice Coefficients
In `3_Dice_Coeff.ipynb`:
1.	Specify the paths to the BabelBrain folder and the BasePath for the participant.
2.	Adjust the frequency and PPW settings (e.g., 500 kHz, PPW=6).
3.	Run the notebook to compute Dice coefficients between MRI-based and CT-based simulations.
