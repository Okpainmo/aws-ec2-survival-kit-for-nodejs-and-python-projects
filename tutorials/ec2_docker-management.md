# Docker Management.

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

## Setting Up Docker To Work as a Jenkins Agent.

Only do this **in controlled environments** (e.g., private network, 
behind a VPN and/or restrict IP access to only the VM running the 
Jenkins controller).

### Steps (for security reasons, ensure to open and restrict port 2375 access to only the jenkins master VM IP - also do well to use TLS and switch to port 2376):

1. **Edit Docker service file on EC2**:

```bash
sudo nano /lib/systemd/system/docker.service
```

Find the `ExecStart` line and change it from:

```bash
ExecStart=/usr/bin/dockerd -H fd:// # or whatever
```

To:

```bash
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock
```

> The above, (kind of) hacks docker on the current(agent) VM, and grants access(via TCP) to a remote Jenkins controller, which can then connect and control the docker service/daemon REMOTELY - thus giving the Jenkins controller the ability to use it as an agent.

2. **Reload and restart Docker**:

```bash
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl restart docker
sudo systemctl status docker
```

3. **Open port 2375 in your EC2 instance security group and grant access to ONLY the Jenkins master VM IP address**. 

- Example Use Case

Allow port 8080 access only from `198.51.100.10`:

* **Type:** Custom TCP
* **Port Range:** 8080
* **Source:** `198.51.100.10/32` - Custom TCP(IPV4)

> Do well to add TLS(Transport Layer Security) and switch to port 2376 - while still restricting access to only the Jenkins controller VM IP address.

4. Go to **Jenkins → Manage Jenkins → Clouds → Add a cloud**

5. Select Docker(Options are AWS EC2, Kubernetes, Docker, and 'Copy Existing Cloud' if you already created one) then give the cloud agent a name and proceed.

6. **On the "Docker Cloud details" drop-down/menu**

    - Docker Host URI

        - Add 

        ```bash
        tcp://ip_of_Jenkins_controller:2375
        ```

    - Test the connection.

        - should return a response like below:

        ```bash
        Version = 28.1.1, API Version = 1.49
        ```

    - Tick 'Enabled', and save.

**On "Docker Agent templates" drop-down/menu**

    - Add a label(identifier) for the docker agent template.
    - Tick 'Enabled'.
    - Add a name - usually same as label above.
    - Add the docker image to use on the agent(the beauty of Docker as a Jenkins agent - flexibility of environments, and parallelism). 

    E.g.

    ```bash
    okpainmo/jenkins_nodejs_agent_linux_deb:6.5.25
    ```

    - Set instance capacity - the number of containers the agent is allowed to spin(3 - 5 should be good I guess).
    - Remote File System Root.

    ```bash
    /var/jenkins_home # for docker
    ```

    - Study(click the '?' marks on the Jenkins UI) other options and make appropriate choices - especially for "Idle timeout" and "Stop timeout".
    - Apply and Save.
    - Then use in your declarative pipeline scripts.

    ```groovy
    pipeline {
        agent {
            docker {
                // image 'node:22'
                label 'agent_name'// e.g. 'my_docker-agent'
                // args '-v /tmp:/tmp'
            }
        }

        stages {    
            stage('Build') {
                steps {
                    sh 'whoami'                   
                }
            }
        }
    }
    ```

**Study on the differences between, and the pros/cons of using Docker as a Jenkins CLOUD agent, and using a NODE(SSH) agent.**

