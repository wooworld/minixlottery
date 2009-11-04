Minix Lottery Scheduling
README

Authors:
Mike Phillips  - mjp0672
Gerry Brunelle - gxb7893
Shi Deng - snd8511

Use Notes:

To use our new code, one must simply copy the included files
    /usr/src/kernel/proc.c
    /usr/src/kernel/proc.h
    /usr/src/kernel/system/do_fork.c

into their respective minix directories. Then, proceed to 

   /usr/src/tools

and type the following commands

   su
   make clean
   make hdboot
   reboot

This will compile and install our modified kernel, and then reboot
minix using our new scheduler. Initially, the dynamic lottery 
scheduler will be used. To change the process scheduling algorithm
being used, see "Changing the scheduling type" section of this
readme.

Minix Notes:

The following code changes were made to code files located in release
3.1.2a of the linux source code, available from minix3.org. Our code
is not guaranteed to work with other versions of the source code.


Changing the scheduling type:

Three scheduling algorithms are available in our code. The first being
the standard minix scheduling algorithm, the second being a static
lottery scheduling algorithm, and the third a dynamic lottery scheduling
algorithm. These algorithms are switched between using an integer
variable

                     int style;

located on or around line 99 of /usr/src/kernel/proc.c. The values of
this integer are assigned as follows:

            0 = standard scheduling
            1 = static lottery scheduling
            2 = dynamic lottery scheduling

To change the scheduling algorithm used, one must change this variable,
recompile the code and the kernel, and then install that kernel to 
minix. Upon rebooting, the desired algorithm will be in use.

Source code changes:

Three source code modules (files) have been modified. Changes in these
files are listed here, and can also be found by searching for the code-
change header located above every code change, which is

               /*Lottery Scheduling*/

/usr/src/kernel/system/do_fork.c

This module is used to create a child process from a parent process. In
this class, we assign the child process 5 lottery tickets. The code to do
this is located on or around line 92, where the child process's 
numTickets is set to 5.

/usr/src/kernel/proc.h

This header file defines, among other things, the proc struct used to 
store information about a process. First, a variable for the 
number of lottery tickets a process holds was added to proc struct
on or around line 73

                 int numTickets

From lines 99-104 are various scheduling priority values. These values
govern the queues used by the standard scheduling algorithm. For our
purposes, the number of scheduling queues was increased from 16-20,
using NR_SCHED_QUEUES on line 99.

Since we use the 16th queue for a list of all user processes in our
lottery scheduling algorithm, we have moved the idle process queue
to the 20th queue by changing IDLE_Q to 19 on line 104.

/usr/src/kernel/proc.c

Most of the code changes occur in this file.

An integer to hold the total number of tickets of all queued processes
is located on line 103. This variable is initialized to 0.

               int totalTickets = 0;

Below that, the scheduling algorithm style variable is located. This
is the variable which changes the scheduling algorithm being used. Notes
on this variable can be found above in this document, and also in the
code itself. This variable is on or around line 109.

To change the number of tickets that a process currently holds, the
setpriority(ntickets,rp) method was added. This method lives in lines
519-562. rp is the current process being changed, and ntickets is the
number of tickets to add or subtract from the process (positive
ntickets is number of tickets to add, negative number of tickets is
tickets to subtract). The functionality of this method is explained
within, and its purpose and design is stated in the design document.

The enqueue method adds a process to the queue of processes.when a
process is added to the queue, its tickets must be added to the
total lottery ticket pool. This is done in enqueue(), on line 605.

             totalTickets = totalTickets + rp->numTickets;

The reverse of this is dequeueing. Whatever the reason, a process
is eventually dequeued. When this happens, the process's tickets
should no longer be considered in the total pool of lottery tickets.
This is done in the dequeue() method, on line 661.

             totalTickets = totalTickets - rp->numTickets;

One of the bigger differences between minix's standard scheduling
algorithm and lottery scheduling is that in lottery scheduling,
all processes are put at the back of the 16th queue. In sched(),
processes are evaluated and the queue they belong in is decided.
On and after line 704 is the lottery scheduling code, which
simply sets the queue variable to 15 (for the 16th queue) and the
front variable to 0, to indicate that it is false.

The last code changes are in the pick_proc method. This method is
where the enxt process to run is chosen. The lottery scheduling
algorithm for choosing a process is detailed in the design
document. The code for this decision is located on lines 758-778.

