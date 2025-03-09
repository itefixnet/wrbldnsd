**Wrbldnsd** is a Windows anti-spam solution by serving [DNSBL](https://en.wikipedia.org/wiki/DNSBL) zones locally. It is a packaging of Rbldnsd, Rsync and Cygwin. You can use Wrbldnsd to locally mirror DNSBL zone files, thus eliminating latency problems in larger environments and decreasing time used for email filtering significantly.

[Rbldnsd](https://rbldnsd.io "A small and fast DNS daemon especially made to serve DNSBL zones") is a small and fast DNS daemon which is especially engineered for serving DNSBL zones. It can serve both IP-based and name-based blocklists. All zones are kept in memory for faster performance with a decent memory footprint.  _[Rsync](http://rsync.samba.org/)_  uses the [Rsync algorithm](http://rsync.samba.org/tech_report/) which provides a very fast method for bringing remote files into sync. It does this by sending just the differences in the files across the link, without requiring that both sets of files are present at one of the ends of the link beforehand. [Cygwin](http://www.cygwin.com/) is a Linux-like environment for Windows. It consists of a DLL, which emulates substantial Linux API functionality, and a collection of tools.

### Installation

Wrbldnsd comes as a ZIP file containing an [NSIS](https://nsis.sourceforge.io/Main_Page) installer. Simply unzip your downloaded copy and run the installer :

> 1.  Accept License agreement.
> 2.  Specify an installation location.
> 3.  Installation starts. By clicking 'Details' button, you can get more detailed information about installation.

### Usage

The installer will set up the service **Rbldnsd** starting the script **/rbldnsd.sh**, allowing you for customization according to your needs

```
#!/bin/bash

# Script to start rbldnsd with desired parameters, check man page for other options
# NB! not all of them are applicable as this is a Windows environment

# address\[/port\] - bind to (listen on) this address (required)
export BINDINFO\=127.0.0.1/53

# each zone specified using \`name:type:file,file...'
export ZONES\=localzone:ip4set:local.zone

# working directory
export WORKDIR\=/work

#Start rbldnsd
/bin/rbldnsd -n -f -b $BINDINFO -w $WORKDIR $ZONES
```

 You need to:

*   update the ip address and port to serve the requests (**BINDINFO**). The default value is non-functional (connections at 127.0.0.1 on port 53)
*   copy dataset files to be used to the _**/work**_ folder and configure **ZONES** parameter. 

There are many other options that can be configured via the script file. Check the man page for options.

You can now start the service and monitor the activity via the log file at _**/var/log**_ folder.

The service is set up in automatic mode and run by the LOCAL SERVICE Account (limited access).

Man page for rbldnsd is available [here](/content/rbldnsd-man-page "Rbldnsd man page"). More documentation can also be found [here](https://rbldnsd.io/documentation/ "Rbldnsd documentation").

Provided by [itefix.net](https://itefix.net)
