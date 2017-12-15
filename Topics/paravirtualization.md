# Paravirtualization

#### Paravirtualization:
It is a variation of virtualization where the guest operating system is modified to work in cooperation with the VMM to optimize performance. 

Advantage over normal virtualization:
- This capacity minimizes overhead and optimizes system performance by supporting the use of virtual machines that would be underutilized in conventional or full virtualization.
- Paravirtualization eliminates the need for the virtual machine to trap privileged instructions. Trapping, a means of handling unexpected or unallowable conditions, can be time-consuming and can adversely impact performance in systems that employ full virtualization.


Disadvantage:
- The main limitation of paravirtualization is the fact that the guest OS must be tailored specifically to run on top of the virtual machine monitor.

Which is why we will need to update the rtems bsp for each architecture, since rtems will need to comply with all architecture specifically.

# STOPPED WORKING ON IT
