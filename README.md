# CFN_Droplet_Reactor_Controls

This project is an initiative supported by the LDRD grant number 22-059, focusing on the development of a droplet reactor system for autonomous growth of nanomaterials. The repository is spearheaded by Yugang Zhang, who can be reached at [yuzhang@bnl.gov](mailto:yuzhang@bnl.gov). This project is conducted at the Center for Functional Nanomaterials (CFN), Brookhaven National Laboratory (BNL).

## Repository Contents

This repository houses the control software developed in Python for managing the hardware components of the droplet reactor system. Here is a breakdown of the hardware components and their respective control mechanisms included:

- **Syringe Pumps:**
  - New Era
  - KEM
- **Flow Management:**
  - Flow Sensing
  - Flow Switch (M-switch, two-way swith )  -
  - Flow EZ (Pressure Monitoring)
- **Temperature Control:**
  - Lakeshore
- **Power Management:**
  - NetBooter
- **Spectroscopy:**
  - UV-VIS Spectroscopy
- **Communication:**
  - Dropbox Apps
  - sftp
- **Motor:**
  - MDrive Motors
- **Camera:**
  - Vimba  (to be used at the CFN)

## Getting Started

This guide provides instructions for setting up and conducting autonomous synthesis using UV-Vis spectroscopy in a chemical lab environment. Follow these steps to get started:

### Initial Setup on HP Windows Control Computer

1. **Launch Anaconda Powershell**:
   - On the HP Windows Control Computer (LNC-168313), click on the `Anaconda Powershell Prompt (Anaconda3)` to open two separate terminal windows.

2. **Navigate to Project Directory**:
   - In each terminal, navigate to the `Droplet_Controls_Update` folder using the command:
     ```
     cd path/to/Droplet_Controls_Update
     ```
   - Replace `path/to/` with the actual path to the folder.

3. **Start IPython**:
   - In both terminals, start IPython by typing:
     ```
     ipython
     ```

### Conducting UV Measurements

1. **Load UV Measurement Application**:
   - In one of the terminals designated for UV measurements, load the UV application script:
     ```
     %run -i UV_Controls.py
     from UV_Controls import *
     ```

2. **Configuration Adjustments**:
   - **Important**: Ensure you have updated the `config.py` with the correct paths and other necessary control settings before proceeding.
   ```
   - from config import get_configs    
   - confs = get_configs()
   - confs contains configuration setting for: Ocean (UV-Vis), FTP, Dropbox, LakeShore, Netboot, Kem, New Era, FGT,  Solutions,  
    ```

3. **Controlling the UV Lamp**:
   - To turn the UV lamp on or off, use the following commands in the IPython shell:
     ```
     turn_on_uv_lamp()
     turn_off_uv_lamp()
     ```

4. **Example UV Data Collection**:
   - To collect UV data, use the command:
     ```
     oc = setup_Ocean( integration_time = 20, ref_fn = 'ref.npy', dark_fn = 'dark.npy', outDir = None )
     # oc.take_dark( filename='dark.npy')
     # oc.take_ref( filename='ref.npy')
     collect_uv_data( oc )

     ```

5. **All together for UV Data Collection**:
   - To setup and collect UV data, use the command:
     ```
     %run -i UV_Controls.py
     turn_on_uv_lamp()
     oc = setup_Ocean( integration_time = 20, ref_fn = 'ref.npy', dark_fn = 'dark.npy', outDir = None )
     collect_uv_data( oc )
     oc.All_xz_scan( step_size = 0.5, F9_setp = [51, 66] ):
     UV_manual_batch(oc, filename='AutoRun1', UV_interval_time =2 , sid0 =0, batch_th=0, motor_control =True, duration = 3*24*60*60)
     turn_off_uv_lamp()

     ```    

     - This command collects UV data with specified parameters like repeat count, filenames, waiting times, sleep time, and duration.


### FTP Control  
      ```
      from SFTP import (FTP)
      ftp  = FTP()
      ftp.put( filename )
      ftp.get( filename )

      ```


### Dropbox Control    
      ```
      from DropBox import (CustomDropbox)
      DB = CustomDropbox()
      DB.upload_file_to_dropbox( 'Done.txt', local_dir = DB.local_dir + 'Data/', dropbox_dir = DB.dropbox_dir + 'Data/',   overwrite=True,)
      DB.download_file_from_Dropbox( dropbox_file='Done.txt', local_dir = DB.local_dir + 'Data/',  dropbox_dir = DB.dropbox_dir + 'Data/',)     

      ```

### Temperature Control    
      ```
      from LakeShore import (LakeShore)
      L = LakeShore(   )
      L.HT( 40) #set 40 C and heat up
      L.stopT() #stop heating

      ```
      - channel 2 temp cannot be read - 20231208

### NetBoot Control   
      ```
      from NetBooter import NetBooter_Control
      NB = NetBooter_Control(  )
      NB.power_on( 5 )  #turn on the channel 5, currently for UV-lamp
      NB.power_off( 5 )  #turn off the channel 5, currently for UV-lamp

      Or using this code
      from NetBooter_Controls import *
      turn_on_uv_lamp()
      turn_off_uv_lamp()

      ```   


### Kem Control    
      ```  
      from KEM_Pump import   KEM_Pump
      kem = KEM_Pump()
      kem.init_pumps( [1] )
      kem.setp( vol=10, rate=-80,  pump=1, home_port=None, cur_port = None,  )
      kem.getp( pump = 1 )
      kem.start(   pump =1, check=True, wait_until_ready = True,  )  

      ```   

### NewEra Control   
      ```
     from NewEra_Pump import NewEra_Pump
     NE = NewEra_Pump( init_pumps=True )
     NE.reset_vol_record_dict_file(  pumps=[1,2]   )     
     NE.setp(    vol=10, rate=80,  pump=1, home_port=None, cur_port = None,  )
     NE.getp( pump=1 )
     NE.start( pump=1, check=True, wait_until_ready = True, lag_time = 0   )
     NE.close()
     ```

### Fluigent Control   
     ```
     from FluigentPy import customFGT
     fgt = customFGT()
     fgt.get_controller_info()
     fgt.set_MSW_postion( 1 )
     fgt.set_valve_position_by_type( valve_type = 'MSW', valve_position=1 )  #TSW1, TSW2,
     fgt.get_sensor( 1 )
     fgt.set_pressure( 1, 10  )
     fgt.close_channel( 1 ) ; #f
     ```
     - DLL is necessary??? not working (line 8-9) -> uncomment for this line(not sure it is okay) : 20231208
     - Code is not working for "line 70-72" -> modified (not sure it is okay) : 20231208


### Solution Control
     ```
     from Solution_Controls import ( Pump )     
     pump = Pump(  )  
     pump.NE.init_pumps()
     pump.KEM.init_pumps()
     pump.inject_one_solution( solution_index=1, target_volume=10, flow_rate=40,  
                            backlash_time=10, close_EZ=True, RUN=True, EZ_sleep=3,
                            backP=True, backPV=300, verbose=True)  
     ```                       
     - NewEra 5 -> doesn't stop after injection -> Add stop function after injection : 20231211

### Power Control
     ```
     from NetBooter_Controls import (turn_on_uv_lamp, turn_off_uv_lamp, turn_on_degasi, turn_off_degasi, turn_on_gas_generator, turn_off_gas_generator )
     turn_on_uv_lamp()
     turn_off_uv_lamp()
     turn_on_degasi()
     turn_off_degasi()
     turn_on_gas_generator()
     turn_off_gas_generator()

     ```  

### Motor Control
     ```
     #modify the MDmotor_Controls.py, for travel_limit, position, etc.
     from MDmotor_Controls import mx, mz
     mx.caget('p')
     ```       

### Reactor Control
     ```
     from Reactor import Reactor
     reactor = Reactor(tubing_type='glass', flow_rate=80, flow_start='top')     
     reactor.reactor_dict
     new_position, new_flow_rate, new_volume = reactor.find_new_position(target_time=20, set_point='F7')  

     ```  

### Synthesis
     ```
     %run -i Synthesis.py
     DE.run_batch (self, flow_rate ,num_drop_per_batch=3, MARKER_vol=180,  Additional_push_vol=3000,  manual=False, num_batch=10 , old_recipes =0  )

     ```       



### Conducting Droplet Experiment with in-situ UV-vis characterization (UV control + Synthesis)

1. **Check config.py**:
  - [Very important] Check all paras in config.py file.

2. **Load Anaconda kernel for UV characterization**:
    ```
    - Go to an appropriate path
    - %run -i Characterizations.py
    - DM = DropletMeasurements()
    - turn_on_uv_lamp()
    - Load or save reference
        DM.oc.take_ref()
        DM.oc.take_dark()
    - if motor control, load or save "Beam_dict.npz"
        DM.All_xz_scan( step_size = 0.5, F9_setp = [51, 66] )

    case1) batch_system (manual or autofly)
      - DM.UV_batch(filename='Run1', UV_interval_time =2 , sid0 =0, batch_th=0, motor_control =True, duration = 3*24*60*60)

    case2) no_batch_system,
      - DM.collect_uv_data( repeat=1, filename = 'Test', wait_time=30*60, sleep_time = 2, duration = 24*60*60 , sid0 =0)

    - turn_off_uv_lamp()
    ```
2. **Load NEW Anaconda kernel for Synthesis**:
    ```
    - Go to an appropriate path
    - %run -i Synthesis.py
    - DE = DropletExperiment( logfn = 'DE_Run1',  init_pumps=False)
    - DE.reset_class_paras(mode_rec = 'water_dye', logfn = 'DE_0112_Run1', solution_indexes_init_flow =  [  1,2,3,6,7,8,9 ], solution_volumes_init= 10,
                  Recipe_MW_order = [1,'ph',2], pH_MW = [6,7], buffer_MW = [3], buffer_order = 1, One_droplet_ul= 40, Oil_ul= 40, push_solution_index = 9)
    - DE.KEM.init_pumps()
    - DE.NE.init_pumps()
    - DE.pump.withdraw_kem_solutions( solution_indexes=[3,6,7,8,9,10], target_volumes=[9900,9900,9900,9900,49900,49900], flow_rates=10000)

    case1) batch_system (manual or autofly)
          if manual_batch (No autofly experiment):
            manual = True,
          elif auto_fly batch experiment:
            manual = False,
          if using existing old data as a first batch:
            old_recipes = the number of old Data
          else:
            old_recipes = 0

      - DE.run_batch (flow_rate=80 ,num_drop_per_batch=3, MARKER_vol=180,  Additional_push_vol=3000,  manual=False, num_batch=10 , old_recipes =0  )

    case2) no_batch_system,
        if systematic study, no autofly, (i.e., generating droplet in the order of recipes):
          manual = True,
        elif auto_fly experiment (i.e., generating droplet from the most recent recipes):
          manual = False
      - DE.run_no_batch(flow_rate=80 ,MARKER_vol=180,  manual=False, old_recipes =0, Temp = 100, num_NP = 1000  ):

    - DE.reset_class_paras() (when we conduct another experiment )
    - DE.shutdown()
    ```

3. **[ETC] if want to simulation for coding**:
    ```
    1) Solution_Controls.py
      - def inject_one_solution ->  
          # pump.start(pump=pump_index, check=True, wait_until_ready=wait_until_ready, lag_time=0)
          # if wait_until_ready:
          #     if backP:
          #         self._handle_backlash(backlash_time)
          #     else:
          #         time.sleep(backlash_time)


    2) Characterizations.py
      find all these commands and "comment out" them.
      - # oc.take_data
      - # move_position

    3) Synthesis.py
      find all these commands and "comment out" them.
      - # self.Lakeshore.HT()
      - def check_syringe_volume ->
        # self.pump.withdraw_kem_solutions(solution_index, v_dis-100, flow_rates=10000, RUN=True, verbose=True)
        # self.pump.withdraw_kem_solutions(solution_index, v_dis-100, flow_rates=10000, RUN=True, verbose=True)
      - def check_Temperature_befor_push ->
        while np.abs(L.get_temperature() - self.Target_temp) > 100 -> 1
      - def save_done_dict(self, Done_dict, fp = 'Done_dict.npz') ->
        # Done_dict_save_tf = get_current_time(pattern='_%Y_%m_%d_%H_%M')
        # try_save_npz(os.path.join( self.outDir ,fp[:-4] + Done_dict_save_tf + fp[-4:]), Done_dict)
        # try_save_npz(self.outDir + 'Done_dict' + Done_dict_save_tf + '.npz', Done_dict, n_loop=3, slep_time=0.02)
    ```


Follow these steps carefully to ensure a successful setup for your autonomous synthesis experiments using UV-Vis spectroscopy.

## Contributing

(Information about how to contribute to this project)

## License

(Information about the project's license)

## Acknowledgments

Special thanks to the contributors and supporters of this project, including LDRD-22059.
