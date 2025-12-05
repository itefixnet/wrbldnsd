# wrbldnsd - Rbldnsd for Windows

wrbldnsd is a packaging of Rbldnsd for Windows systems. It provides a Windows anti-spam solution by serving DNSBL (DNS-based Blackhole List) zones locally. It is a self-contained package including Rbldnsd, Rsync, and Cygwin runtime, offering a ready-to-use DNSBL server for Windows.

## Features

- **Complete Rbldnsd implementation**: Full-featured DNS daemon engineered for serving DNSBL zones on Windows
- **Fast DNS serving**: Small and fast DNS daemon with all zones kept in memory for optimal performance
- **IP and name-based blocklists**: Serves both IP-based and name-based blocklists
- **Local DNSBL mirroring**: Eliminate latency problems by serving DNSBL zones locally
- **Efficient email filtering**: Significantly decrease time used for email filtering in larger environments
- **Rsync integration**: Fast synchronization of remote DNSBL zone files using the Rsync algorithm
- **Windows service**: Runs as a Windows service with automatic startup
- **Limited access mode**: Service runs under LOCAL SERVICE Account for enhanced security
- **Cygwin integration**: Includes Cygwin runtime for Linux-like environment on Windows
- **Easy configuration**: Simple script-based configuration for zones and binding

## Requirements

- Vista or later
- No external dependencies required — all components included in the package

## Download

Latest releases of wrbldnsd are available on GitHub:

https://github.com/itefixnet/wrbldnsd/releases

Each release includes:
- The complete wrbldnsd ZIP package (Rbldnsd, Rsync, Cygwin runtime)
- NSIS installer for easy setup
- Release notes and version history

## Installation

1. Download and unzip the wrbldnsd archive

2. Run the NSIS installer:
   - Accept the License agreement
   - Specify an installation location
   - Click 'Details' button for detailed installation information
   - Complete the installation

3. The installer will set up the Rbldnsd service and create the configuration script at `/rbldnsd.sh`

## Basic Usage

### Configuration

Edit the `/rbldnsd.sh` script to configure your DNSBL server:

```bash
#!/bin/bash

# Script to start rbldnsd with desired parameters, check man page for other options
# NB! not all of them are applicable as this is a Windows environment

# address - bind to (listen on) this address (required)
export BINDINFO=127.0.0.1/53

# each zone specified using `name:type:file,file...'
export ZONES=localzone:ip4set:local.zone

# working directory
export WORKDIR=/work

#Start rbldnsd
/bin/rbldnsd -n -f -b $BINDINFO -w $WORKDIR $ZONES
```

### Setup Steps

1. **Update the IP address and port** (BINDINFO):
   - The default value `127.0.0.1/53` is non-functional
   - Configure to listen on the desired network interface and port

2. **Copy dataset files** to the `/work` folder and configure the ZONES parameter:
   - Specify zone name, type (e.g., ip4set), and data file
   - Multiple zones can be configured

3. **Start the service**:
   - The service is set up in automatic mode
   - Monitor activity via the log file in `/var/log` folder

### Additional Options

Many other options can be configured via the script file. Check the man page for available options.

wrbldnsd is tested successfully for serving DNSBL zones on Windows. You should test and verify that it works for your specific needs.

## Links

- **Rbldnsd homepage**: https://rbldnsd.io/
- **Rbldnsd documentation**: https://rbldnsd.io/documentation/
- **Rsync homepage**: http://rsync.samba.org/
- **Cygwin homepage**: http://www.cygwin.com/

## License

wrbldnsd is licensed under the BSD 2-Clause License. See LICENSE file for details.
