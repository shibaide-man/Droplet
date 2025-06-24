# Droplet_Data_Machine_Learning
Use analyed data and apply machine learning for Droplet Project

This project is an initiative supported by the LDRD grant number 22-059, focusing on the development of a droplet reactor system for autonomous growth of nanomaterials. The repository is spearheaded by Yugang Zhang, who can be reached at [yuzhang@bnl.gov](mailto:yuzhang@bnl.gov). This project is conducted at the Center for Functional Nanomaterials (CFN), Brookhaven National Laboratory (BNL).

## Repository Contents

This repository houses the machine learning for droplet project. Here is a breakdown of the  components and their respective functions included:

- **Codes:**
  - DropBox.py: dropbox communication for upload and download.
  - Message.py: send email or text
  - Utility.py / general_funcs.py: general funcs
  - logger.py: logger
  - GP_Adaptive directory: underlying GP_Adaptive code
  - **Droplet_MachineLearning.py: "Droplet_machine learning codes (class)"**

 
- **Pipelines:**
  - Droplet_ML_test.ipynb: Template pipeline for droplet_machine_learning
  - test_dropbox.ipynb: Test pipeline (example) for creating fake analyzed results and "analyzed_dict.npz" & "Done_dict.npz" in DropBox.

- **Past:**
  - Past codes and pipelines for batch and non-batch system.
 
  
## Getting Started

This guide provides instructions for real-time machine_learning during autonomous synthesis using UV-Vis spectroscopy in a chemical lab or SAXS/WAXS at the sychrontron beamline. Follow these steps to get started:


### Initial Setup
1. **Run "Droplet_MachineLearning.py"**:
   - Create new jupyter notebook and run "Droplet_MachineLearning.py".
     ```
     %run -i Codes/Droplet_MachineLearning.py
     ```
2. **Check and Modify fundamental parameters**:
   - bounds : np.array of reagent volumes, Temperature and reaction time.
   - vol_precision: volume minimum distance between conditions (unit: uL) 
   - T_precision: reaction temperature minimum distance between conditions (unit: Celsius)
   - t_precision: reaction time minimum distance between conditions (unit: minutes)
   - vol_max: the maximum volume of single droplet (unit: uL)
   - vol_min: the minimum volume of single droplet (unit: uL)
   - init_Tt: list of initial T and t condition for batch system.
   - seed: random seed number 
   - init_exp_num: the initial number of experiments (for the initial experiments per batch)
   - init_bat_num: the initial number of batch (for the initial batch number)
     - **Typically for batch system:**
       - init_exp_num = the number of initial experiments (or the number of old data)
       - init_bat_num = 1
     - **For non-batch system (one-by-one):**
       - init_exp_num = 1
       - init_bat_num = the number of initial experiments (or the number of old data)
   - adaptive_num: the minimum number of data for "adaptive_model" GP training
   - outDir = '/home/group/Droplet/Results/UV_Auto_Jin/', #UV_Auto_Jin #Test_Auto_Jin
   - outDir_BK = '/home/group/Droplet/Results/UV_Auto_Jin/BK_dict/', #UV_Auto_Jin #Test_Auto_Jin
   - Recipe_filename =     'Recipe_dict.npz' ,    #  Recipe droplet dictionary from BoTorch
   - Done_filename =     'Done_dict.npz' ,        #  Generated droplet dictionary  
   - Anlysis_filename =     'Analyzed_dict.npz',  #  UV-Analyzed droplet dictionary
   - logfn = 'DropletMachineLearning_Run1.log',
   - dropbox_dir='/CFN/1L03/', 
   - dropbox_archive_dir='/CFN/1L03_BK/', 
     ```
      DML = Droplet_MachineLearning(bounds = np.array([
                        [1.0, 30.0],  #channel one - NCit, 16mM
                        [-30.0, 30.0],  #channel three, fourth - HCL/NaOH, 10mM/10mM
                        [1.0, 30.0],  #channel five - HAu, 2mM  ;  ##channel six is blue                   
                       [80.0, 100.0],   # T bounds
                        [ 2.0, 40.0],     # t bounds                   
                       ] ),
        vol_precision = 1,
        T_precision = 5,
        t_precision = 5,
        vol_max = 40,
        vol_min = 2,
        init_Tt = [100,10],
        seed = 123, 
        init_exp_num = 5, 
        init_bat_num = 1,
        adaptive_num = 10,
        outDir = '/home/group/Droplet/Results/UV_Auto_Jin/', #UV_Auto_Jin #Test_Auto_Jin
        outDir_BK = '/home/group/Droplet/Results/UV_Auto_Jin/BK_dict/', #UV_Auto_Jin #Test_Auto_Jin
        
        Recipe_filename =     'Recipe_dict.npz' ,    #  Recipe droplet dictionary from BoTorch
        Done_filename =     'Done_dict.npz' ,        #  Generated droplet dictionary  
        Anlysis_filename =     'Analyzed_dict.npz',  #  UV-Analyzed droplet dictionary
        logfn = 'DropletMachineLearning_Run1.log',
        dropbox_dir='/CFN/1L03/', 
        dropbox_archive_dir='/CFN/1L03_BK/', )
     ```

3. **Start the machine_learning**:
   - Start active learning of ML.
     - num_inside_batch: the number of experiments in one batch.
       - for non-batch system: num_inside_batch =1
     - N_batch: total number of batch size => Total number of experiments = num_inside_batch*N_batch
     - AF: acquisition function -> ['qEI', 'EI', 'PI', 'qPI', 'UCB', 'qUCB', 'KG', 'MES']
     - UCB_beta: ucb beta
     - old_data_fn: the npz filename of old_data.
     ```
     DML.Run_ML_batch(num_inside_batch=5, N_batch=20, AF= 'UCB', UCB_beta=100, old_data_fn=None)
     ```
   - If want to stop the code and resume active learning.
     ```
     DML.resume_active_learning()
     ```
