# hal-cia402
HAL Interface for CiA402 Devices,

this component acts as a glue layer between hardware to Hal modules like Ethercat, CAN-Bus or others.
It translates raw IO Data from the PDOs to the common linuxcnc Hal pin structure and has build in logic
for the CiA402 State Control, feedback handling, external homing and build in scaling functions.

It delivers two functions: read_all and write_all.


The concept of integration in the correspondending task should be as following: 



  ###################      ###########       #########     ############      #####################
  #HARDWARE INPUT   #      # CiA402  #       #Motion #     #  CiA402  #      #  Hardware Output  #
  #  like           #-->>--#read_all #-->>>--#Pids   #-->--#write_all #-->>--#        like       #
  #Ethercat read_all#      #  etc.   #       #       #     #          #      # Ethercat write_all#
  ###################      ###########       #########     ############      #####################




Hal Example:


  #########
  # Setup
  #########
  loadrt [KINS]KINEMATICS
  loadrt [EMCMOT]EMCMOT servo_period_nsec=[EMCMOT]SERVO_PERIOD num_joints=[KINS]JOINTS
  (loadusr -W lcec_conf ethercat-conf.xml)
  loadrt lcec
  loadrt cia402 count=3
  loadrt pid names=x-pid,y-pid,z-pid

  ##########################
  # Functions servo-thread
  ##########################
  addf lcec.read-all servo-thread
  addf cia402.0.read-all servo-thread
  addf cia402.1.read-all servo-thread
  addf cia402.2.read-all servo-thread

  addf motion/ PIDs / PCL / etc .

  addf cia402.0.write-all servo-thread
  addf cia402.1.write-all servo-thread
  addf cia402.2.write-all servo-thread
  addf lcec.write-all servo-thread
  #########################################
  #nets .....


Modes of Operation: 

  By default the component is set to CSP Mode, CSV Mode could be selected with an: 
  
    setp cia402.0.csp-mode 0  in hal.

  Mode changing in runtime is currently not supported, to avoid
  unwanted behaviour.

Homing:

  For using the servo drives internal homing procedure configure your
  joint homing to  Home on Index Pulse only and connect the components
  home input to the motion index-enable Pin:

    HOME_SEARCH_VEL = 0.0
    HOME_LATCH_VEL = 0.2  (Any value but zero, the homing speed is predetermined by the drives configured speed
    HOME_USE_INDEX = TRUE

  If you are using PIDs, don't forget to connect the PIDs index-enable pin.


Flexibility:

  Even though this component exports many pins, you can choose which functions you want to use:

  If you would like to use the CiA State Machine, connect Statusword and Controlword.
  For single use of the scaling function connect only the fb and cmd pins from position or velocity.
  If no Drives homing is needed, let the Pins unconnected.

