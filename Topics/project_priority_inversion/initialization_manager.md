# Initialization Manager

Initializes BSP, RTEMS device drivers, filesystem, and application.

## Initialization Tasks
Initialization task(s) are the mechanism by which RTEMS transfers initial control to the user’s application.
Initialization tasks differ from other application tasks in that they are defined in the User Initialization Tasks Table and automatically created and started by RTEMS as part of its initialization sequence.

## Idle Tasks
The Idle Task is the lowest priority task in a system and executes only when no other task is ready to execute.
The default implementation of this task consists of an infinite loop.
The Idle Task is preemptible and WILL be preempted when any other task is made ready to execute.
This characteristic is critical to the overall behavior of any application.

## Initializing RTEMS
The Initialization Manager `rtems_initialize_executive()` directive is called by the `boot_card()` routine which is invoked by the BSP once a basic C run-time environment is set up.
This consists of
  - a valid and accessible text section, read-only data, read-write data and zero-initialized data,
  - an initialization stack large enough to initialize the rest of the Board Support Package, RTEMS and the device drivers,
  - all registers and components mandated by Application Binary Interface, and
  - disabled interrupts.

Each RTEMS feature used the application may optionally register an initialization handler.
The system initialization API is available via `#included <rtems/sysinit.h>`.

## Directives
### INITIALIZE_EXECUTIVE - Initialize RTEMS
Iterates through the system initialization linker set and invokes the registered handlers.
The final step is to start multitasking.

This directive should be called by boot_card() only.
```
void rtems_initialize_executive(void);
```
