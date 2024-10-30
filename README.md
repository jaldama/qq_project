# qq_CLI


For Viewing/Downloading files from VCF script server

Requirements:

Python 3.6 or >

Must be on VPN

Unique Script Server entry in .ssh/config file


Script will attempt to install Pexpect via PIP if not already present


Syntax:
usage: qq hostname [-ls] [-cp] [-gui] ticket

```List and copy compressed files from script server to local.```

positional arguments:
  hostname              The hostname of the script server.

optional arguments:
  -h, --help            show this help message and exit
  -ls ticket, --list ticket
                        List ticket directory contents.
  -cp ticket, --copy ticket
                        Copy regular files from script server to local.
  -gui ticket, --graphical ticket
                        Create a graphical interface for downloading items in ticket directory.
