# qq_CLI


For Viewing/Downloading files from VCF script server

Requirements:
Python 3.6 or >
Must be on VPN
Unique Script Server entry in .ssh/config file


Script will attempt to install Pexpect via PIP if not already present

Syntax:
qq -[Option] HOST_ENTRY TICKET_NUMBER

Options:

-ls --list (List contents of remote Script Server ticket directory)

-cp -copy (Copy contents from remote Script Server ticket directory to local current directory)

-gui --graphical (Start a web server serving the contents of the directory, and opens web page to server address)
