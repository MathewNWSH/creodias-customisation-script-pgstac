# creodias-customisation-script-pgstac
Fast deployment of pgstac (via docker) and pypgstac (via pip) on CREODIAS infrastructure

## Customisation Script
You can customise your instance after it has launched using the options available here. "Customisation Script" is analogous to "User Data" in other systems.

```console
#!/bin/bash
# Update the package list
apt-get update

# Install required packages with automatic yes
apt-get install -y ca-certificates curl

# Create the directory for Docker keyrings
install -y -m 0755 -d /etc/apt/keyrings

# Download the Docker GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc

# Ensure the keyring file is readable
chmod a+r /etc/apt/keyrings/docker.asc

# Add Docker repository to APT sources
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
$(. /etc/os-release && echo "$VERSION_CODENAME") stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null

# Update the package list again
apt-get -y update

# Install Docker components with automatic yes
apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Install pypgstac
python3 -m pip install pypgstac[psycopg]

# Export initial env variables:
export POSTGRES_USER=
export POSTGRES_PASSWORD=
export POSTGRES_DB=
export PGUSER=
export PGPASSWORD=
export PGDATABASE=

export PGHOST=0.0.0.0
export PGPORT=5432

export postgres_ram="2g"
export postgres_cpus=2

export POSTGRES_IMAGE="postgis/postgis:17beta3-master"


# Run pgstac via installed docker:
docker run -d \
  --name stac-db \
  --restart unless-stopped \
  --memory="${postgres_ram}" \
  --cpus="${postgres_cpus}" \
  -e POSTGRES_USER=${POSTGRES_USER} \
  -e POSTGRES_PASSWORD=${POSTGRES_PASSWORD} \
  -e POSTGRES_DB=${POSTGRES_DB} \
  -e PGUSER=${PGUSER} \
  -e PGPASSWORD=${PGPASSWORD} \
  -e PGDATABASE=${PGDATABASE} \
  -p 5432:5432 \
  -v stac-db-data:/var/lib/postgresql/data \
  ${POSTGRES_IMAGE}

# Base migrations install PgSTAC into a database with no current PgSTAC installation
pypgstac migrate
```
# How to use?
Compile the lunch instance tab as instructed and paste the above code into:
![Logo](graph_.png)
and run VM.