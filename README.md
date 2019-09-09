[![Build Status](https://travis-ci.com/JyrgenSuvalov/X-Road-cs-api.svg?branch=master)](https://travis-ci.com/JyrgenSuvalov/X-Road-cs-api)

# Central Server API

This API is used to add new X-Road members directly to X-Road Central Server without web admin interface.

An official management API will be included in a future X-Road release. You can find the roadmap for X-Road [here](https://www.niis.org/xroad-roadmap).

**NB! Make sure your API is not accessible from public internet, and is properly secured in your internal network!**

## API description

API is described using OpenAPI specification: [openapi-definition.yaml](openapi-definition.yaml)

## Installation

Installation was tested with Ubuntu 18.04.

### Program

Provided systemd and nginx configurations assume than program files are installed under `/opt/csapi`. Program is running under `xroad` user to be able to access X-Road configuration files and database without any additional configurations.

Create `/opt/csapi` directory:
```bash
sudo mkdir -p /opt/csapi
```

And copy files `member.py`, `server.py`, and `requirements.txt` into `/opt/csapi` directory.

You will need to install support for python venv:
```bash
sudo apt-get install python3-venv
```

Then install required python modules into venv:
```bash
cd /opt/csapi
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### Systemd configuration

Add service description `systemd/csapi.service` to `/lib/systemd/system/csapi.service`. Then start and enable automatic startup:
```bash
sudo systemctl daemon-reload
sudo systemctl start csapi
sudo systemctl enable csapi
```

### Nginx configuration

Add nginx configuration from this repository: `nginx/csapi.conf` to nginx server: `/etc/nginx/sites-enabled/csapi.conf`

Create a certificate for nginx (installed by default in X-Road Central Server):
```bash
sudo mkdir -p /etc/nginx/csapi
cd /etc/nginx/csapi
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout csapi.key -out csapi.crt
```

Cert info (NB! CN should be the domain name of your central server):
```
Country Name (2 letter code) [AU]:EE
State or Province Name (full name) [Some-State]:Harjumaa
Locality Name (eg, city) []:Tallinn
Organization Name (eg, company) [Internet Widgits Pty Ltd]:RIA
Organizational Unit Name (eg, section) []:CSAPI
Common Name (e.g. server FQDN or YOUR name) []:central-server.domain.local
Email Address []:
```

Make sure key is accessible to nginx:
```bash
sudo chmod 640 /etc/nginx/csapi/csapi.key
sudo chgrp www-data /etc/nginx/csapi/csapi.key
```

On client side (XTSS app):
```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout client.key -out client.crt 
```

Cert info:
```
Country Name (2 letter code) [AU]:EE
State or Province Name (full name) [Some-State]:Harjumaa
Locality Name (eg, city) []:Tallinn
Organization Name (eg, company) [Internet Widgits Pty Ltd]:RIA
Organizational Unit Name (eg, section) []:APIClient
Common Name (e.g. server FQDN or YOUR name) []:
Email Address []:
```

Copy client.crt to Central Server machine: `/etc/nginx/csapi/client.crt`

For testing copy nginx `csapi.crt` to client and issue curl command:
```bash
curl --cert client.crt --key client.key --cacert csapi.crt -i -d '{"member_code": "XX000003", "member_name": "XX Test 3", "member_class": "GOVXXX"}' -X POST https://central-server.domain.local:5443/member
```

Note that you can allow multiple clients (or nodes) by creating certificate bundle. That can be done by concatenating multiple client certificates into single `client.crt` file.

## Testing

Note that `server.py` is a configuration file for logging and Flask and therefore not covered by tests.

Running the tests:
```bash
cd <project_directory>
python -m unittest
```

Or alternatively run the test file directly:
```bash
python test_member.py
```

In order to measure code coverage install `coverage` module:
```bash
pip install coverage
```

Then run coverage analyse:
```bash
coverage run test_member.py
coverage report member.py
```

Alternatively you can generate html report with:
```bash
coverage run test_member.py
coverage html member.py
```

In order to lint the code install `pylint` module:
```bash
pip install pylint
```

Then run the analyse:
```bash
pylint member.py
```
