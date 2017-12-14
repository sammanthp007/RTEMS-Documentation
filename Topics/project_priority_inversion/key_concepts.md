This documents contains a summary of what I have learned about rtems from the C-User guide's chapter 2: Key Concepts

# Objects
RTEMS provides directives which can be used to dynamically create, delete, and manipulate a set of predefined object types. These types include tasks, message queues, semaphores, memory regions, memory partitions, timers, ports, and rate monotonic periods.

1. Object Name:
  - data_type `rtems_name` is used to store object names
  - The `rtems_build_name` routine is provided to build an object name from four ASCII characters.
  e.g
  ```
    rtems_name my_name;
    my_name = rtems_build_name( 'L', 'I', 'T', 'E' );
    ```
  - `rtems_object_get_name` can be used to obtain the name of any RTEMS object using just its ID. This routine attempts to convert the name into a printable string.
  ```
  #include <rtems.h>
#include <rtems/bspIo.h> void print_name(rtems_id id) {
char buffer[10]; /* name assumed to be 10 characters or less */ char *result;
result = rtems_object_get_name( id, sizeof(buffer), buffer );
printk( "ID=0x%08x name=%s\n", id, ((result) ? result : "no name") );
}
```

2. Object ID:
  - An object ID is a unique unsigned integer value which uniquely identifies an object instance. Object IDs are passed as arguments to many directives in RTEMS and RTEMS translates the ID to an internal object pointer.
  - The thirty-two bit format for an object ID is composed of four parts: API, object class, node, and index.
  - The sixteen bit format for an object ID is composed of three parts: API, object class, and index.
  - The data type `rtems_id` is used to store object IDs.
  - The object ID is required as input to all directives involving objects, except those which create an object or obtain the ID of an object.

# Communication and Synchronization
A real-time executive should provide an application with the following capabilities:
  - Data transfer between cooperating tasks
  - Data transfer between tasks and ISRs
  - Synchronization of cooperating tasks
  - Synchronization of tasks and ISRs

The following managers were specifically designed for communication and synchronization:
  -  Semaphore
  -  Message Queue 
  -  Event
  -  Signal

The semaphore manager supports mutual exclusion involving the synchronization
of access to one or more shared user resources. Binary semaphores may utilize
the optional priority inheritance algorithm to avoid the problem of priority
inversion. The message manager sup- ports both communication and
synchronization, while the event manager primarily provides a high performance
synchronization mechanism. The signal manager supports only asynchronous
communication and is typically used for exception handling.
