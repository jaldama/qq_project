# qq_CLI


For Viewing/Downloading files from VCF script server directly to local
```
Requirements:
Pexpect Library --- #Script will attempt to install Pexpect via PIP if not already present

Python 3.6 or >

Must be on VPN

Unique Script Server entry in .ssh/config file. Example of '~/.ssh/config' entry:


Host scriptsrver
    HostName scriptserver.server.com
    User Josh
    IdentityFile ~/.ssh/id_rsa
```






Syntax:
```
usage: qq HOSTNAME [-ls] [-cp] [-gui] TICKET

List and copy compressed files from script server to local.

positional arguments:
  hostname              The hostname of the script server.

optional arguments:
  -h, --help            show this help message and exit
  -ls ticket, --list ticket
                        List ticket directory contents.
  -cp ticket, --copy ticket
                        Copy regular files from script server to local.
  -gui ticket, --graphical ticket
                        Create a graphical interface for downloading items in ticket directory.```
