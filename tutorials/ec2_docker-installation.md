# Docker Installation On AWS EC2.

Using virtual machines, makes it very important and necessary to utilize containers. Containers help massively in
providing relevant isolation, and giving us proper control over the projects/applications/services that we add to our VM instances.

Containers have also grown to become a very crucial ingredient in project development and deployments. In today's world using containers is simply inevitable - and Docker is the most popular service that provides affords us that super-power.

Here’s the most **standard and secure way** to install Docker on an EC2 Ubuntu instance (Ubuntu 20.04, 22.04, etc.):

## Resources.

Docker ubuntu linux installation docs: https://docs.docker.com/engine/install/ubuntu/

## Chores.

### Step-by-step Docker Installation on EC2 Ubuntu.

#### 1. Set up Docker's apt repository.

```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

#### 2. Install the Docker packages.

- To install the latest version, run:

```bash
 sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

- To install a specific version:

    - To install a specific version of Docker Engine, start by listing the available versions in the repository:

        ```bash
            # List the available versions:
            apt-cache madison docker-ce | awk '{ print $3 }'

            5:28.1.1-1~ubuntu.24.04~noble
            5:28.1.0-1~ubuntu.24.04~noble
        ```

    - Select the desired version and install:

        > `installing  5:28.1.1-1~ubuntu.24.04~noble` 

        ```bash
            VERSION_STRING=5:28.1.1-1~ubuntu.24.04~noble
            sudo apt-get install docker-ce=$VERSION_STRING docker-ce-cli=$VERSION_STRING containerd.io docker-buildx-plugin docker-compose-plugin
        ```

#### 3. Verify that the installation is successful by running the `hello-world` image.

```bash
sudo docker run hello-world
```

