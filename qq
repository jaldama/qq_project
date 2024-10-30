#!/usr/bin/python3

import subprocess
import os
import argparse
import sys
import time
import re
import webbrowser

# Function to install a package using pip
def install_package(package):
    try:
        subprocess.check_call([sys.executable, '-m', 'pip', 'install', package])
    except subprocess.CalledProcessError as e:
        print(f"Failed to install {package}: {e}")
        sys.exit(1)

# Try importing pxssh and handle import errors
try:
    from pexpect import pxssh
except ImportError:
    print("The 'pexpect' library is not installed.")
    install = input("Would you like to install it? (yes/no): ").strip().lower()
    if install in ['yes', 'y']:
        print("Installing 'pexpect'...")
        install_package('pexpect')
        from pexpect import pxssh  # Try importing again after installation
    else:
        print("Please install 'pexpect' manually to use this script.")
        sys.exit(1)

ssh_config_path = os.path.expanduser('~/.ssh/config')

def ssh_login(hostname):
    """Establish an SSH connection and return the session."""
    try:
        s = pxssh.pxssh()
        #s.logfile = open('ssh_log.txt', 'wb') #---Uncomment line for prompt Logging
        s.login(hostname, ssh_config=ssh_config_path)
        s.sendline('export PS1="[PEXPECT]\\$ "')
        s.prompt()
        return s
    except pxssh.ExceptionPxssh as e:
        print("pxssh failed on login.")
        print(str(e))
        return None

def execute_on_remote(s, command):
    """Execute a command on the remote server using an established SSH session."""
    s.sendline(command)
    s.prompt()
    output = remove_ANSI(s.before.decode('utf-8'))
    if output.startswith(command):
        output = output[len(command):].strip()
    return output

def remove_ANSI(string):
    output = re.sub(r'(\x1b\[[0-?9]*[lh])', '', string) #removes ANSI escape characters from captured server output
    return output

def list_directory(hostname, ticket):
    """List the ticket directory's contents"""
    s = ssh_login(hostname)
    if s:
        command = f'ls -halLt /opt/nfs/vcf_gs_csp_prd_sr/{ticket}.bcm'
        output = execute_on_remote(s, command)
        if output:
            print(output)
        s.logout()

def rsync_directory(hostname, ticket):
    """Rsync the contents of the ticket directory to the current local directory"""
    s = ssh_login(hostname)
    if s:
        # Rsync ticket directory contents to remote:/home/USER
        rsync_command = (
            f'rsync -av --partial '
            f'--exclude "*/" '
            f'--exclude "*-link" '
            f'--exclude "*renamed-by-CDM*" '
            f'--exclude "*vcf-cdm-copy*" '
            f'--exclude "*vmx - Vi*" '
            f'/opt/nfs/vcf_gs_csp_prd_sr/{ticket}.bcm/ ~/.{ticket}.scp'
        )

        print("Attempting to download all non-directory artifacts:")
        output = execute_on_remote(s, rsync_command)
        if output:
            print(output)

        # Execute the final rsync command to copy files from remote:/home/USER to current local directory
        local_command = [
            'rsync',
            '-av',
            '--progress',
            '--partial',
            '--partial-dir=.Unfinished',
            f'{hostname}:~/.{ticket}.scp/*',
            '.'
        ]

        try:
            process = subprocess.Popen(local_command, stdout=subprocess.PIPE, stderr=subprocess.PIPE, universal_newlines=True)
            print()
            for line in process.stdout:
                if '%' in line:
                    print(f"\r{line.strip()}", end="")
                    sys.stdout.flush()
            process.wait()

        except subprocess.CalledProcessError as e:
            print("Error occurred:")
            print(e.stderr)

        # Clean up the temporary directory on the remote server
        execute_on_remote(s, f'rm -rf ~/.{ticket}.scp')
        s.logout()

def gui_server(hostname, ticket):
    """Opens a web browser interface for viewing and downloading remote ticket directory"""
    try:
        s = ssh_login(hostname)
        if s:
            command = 'ip -4 -br -j  address show ens192 | jq -r .[].addr_info[].local'
            ip_number = execute_on_remote(s, command)
            command = f'python -m http.server -d /opt/nfs/vcf_gs_csp_prd_sr/{ticket}.bcm/ 0'
            s.sendline(command)

            # Check the output for the port number and extract
            s.prompt(timeout=1)
            output = s.before.decode('utf-8').strip()
            port_number = capture_port(output)

            # Start web browser at the ticket remote directory
            print(f"HTTP server is running at http://{ip_number}:{port_number}. Press Ctrl+C to stop.")
            webbrowser.open(f'http://{ip_number}:{port_number}')

            # After web browser starts, block script until User Ctrl+C
            while True:
                time.sleep(1)

    except Exception as e:
        print(f"An error occurred: {e}")

def capture_port(url):
    pattern = r':(\d+)/'
    match = re.search(pattern, url)
    if match:
        port_number = match.group(1)
    return port_number

def main():
    parser = argparse.ArgumentParser(description="List and copy compressed files from script server to local.")
    parser.add_argument('-ls', '--list', type=str, nargs=2, metavar=('hostname', 'ticket'), help='List ticket directory contents.')
    parser.add_argument('-cp', '--copy', type=str, nargs=2, metavar=('hostname', 'ticket'), help='Copy regular files from script server to local.')
    parser.add_argument('-gui', '--graphical', type=str, nargs=2, metavar=('hostname', 'ticket'), help='Create a graphical interface for downloading items in ticket directory.')

    args = parser.parse_args()

    # Check if no arguments were provided and display help
    if not any(vars(args).values()):
        parser.print_help()
        sys.exit(0)

    actions = {
        'list': (args.list, list_directory),
        'copy': (args.copy, rsync_directory),
        'graphical': (args.graphical, gui_server),
    }

    for action, (arg, func) in actions.items():
        if arg:
            hostname, ticket = arg
            func(hostname, ticket)

    #os.system('printf "\e[?2004l"') # Safety system call for ending bracketed-paste. Uncomment if pasted output frequently captures ANSI Escape characters

if __name__ == '__main__':
    main()
