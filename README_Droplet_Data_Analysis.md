# Droplet_Data_Analysis
Data analysis for Droplet Project

This project is an initiative supported by the LDRD grant number 22-059, focusing on the development of a droplet reactor system for autonomous growth of nanomaterials. The repository is spearheaded by Yugang Zhang, who can be reached at [yuzhang@bnl.gov](mailto:yuzhang@bnl.gov). This project is conducted at the Center for Functional Nanomaterials (CFN), Brookhaven National Laboratory (BNL).

## Repository Contents

This repository houses the data analysis for droplet project. Here is a breakdown of the  components and their respective functions included:

- **Codes:**
  - DropBox.py: dropbox communication for upload and download.
  - Message.py: send email or text
  - Utility.py / general_funcs.py: general funcs
  - logger.py: logger
  - XQu_UV_model.py: ML model for droplet recognition for UV data
  - nanoparticle2 directory: underlying SAXS analysis code
  - UV_analysis.py: UV analysis code
  - SAXS_analysis.py: SAXS analysis code
  - **Droplet_analysis.py: "Droplet_analysis codes (class)"**
 
- **Pipelines:**
  - Droplet_analysis_pipeline_system.ipynb: Template pipeline for droplet_analysis
  - Droplet_analysis_pipeline-UV_simulation.ipynb: Test pipeline (example) for UV experiment using correlation method.
  - Droplet_analysis_pipeline-SAXS_simulation_waxsCorrelation.ipynb: Test pipeline (example) for SAXS/WAXS experiment using correlation method.
  - Droplet_analysis_pipeline-SAXS_simulation_Intensity.ipynb: Test pipeline (example) for SAXS/WAXS experiment using intensity difference method.
  - Droplet_analysis_pipeline_system_test.ipynb: Test pipeline (example) for UV experiment.

## Getting Started

This guide provides instructions for real-time data analysis during autonomous synthesis using UV-Vis spectroscopy in a chemical lab or SAXS/WAXS at the sychrontron beamline. Follow these steps to get started:

### Initial Setup
1. **Check and Modify fundamental py files**:
   - Check and modify fundamental py files.
     - DropBox.py
     - UV_analysis.py  
     - SAXS_analysis.py
     - Droplet_analysis.py
       
2. **Run "Droplet_analysis.py"**:
   - Create new jupyter notebook and run "Droplet_analysis.py".
     ```
     %run -i Codes/Droplet_analysis.py
     ```

3. **Load and set references for droplet recognition/analysis**:
   - Load the measured references.
   - Save the Ref dictionary.
   - For UV data analysis,
     ```
     Ref ={}
     Ref['marker'] = [ref_x, ref_y]
     ```
     ref_x, ref_y: 1d array of x and y for droplet recognition and analysis.
   - For SAXS/WAXS data analysis,
     ```
     Ref = {}
     Ref['marker'] = [ref_x, ref_y]
     Ref['lmfit_ref'] = [saxs_x, saxs_y]
     ```
     ref_x, ref_y: 1d array of x and y for droplet recognition.
     saxs_x, saxs_y: 1d array of x and y for data analysis.
                 
     For example, (typical WAXS/SAXS measurements)
         Ref['marker'] = WAXS_q, WAXS_iq (for oil)
         Ref['lmfit_ref'] = SAXS_q, SAXS_iq (for water)
     
4. **Initial_setup for filename, sid, and directories**:
   - Setup the special filename for experiments. 
   - Setup data directory.
   - Find where the sids in the data filename.
     ```
     Run ='Cir_Avg_YZ_Run1_Cu'
     inDir_saxs =  '/home/group/NSLSII_Data/SMI/2023_Cycle3/AutoRun_2023C3/310844_Zhang1/Results/SAXS/'
     npz_outDir = '/home/group/Droplet/Results/UV_Auto_Jin/'
     flist[0][-26:-20] = '394274' -> sid = [-26:-20]
     ```
     
5. **Setup output function**:
   - Setup output function for machine learning output.
   - For UV analysis:
     parameters = [amp, cen, wid] (amplitude, center, width)
   - For SAXS analysis:
     parameters = [r/10, s] (radius nm, sigma)
     ```
     def output_func(dataA ,  parameters):
       return 1/parameters[1]
     ```

### Start Droplet_analysis

1. **Initialize DropletAnalysis setup**:
   - mode_rec(str): 'UV' ,'SAXS'
   - inDir_main(str): path where the data for analysis is saved.
   - inDir_rec(str): path where the data for droplet recognition is saved.
   - For 'UV' measurement, 
      inDir_main = inDir_UV
      inDir_rec = None
   - For 'SAXS' measurement,
      inDir_main = inDir_SAXS
      inDir_rec = inDir_WAXS. (for WAXS correlation)
   - outDir(str): path where the npzfile will be saved. 
   - outDir_BK(str): path where the npzfile_BK (backup) will be saved.
   - logfn(str): log filename           
   - Datasave: True- npz file will be saved
   - raise_error: True- the code will stop if there is any error
   ```
   For UV,
   DA = DropletAnalysis(mode_rec = 'UV', Ref= Ref, inDir_main = inDir_uv, inDir_rec = None,local_dir_Log = npz_outDir,
               outDir=npz_outDir, outDir_BK=npz_outDir , logfn = 'DropletAnalysis_Run1.log',
                  Datasave = True, raise_error=False)
   
   For SAXS/WAXS,
   DA = DropletAnalysis(mode_rec = 'SAXS', Ref= Ref, inDir_main = inDir_saxs, inDir_rec = inDir_maxs,
                   local_dir_Log = npz_outDir,
               outDir=npz_outDir, outDir_BK=npz_outDir , logfn = 'DropletAnalysis_Run2.log',
                  Datasave = True, raise_error=False)
   ```
     
2. **Start the droplet_analysis**:
   - Run real-time analysis for Manual/autofly batch/no_batch experiment  
      - label(str): special file name to sort out specific experiment data in inDir directory
      - where_sid(list):  [sid_start_location in filename, sid_end_location in filename] 
          They are VERY IMPORTANT to sort out the data by sid number
      - skip_index: skip the first number of data
      - rec_method(str): 'cor', 'int', 'model', Droplet recognition method
          cor: correlation data with ref data
          int: find intensity level of data for specific region
          model: UV ML model  (made by collaborator XQu)
      - marker_smaller_than_threshold(T/F): 
          For correlation method,  score > thresh_marker -> "Marker"
          Typically intensity method, marker intensity is lower than NP intensity:
                marker_smaller_than_threshold = True
                -score > -thresh_marker -> "Marker"
   
      **********HIGHLY-SENSITIVE**********
      - thresh_marker: threshold of marker in order to determine whether the data is marker or not.
      - nlook(int): get the averaged boundary dict value from the range of (-nlook,nlook+1)
      - first_marker_min_len(int): the threshold for the first marker length
      - long_marker_len(int): the threshold for the long_marker length
      - marker_min_len(int): the threshold for the marker length
      - glitch_len(int): join two markers if distance between two markers are less than "glitch_len"
      ************************************
      - num_drop_per_batch: it will create long_markers_index(num_drop_per_batch)
          if batch_system:
              num_drop_per_batch = the number of droplets in one batch
              long marker indexes (droplet index starting from 0), for example [9,19,...] -> 10th, 20th,..
          if not batch_system:
              num_drop_per_batch = 0
              long marker indexes are already assigned as described in "Utility.py"
      - output_func: 
          function for output
          for example) 
              def output_fun(dataA ,  parameters):
                  return parameters[0]
      - init_step1: Initialize batch_paras for step1 boundary dict (correlation of data using reference)
      - init_step2: Initialize batch_paras for step2 droplet dict (recognition)
      - init_step3: Initialize batch_paras for step3 SAXS (or UV) dict (curve-fitting)
      - init_step4: Initialize batch_paras for step4 Analyzed dict (match with recipe from 'Done_dict.npz' and create 'Analyzed_dict.npz')
   ```
   For UV,
   DA.Run_Analysis(label = 'v1', where_sid = [-9,-4], skip_index=22000 ,rec_method = 'cor',
              marker_smaller_than_threshold = False,
              thresh_marker=0.4, nlook = 0, first_marker_min_len=10, long_marker_len=21, marker_min_len=2,
              glitch_len = 2, num_drop_per_batch= 8, 
              output_func=output_func ,
              init_step1 = True, init_step2= True, init_step3=True, init_step4=True, 
              old_recipes=0, duration=24*60*60, )
   
   For SAXS/WAXS,
   DA.Run_Analysis(label = 'Cir_Avg_YZ_Run1_Cu', where_sid = [-26,-20], skip_index=150 ,rec_method = 'cor',
              marker_smaller_than_threshold = False,
              thresh_marker=0.8, nlook = 0, first_marker_min_len=5, long_marker_len=20, marker_min_len=2,
              glitch_len = 0, num_drop_per_batch= 15, 
              output_func=output_func ,
              init_step1 = True, init_step2= True, init_step3=True, init_step4=True, 
              old_recipes=2, duration=24*60*60, )
   ```

### Check Droplet_analysis

1. **Load pyScatt packages**:
   ```
   from pyScatt.packages import *
   ```
   
2. **Load log file**:
   ```
   print_log_file('/home/group/Droplet/Results/UV_Auto_Jin/SAXS_DropletAnalysis_Run2.log')

   or

   with open(log_file_path, 'r') as file:
    # Read the entire content of the file
    log_content = file.read()
   print(log_content[:10000])
   ```
   
3. **Check boundary and droplet dict**:
   - DA.boundary_dict: boundary dictionary
   - DA.droplet_dict: droplet dictionary
   ```
    %matplotlib notebook
    i =0
    index, avg, std =[],[],[],
    avg_nlook =[]
    nlook =0
    thresh_blue = DA.thresh_marker
    
    UV_boundary_dict = DA.boundary_dict
    UV_droplet_dict = DA.droplet_dict
    BLUE_MARKER= DA.long_markers_index
    
    
    for sid in list(UV_boundary_dict.keys()):
        index.append(UV_boundary_dict[sid][0])
        avg.append(UV_boundary_dict[sid][2])
        std.append(UV_boundary_dict[sid][2])
    
    
    droplet_dict= UV_droplet_dict
        
    short_droplet_index =[]
    long_droplet_index = []
    for dn in list(droplet_dict.keys()):
        if droplet_dict[dn][-1]:
            long_droplet_index.append(int(droplet_dict[dn][2]))
        else:
            short_droplet_index.append(int(droplet_dict[dn][2]))
    
        
    fig, ax = plt.subplots(figsize=(8,4)) 
    plot1D(x= index, y=avg ,ax=ax, m='D', ls='--', c='k', legend='correlation')
    ax.axhline(y = DA.thresh_marker, c ='green', ls='--')
    for i in range(len(short_droplet_index)):
        ax.axvline(x = short_droplet_index[i], ymin=0, ymax = 0.95)
    
    for i in range(len(long_droplet_index)):
        ax.axvline(x = long_droplet_index[i], ymin=0, ymax = 0.95, c='r', )
    
    for i in list(droplet_dict.keys()):
        if i in BLUE_MARKER:
            c= 'r'
        else:
            c= 'b'
        ax.text(  int(droplet_dict[i][2]), thresh_blue,  (i), horizontalalignment='center', size=10, color=c)
    
    ax.set_title('Auto_Mapping_Droplets--Run=%s'%Run)
    ax.set_xlabel('Frame')
    ax.set_ylabel('Correlation')
   ```

3. **Plot for each droplet**:
   - DA.Fitting_dict: fitting dictionary
   - DA.Analyzed_dict: analyzed dictionary
   ```
   %matplotlib inline
   Dict = DA.Fitting_dict['data_dict']
   Observation =DA.Fitting_dict['output']
   Recipe = DA.Analyzed_dict
   f_size = len(Dict)//3 +1
  
   fig = plt.figure( figsize=[9,2*f_size])
   for i in range(len(Dict)):
      ax = fig.add_subplot(f_size,3,i+1)
      key = list(Dict.keys())[i]
      #_x , _y = Dict[key][0], Dict[key][1]  #raw
      _x , _y= Dict[key][-1][0], Dict[key][-1][1] #bg
      yi= find_index(_y,0.12)
      fit_x, fit_y = Dict[key][-1][2], Dict[key][-1][3]
      plot1D(x=_x[:yi], y=_y[:yi], marker='o',markersize=1, linewidth =1, logy=True,legend= '%.2f'%(Observation[key]), ax=ax)
      plot1D(x=fit_x, y=fit_y, logy=True,marker='o',markersize=1, linewidth =1, color='r', ax=ax)
  
      ax.set_title('%s + %s th %s'%(i, DA.initial_NP, Recipe[i+DA.initial_NP]['conditions']), fontsize=6)
      ax.set_ylabel('Abs')
      #ax.set_ylim(0,0.2)
      ax.set_xlabel('Wavelength')
      ax.legend(fontsize=5)
      
   plt.tight_layout()
   ```

   
## Contributing

(Information about how to contribute to this project)

## License

(Information about the project's license)

## Acknowledgments

Special thanks to the contributors and supporters of this project, including LDRD-22059.
