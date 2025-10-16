# 1. Project Description and Scope

**Background / Context**

Uvic Rocketry currently has no way of acquiring visual flight data. Visual footage of systems like recovery activation, parachute deployment etc would be extremely useful in providing insight for future designs. The scope of the Camera system is to design a suite of cameras to monitor numerous locations of the rocket. These locations will consist of:

1. Outside the rocket pointing down towards the motor

2. Inside the fuselage looking towards the horizon

3. ~~Outside the rocket looking up towards the nosecone~~

4. Inside the recovery bay looking up

Note: Mounting will not be part of this system’s scope.

**Design Specifications**

The camera system design must address the following elements:

* **Camera selection**: Must record at suitable fps and resolution for footage to be useful  
* **Footage storage**: The system will be responsible for storing its own footage  
* **Integration**: The system must integrate with the lower avionics bay team, power supply team, and flight computer team. Requirements for this system must be clearly communicated with these teams1.  
* **Wiring Harness(s):** A wiring harness must be designed that suits the needs of your system  
* **Cost & Mass:** Provide a cost and mass estimate for your selected design  
* **Optional:** An STM32 based software driver for your selected camera system

**Evaluation Criteria**

Concepts should be evaluated on

* **Camera performance** (fps, resolution)  
* **Total Storage Space**  
* **Size and Mass Efficiency**  
* **Ease of Integration**

# 2. Requirements

**[[UVR-COMP-LC26] Anduril-3 Master Requirements List](https://docs.google.com/spreadsheets/d/1ZD85g12owsyN6Y8WK_DpIhRM8TG9Gyk9y_tQ9WLFcQ0/edit?gid=0#gid=0)**

## **Constraints**

**Video**

* Resolution: 1920×1080 at 30fps minimum (60fps preferred, higher quality if possible)  
  * Bitrate/quality/FPS must be sufficient for post-flight analysis and use in promotional material  
* Units should operate independently without needing to be remotely signaled to start recording before launch  
  * Need to be able to potentially record for entire launch window

**Optics**

* FOV and lens size should suit the location and function of each camera \- this could mean deciding on one set as a compromise, or selecting different options for different locations  
* Cameras inside the rocket must be able to focus on the components they are monitoring  
  * Autofocus likely needed to capture parachute deployment and other moving parts  
* Cameras outside the rocket must be able to focus at infinity and provide as wide of an FOV as possible while minimizing visual obstructions

**Data**

* Data loss must be avoided or minimized as a priority, especially under failure conditions  
  * System should be specifically designed to prevent corruption during power loss events  
  * Storage format should make partial data recovery from incomplete recordings as easy as possible  
* Full flight should be captured (with buffer before launch) and must not be overwritten after landing

**Other**

* 4 initial camera positions, **possibly more contingent on mass \+ cost \+ space**  
  * From Analysis:  
    * *Initial (all attached to lower AV bay)*  
      5. *Outside of rocket looking at fins (to watch for fin flutter)*  
      6. *Outside of rocket looking at nosecone (to watch recovery deployment)*   
      7. *Inside of rocket looking at horizon (for coolness and outreach)*  
      8. *Inside of rocket looking at recovery deployment*  
    * *Extra if we have space/time/money*  
      5. *Fully internal in lower av bay*   
      6. *One in the nose cone looking into recovery*  
      7. *One looking into nose cone*  
  * Cameras must be modular/adaptable to allow for different placements/purposes without complete redesign  
* Power must be maintained through deployment of parachutes (not possible if located away from power supply)  
  * Initial 4 camera locations may permit power through modular power supply while others would require separate batteries. (remembering that any power source in the rocket is batteries of some kind)   
* Must fit within available space  
  * Must minimize internal volume   
  * Must minimize external form factor  
* Must fit within mass budget  
* Must minimize cost while maintaining quality/reliability  
* Must maintain performance while potentially operating in high-temperature environment without the possibility of dissipating heat (lack of air circulation inside rocket)  
  * Maintain operations in environments up to 40°C

## **Design Considerations**

**Camera Operation** 

Each camera records independently with onboard processing, storage, and launch detection. Without external communication, cameras must either have power and record from setup on launchpad, or start recording on launch via dedicated sensor or communication with flight computer.

**Controller** 

Controller needs to handle video data throughput from camera sensor to SD card \- at 1080p60fps this could be 10-20+ MB/s depending on bitrate. Power consumption is a concern, especially if units are battery-powered \- might need low-power operation during pad wait time, possibly sleep modes with accelerometer or flight controller wake-up. Thermal management will be tricky since cameras will be enclosed without airflow, may need to consider options for heat dissipation or select lower-power options. Controller must be able to handle clean shutdowns/write completions even during sudden power loss or high vibration.

**Power** 

Dedicated (separate) battery power for any cameras that get separated from central power supply during flight (e.g. in nosecone). Central power for cameras where possible, which saves mass by avoiding unnecessary batteries. Since the initial camera positions are in the LAB, power by modular supply is likely possible. Small backup batteries may be a good idea as any power supply failures are more likely to happen during flight (not while waiting on pad), and therefore only a small amount of additional charge would be needed to complete the footage.

**Storage** 

Large (128GB?) SD cards would allow simple continuous recording without complicated file management (e.g. circular buffer). Large margin over 6-hour requirements. Sequential writing minimizes the chance of corruption during vibration (may not actually be a concern). Trading storage costs for reliability, but storage is so cheap that it’s likely not worth it to overcomplicate this and risk corruption or overwrites. Depending on file quality/size, it may be possible to go with a smaller card after determining storage used per unit time.

**Design** 

Decide on a standardized set of core electronics (controller, accelerometer, storage, power management) but possibly have position-specific optics (FOV, lens size, focus). COTS camera module with integrated lens for basic positions. Specialized sensors & optics possible for more complex views (fisheye? low-light? IR?). Cameras inside the rocket will likely need illumination via LEDs. Can design the core control module while camera placements are still being finalized, after which optics can be refined. 