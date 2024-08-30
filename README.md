# creodias-customisation-script-pgstac
Fast deployment of pgstac (via docker) and pypgstac (via pip) on CREODIAS infrastructure

## Customisation Script
You can customise your instance after it has launched using the options available here. "Customisation Script" is analogous to "User Data" in other systems.

```console
#!/bin/bash
# Update the package list
sudo apt-get update

# Install required packages with automatic yes
yes | sudo apt-get install ca-certificates curl

# Create the directory for Docker keyrings
sudo install -m 0755 -d /etc/apt/keyrings

# Download the Docker GPG key
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc

# Ensure the keyring file is readable
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add Docker repository to APT sources
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
$(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Update the package list again
sudo apt-get update

# Install Docker components with automatic yes
yes | sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Install pypgstac
python3 -m pip install pypgstac[psycopg]
export PATH=\$PATH:/home/eouser/.local/bin

# Export initial env variables:
export POSTGRES_USER=
export POSTGRES_PASSWORD=
export POSTGRES_DB=
export PGUSER=
export PGPASSWORD=
export PGDATABASE=

export postgres_ram="2g"
export postgres_cpus=2


# Run pgstac via installed docker:
sudo docker run -d \
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
  postgis/postgis:17beta3-master

# Base migrations install PgSTAC into a database with no current PgSTAC installation
pypgstac migrate
```