# konnect2
A lightweigh emulator of Avaya's connect2 program

Connect2 is a Unix based command line tool used by Avaya Tier 3 engineers and Tier 4 developers. It automatically connects to an Avaya PBX and logs the user in, handling any ASG challenges. It also lets you run OSSI scripts and collect MST traces. It was originally written in the C language to run on the Solaris operating system.

Konnect2 is meant to be a more or less drop in replacement for connect2, but written in the Expect programming language. It runs anywhere expect runs, most notably Linux and on Windows using the Cygwin sublayer. Currently konnect2 provides the following features when connected to an Avaya S8x00 server:

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
