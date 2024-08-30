# creodias-customisation-script-pgstac
Fast deployment of pgstac (via docker compose) and pypgstac (via pip) on CREODIAS infrastructure

## Customisation Script
You can customise your instance after it has launched using the options available here. "Customisation Script" is analogous to "User Data" in other systems.

```
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
```