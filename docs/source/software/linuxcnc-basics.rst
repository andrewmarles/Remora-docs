LinuxCNC basics
=================

LinuxCNC is well documented, but there are some key concepts of LinuxCNC that are important to know so that we can understand why the Remora component was developed and how it off loads tasks that a traditional parallel port stepper based LinuxCNC setup would do.


LinuxCNC task threads
----------------------

In LinuxCNC, a thread is a list of functions that runs at specific intervals as part of a realtime task. When a thread is first created, it has a specific time interval (period), but no functions. Functions are added to the thread, and these will be executed in order every time the thread runs. In a traditional stepper configuration, two threads a created:

* Base Thread - a short period (high frequency) thread that is used for software step generation.
* Servo Thread - a longer period (lower frequency) thread that is used for calculating motor positions, checking IO and logic functions

The Remora component allows the Base Thread tasks to be off loaded to the controller board


Hardware Abstraction Layer (HAL)
---------------------------------

| This part of the config can be a bit confusing as this is a very powerful and versatile part of Linux CNC
| One dose not need to know every part or function to get up and running but it is a good idea to have a basic knowledge.
| The link below will take you to the documentation for the HAL, see the section named "HAL (Hardware Abstraction Layer)"
| https://linuxcnc.org/docs/html/ 
| Also when looking up functions this can be useful
| https://linuxcnc.org/docs/html/man/man9/?