# scripts


Following instructions are based on https://github.com/sinanpetrustoma/autostopping

    Autonomous Databases
    OAC native

Step 1: Install and configure OCI CLI on a Compute Intance (Linux VM with one CPU would be ok) in your tenancy. https://docs.cloud.oracle.com/iaas/Content/API/SDKDocs/cliinstall.htm It takes about 10min

This version uses Instance Principals for Authenticaion: create a Dynamic Group and add a rule for the OCID of the compute instance where the scripts will run create a policy in the root compartment as follows: Allow dynamic-group <group_name> to use all-resources in tenancy

An alternative is to restrict the policiy to a concrete compartment B. Asuming B is as subcompartment of A: Allow dynamic-group <group_name> to use all-resources in compartment A:B

Step 2: Save the delivered shell scripts on the VM, e.g. in /home/opc/ocicli/scipts/ Create a sub-directory named log: i.e. /home/opc/ocicli/scripts/log/

Step 3: Change the OAC_LISTFILE/ADB_LISTFILE path in the .sh scripts accordingly. See Step 7. e.g.: OAC_LISTFILE="/home/opc/ocicli/scripts/oaclistfile.txt"

Step 4: Make the shell scripts executable chmod +x /home/opc/ocicli/scripts/*.sh

Step 5: As we are using Instance Principals for Authentication, there is no config file contaning the Tenancy ID: Change the value of TENANCY_OCID to your tenancy ID and REGION in all scripts

Step 6: Create an automation using crontab. In the following example shutting down the resources every day at 6p.m., starting first ADB at 7 a.m and OAC at 7.15 a.m. and keeping the log files for 7 days. 

0 18 * * * /home/opc/ocicli/scripts/adb_startstop.sh stop > /home/opc/ocicli/scripts/log/adb_startstop '+\%Y\%m\%d_\%H\%M\%S'.log 

0 18 * * * /home/opc/ocicli/scripts/oac_startstop.sh stop > /home/opc/ocicli/scripts/log/oac_startstop '+\%Y\%m\%d_\%H\%M\%S'.log 

0 7 * * * /home/opc/ocicli/scripts/adb_startstop.sh start > /home/opc/ocicli/scripts/log/adb_startstop '+\%Y\%m\%d_\%H\%M\%S'.log 

15 7 * * * /home/opc/ocicli/scripts/oac_startstop.sh start > /home/opc/ocicli/scripts/log/oac_startstop '+\%Y\%m\%d_\%H\%M\%S'.log 

0 3 * * * find /home/opc/ocicli/scripts/log/ -type f -mtime +7 -name '*.log' -execdir rm -- '{}' ;

make sure your environment is sourced in cron by add following line to crontab (change the path to your .profile file): SHELL=/bin/bash BASH_ENV=/home/opc/.bash_profile

IMPORTANT: the scripts do not start the resources in the next morning: Option 1: every one in the team starts only the resources needed manually. Option 2: duplicate the scripts and replace "stop" by "start" and create crontab roles to run them
