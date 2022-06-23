# Linux-Resource-Monitor
CSCB09 ASSIGNMENT 1: C program to monitor different resources

--------------------------------------------------------------------
--------------------------------------------------------------------
--------------------------------------------------------------------


DESCRIPTION:
HI! 
assgn1.c is a program written completely in C that allows you to view information about your machine/computer! 

The information that you can view are:
-Physical/Total Memory Usage
-Virutal/Total Memory Usage
-Users connected
-Sessions each user is connected to
-Number of cores in your machine
-Total CPU Usage
-System information:    System Name 
  			 Machine 
   			 Version 
			 Release 
			 Architecture 
	

--------------------------------------------------------------------
--------------------------------------------------------------------
--------------------------------------------------------------------


HOW TO USE:
-Open your Linux terminal and move to the directory in which you have saved your assgn1.c file

-The most basic way to run your program is to compile it with: "gcc a1.c" and then to execute it with "./a.out"



Now comes the beauty of the program
You can also enter multiple command line arguments. There are:


	--graphics
		To include graphical output in the cases where a graphical outcome is possible
		These are only for information about the memory and cpu
		
	--system
		Reports how much utilization of the CPU is being done
		Reports how much utilization of memory is being done (report used and free memory)

	--user
		Reports how many users are connected in a given time
		Reports how many sessions each user is connected to


	--samples=N
		If used the value N will indicate how many times the statistics are going to be collected and results will be average and reported based on the N number of repetitions.
		If not value is indicated the default value will be 10.
		
	--tdelay=T
		To indicate how frequently to sample in seconds.
		If not value is indicated the default value will be 1 sec.
	


In addition you can add flags. These are:

	-g
		Works the exact same way as --graphics

	2 integers with whitespace in between them at the END of the command. The first integer will be counted as the sample N and the second integer will be counted as tdelay N

-When an incorrect command is entered, the program will not execute as desired and instead you will be shown the message "Enter correct commands"

-To close your program while it is running press, CTRL+C or CMD+C


--------------------------------------------------------------------
--------------------------------------------------------------------
--------------------------------------------------------------------
HOW TO NOT USE:

-DUPLICATE COMMAND LINE ARGUMENTS ARE NOT ALLOWED
	For example:   "./a.out --user --user"
			"./a.out --system --user --system"
			"./a.out --tdelay= 5 --tdelay=4"
			"./a.out -g --graphics"
			These are not accepted by the program and will not cause execution.Instead you will be asked to enter the correct command.
			
-Positional arguments for samples and tdelay must occur at the end and BOTH must be present
Valid commands are:    ./a.out 13 2
		       ./a.out --user 15 2
		       ./a.out --user -g --system 7 2


Invalid commands are:  ./a.out --user 3
			./a.out 13
			./a.out 13 3 --user
			./a.out 13 -g 3
			
			
--------------------------------------------------------------------
--------------------------------------------------------------------
--------------------------------------------------------------------
LIBRARIES INCLUDED:


#include <stdio.h> - for standard input/output. e.g printf() function

#include <sys/sysinfo.h> - for using the sysinfo struct and collecting information about memory

#include <stdlib.h> - for using the atoi() function that converts a string to an integer

#include <sys/utsname.h> - for using the utsname struct to collect information about the system

#include <unistd.h> - for using the sleep() function to add delay between our samples

#include <utmp.h> - for using the utmp struct to collect information about users

#include <string.h> - for using string operations such as strcmp() and strncmp()

#include <stdbool.h> - to be able to use and create boolean variables and functions

#include <ctype.h> - to use the isdigit() function to be able to check if every character in a string is a number 

--------------------------------------------------------------------
--------------------------------------------------------------------
--------------------------------------------------------------------
FUNCTIONS IN PROGRAM (BACKEND):





--------------------------------------------------------------------



>>>>>>  void printMemoryInfo(int sample, int delay, bool g)
			
-This function takes an integer value of sample which is the sample size

-This function takes an integer value of delay which the time between each sample data is collected and shown

-This function takes a boolean value of g which is true if the user wants output in graphical format and false otherwise

-When called the function prints out Physical Used Memory/ Total Memory along with Virtual Used Memory/Total Virtual Memory in each line.

-Each line corresponds to a single sample and the time difference it takes between displaying the different samples is the delay

CALCULATIONS USED:
-Physical Memory Usage = totalram-freeram
-Total Memory Usage = totalram
-Virtual Memory Usage = totalram+totalswap-freeswap-freeram
-Total Memory Usage = totalram + totalswap

-They are all displayed in GIGABYTES

-If the graphics flags are being used then a graphical representation of the relative change between the 2 samples will be shown beside each line
	| this bar will be present beside every line and it signals that a graphical represenation is being used
	: this is used for each 0.01 increase in Physical Memory Used
	@ this is used to end the graphical representation of an increase in memory usage
	# this is used for each 0.01 decrease in Physical Memory Used
	* this is used to end the graphical representation of an decrease in memory usage
	0 is used to indicate no change in physical memory usage
	
AN EXAMPLE:
	Number of samples: 10 -- every 1 seconds
	-----------------------------------------
	### Memory ### (Phys.Used/Tot -- Virtual Used/Tot)
	7.23 GB / 7.50 GB -- 7.56 GB / 18.40 GB
	-----------------------------------------




		
--------------------------------------------------------------------





>>>>>>  void printSessionUsers(int sample, int time)

-This function takes an integer value of sample which is the sample size

-This function takes an integer value of time which the time between each sample data is collected and shown

-We create a utmp struct pointer that will allow us to access information about the users and the sessions. We use setutent() and getutent() and check which users are in normal process with USER_PROCESS. Then we print out the ut_user, ut_line and ut_host which are the username, device name of tty-"/dev/" and kernel version for run-level messages respectively.

- https://man7.org/linux/man-pages/man5/utmp.5.html may help to understand better 

- we use escape codes for easy and user-friendly visualization (https://gist.github.com/fnky/458719343aabd01cfb17a3a4f7296797)

-Session users are printed for each sample and then erased and then the new users for each sample is printed again. Under all the names of the users there will be a line saying "Sample: n" where n is the value of the sample number. If 10 samples are to be printed, that PARTICULAR line under all the users will show from Sample:1 - Sample:9 and then for the final sample it will disappear and the users for the final sample will be only shown

-After all iterations are complete, the final users that will be displayed are users from the FINAL sample data collected




--------------------------------------------------------------------





>>>>>>  void printCpuInfo(int sample, int delay, bool g)

-This function takes an integer value of sample which is the sample size

-This function takes an integer value of delay which the time between each sample data is collected and shown

-This function takes a boolean value of g which is true if the user wants output in graphical format and false otherwise

-In the first part of the function,s we open the file /proc/cpuinfo as it contains information about the number of processors in our machine. We open the file and iterate through every word and check how many times the word "processor" occurs in the file. That is the number of cores in our machine and we print that out to the user.

-In the second part of the function, we open the file /proc/stat twice for each sample. We open the file and only read the first line of the file and collect information about the user, nice, system, idle, iowait, irq, softirq and steal. We then create 2 variables and store the values accordingly: [long prevnonidle = user+nice+system+irq+softirq+steal],[long previdle = idle+iowait]. We then close the file and sleep for the required amount of seconds and open the file again and store the information again in the same variables as we did the last time. We then create two variables again and store the information again accordingly: [long nonidle = user+nice+system+irq+softirq+steal],[long iidle = idle+iowait].
Then we perform the following calculations: 

long prevtotal = previdle+prevnonidle;
long total = iidle+nonidle;
long totald = total - prevtotal;
long idled = iidle - previdle;
double val1 = totald-idled;
double val2 = totald;
float percent = (val1/val2)*100;

Here percent is the cpu usage. 


-The formula used has been collected from: (https://stackoverflow.com/questions/23367857/accurate-calculation-of-cpu-usage-given-in-percentage-in-linux)
	Here it says, 
		PrevIdle = previdle + previowait
		Idle = idle + iowait

		PrevNonIdle = prevuser + prevnice + prevsystem + previrq + prevsoftirq + prevsteal
		NonIdle = user + nice + system + irq + softirq + steal

		PrevTotal = PrevIdle + PrevNonIdle
		Total = Idle + NonIdle

		# differentiate: actual value minus the previous one
		totald = Total - PrevTotal
		idled = Idle - PrevIdle

		CPU_Percentage = (totald - idled)/totald

-Escape codes are also implemented for user-friendly visualization of the text (https://gist.github.com/fnky/458719343aabd01cfb17a3a4f7296797)

AN EXAMPLE: 
	------------------------------------------
	### CPU Information and Statistics ###
	Number of cores: 8
	Total cpu use: 12.58 % 
	------------------------------------------
	
	Here the output is without graphical representation
	
	------------------------------------------
	### CPU Information and Statistics ###
	Number of cores: 8
		   ||| 2.77 
		   ||| 2.15  
		   ||||| 4.28 
		   ||| 2.52  
		   ||| 2.04  
		   |||| 3.14 
		   ||| 2.88  
		   ||| 2.64  
		   ||| 2.15  
		   ||| 2.41  
	Total cpu use: 2.41 %  
	------------------------------------------
	This is with graphical representation. As seend, there are 8 processors. Moreoever, 10 samples were collected and the cpu usage for each sample is shown. 2.77 is rounded up to 3 so it has 3 bars beside it and similary, 3.14 is rounded up to 4 so it has 4 bars beside it.
	The FINAL value where it says Total cpu use: 2.41 % is the value of the final sample collected



--------------------------------------------------------------------





>>>>>>  void printSystemInfo()

-This function is responsible for printing out informaton about the system of the computer.

-When called, it prints out the system name, machine name, release, version and architecture of the computer in separate lines

-We create a utsname struct in our function and use it to access the sysname, nodename, release, version and machine fields and print them out to the user

AN EXAMPLE: 
	------------------------------------------
	###System Information###
	System Name      = Linux
	Machine name     = zeyrox
	Release          = 5.13.0-28-generic
	Version          = #31~20.04.1-Ubuntu SMP Wed Jan 19 14:08:10 UTC 2022
	Architecture     = x86_64
	------------------------------------------



--------------------------------------------------------------------





>>>>>>  bool isNumber(char *string)

-This function is not directly responsible for handling anything in the program but it is important nevertheless. 

-It takes a pointer to a string as an argument and returns true if the string is a positive integer and false if it is not an integer




--------------------------------------------------------------------





>>>>>> main(int argc, char **argv)

-this function is responsible for handing different possibilites of user inputs and printing the desired information.

-every case is considered and properly commented out in the assgn1.c file


--------------------------------------------------------------------
--------------------------------------------------------------------
--------------------------------------------------------------------

Invalid User Inputs Examples:

./a.out 3 		           (Both positional arguments for tdelay and samples must be present)

./a.out --user --user              (duplicates are not allowed)

./a.out asvs                       (Invalid command line argument)

./a.out --tdelay=5 --tdelay=4      (duplicates are not allowed)

./a.out 13 2 --user                (positional arguments must only occur at the end)






 












