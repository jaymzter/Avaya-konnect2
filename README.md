# konnect2
A lightweigh emulator of Avaya's connect2 program

Connect2 is a Unix based command line tool used by Avaya Tier 3 engineers and Tier 4 developers. It automatically connects to an Avaya PBX and logs the user in, handling any ASG challenges. It also lets you run OSSI scripts and collect MST traces. It was originally written in the C language to run on the Solaris operating system.

Konnect2 is meant to be a more or less drop in replacement for connect2, but written in the Expect programming language using a "clean room" or "Chinese wall" approach (i.e. not based on or referencing any actual connect2 code). It runs anywhere expect runs, most notably Linux and on Windows using the Cygwin sublayer. Currently konnect2 provides the following features when connected to an Avaya S8x00 server:

1. Automatically connect and log in to the PBX using SSH
2. Once logged in you can run OSSI scripts straight from the SAT terminal. Konnect2 will activate the Avaya OSSI interface, run your script and write the results to a local file for analysis later
3. You can also automagically run a single ossi script without having to manually log into the PBX
4. Session logging is done by capturing all screen output
5. There is a primitive formatting facility, but because the OSSI doesn't have a sset output format, your command results should best be massaged by an external program.

Once connected, the following internal commands are available:
```
~~ to send a ~

~h internal help message

~o run an OSSI script (ONLY for use in Avaya SAT)

~r enable writing logfile on localhost

~s disable logfile

~q to quit

~x to run a local shell
```

## Files in this repository
* konnect2 - the main program
* ossi.sc  - a folder containing sample ossi scripts
* ossifmt  - a pretty formatter for the files konnect2 creates when running ossi scripts

# How to make OSSI scripts
The OSS Interface (OSSI) provides form independent access to the system management, maintenance, and traffic data that is normally available through the System Access Terminal (SAT). All commands available via the SAT interface can be run via OSSI. The default format of the commands will match the format of the commands entered on the SAT interface so that existing command interpreter functions can be used. This means that a command will consist of three parts: action, object, and qualifier. For example, `add station 2000` would be a valid command format for the OSS Interface if it is valid for the SAT interface.

## Initial Option Selection
After a successful login, the user is prompted for a Terminal Type, since OSSI is an internal interface, it is not included in the list of terminals displayed to the user. The OSSI is selected by typing `ossi` followed by a carriage return. If any OSSI options are desired, the options are selected by appending the appropriate characters to ossi. For example, if the terminal is specified to be `ossinw`, the **n** and **w** options will be activated.

Some important OSSI options and associated selection characters are: 
* **n** - Suppress Field IDs (FIDs, see below). To minimize the volume of data transmitted, this option sends the requested data without the accompanying FIDs.
* **s** - Silent option. Normally OSSI returns the command that was input to begin the command response. This option suppresses the return of the input command.
* **m** - Documentation mode. This option places OSSI in documentation mode. Administration and retrieval of data is not supported in this mode. For a given command, a list of the FIDs for the associated object are returned along with the titles and help associated with each field. In addition, the fields that become active or change as a function of the data are identified.
* **t** - Terminal interface. This option is provided for testing the OSSI from a terminal and for viewing the documentation output. Input is echoed and backspace is supported to make it easier to enter commands in OSSI format. Long output is displayed in blocks and an option is provided to continue or suppress output after each block.

The terminal type and OSSI options can be changed between commands by sending the "newterm" command. This causes the terminal selection sequence to be initiated. The Terminal Type prompt is sent to the OSS and the same selection criteria described above is valid. This command would be useful to an OSS that requires administration via both SAT cut-through and OSSI, such as what konnect2 does.

## Command Format
An OSSI command consists of a set of records. Each record consists of a flag in the first position and a carriage return as the terminator. Any record that does not begin with a recognizable flag is discarded by the switch. This includes session control commands such as "logoff" and "newterm".

The OSSI command record flags and their definitions are:
* **c** - Command record. This record must include valid action, object, and qualifier(s). The valid commands and format are identical to the commands that are entered on the command line of the SAT interface
* **f** - Field identifier record. These records consist of a list of FIDs separated by tab characters. For report commands, the output can be expanded to include all fields from the object by requesting fields "all". These records are optional for both view and update commands. Any number of FID records may be sent with a command. If FID records are omitted, view commands send all fields that would appear.
* **d** - Field data record. These records consist of a list of field data values separated by tab characters. The data values are associated in sequence with the FIDs that have been sent. Any number of field data records may be sent with a command. Data records may be interspersed with FID records as long as the correct sequence is maintained. If FID records are not sent, update commands assume the data is sent in default field order.
* **t** - Command terminator. Normally command data is buffered by the OSSI until this record is received. The command is then processed and the appropriate output is returned. If command validation has been requested, this record is interpreted as a request to commit the data and terminate the command on the SAT for the command.

## Field Identifier Format
FIDs are unique for a given object and are based on internal field handling by the System Management software. A FID consists of eight hexadecimal characters.

## Usage and Examples for OSSI scripts
A sample ossi file (dispsoft.sc):
```
$ cat ossi.sc/dispsoft.sc
cdisp soft
f0fa2ff00       0010ff00
devents a
t
cdisp soft
f0fa2ff00       0010ff00
dclear  a
t
```
Log into a switch and use ossi in documentation mode to test. This will show the FIDs in a read only manner. 
```
System: Extra Large       Software Version: R015x.00.0.819.0

Terminal Type (513, 715, 4410, 4425, VT220, NTT, W2KTT, SUNT): [513] ossimt
t
cdisp ev
t
cdisp ev
                EVENT REPORT
                The following options control which events will be displayed.
                EVENT CATEGORY
 000bff00 Category:     10
        all     contact-cl      data-error      denial  meetme  vector
                REPORT PERIOD
 0010ff00 Interval:     1
        h(our)  d(ay)   w(eek)  m(onth) a(ll)
*00071d00       2
*00071e00       2
*00071f00       2
*00072000       2
*00081d00       2
*00081e00       2
*00081f00       2
*00082000       2
                SEARCH OPTIONS
 0013ff00 Vector Number:        5
        Enter vector number between 1-2000, or blank
 0012ff00 Event Type:   5
        Enter event type between 0-9999, or blank
 000cff00 Extension:    13
        Enter assigned extension, or blank
n
                EVENTS REPORT
                Event Event                      Event     Event      First       Last     Evnt
                Type  Description                Data 1    Data 2     Occur       Occur     Cnt
 0004ff00       5
 0012ff00       25
 0013ff00       9
 0006ff00       8
 0010ff00       2
 0007ff00 /     2
 0008ff00 /     2
 0009ff00 :     2
 0011ff00       2
 000aff00 /     2
 000bff00 /     2
 000cff00 :     2
 000dff00       3
t
...
```
The same form as seen from the SAT
```
display events                                                  Page   1 of   1 
                                  EVENT REPORT

      The following options control which events will be displayed.

         EVENT CATEGORY

            Category: vector    

         REPORT PERIOD

            Interval: a      From:   /  /  :   To:   /  /  :  

         SEARCH OPTIONS

                           Vector Number:      
                              Event Type:      
                               Extension:             
```
Knowing how the form looks in the SAT, you can tie the FIDs to the correct fields. With th information above it is trivial to create an ossi script to retrieve Denial Events. 
```
cdisp eve
000bff00        0010ff00
ddenial a
t
```
Where: 
* **c** - Command record. This record must include valid action, object, and qualifier(s).
* **d** - Field data record. These records consist of a list of field data values separated by tab characters. The data values are associated in sequence with the FIDs that have been sent. Any number of field data records may be sent with a command. Data records may be interspersed with FID records as long as the correct sequence is maintained. If FID records are not sent, update commands assume the data is sent in default field order.
* **f** - Field identifier record. These records consist of a list of FIDs separated by tab characters.
* **t** - Command terminator.
