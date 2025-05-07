# SSH Server Communications.

DevOps as a whole, is a craft that involves a lot of remote interactions between servers. The main protocol for communicating/interaction between two or
more servers remains SSH(Secure Shell).

SSH(Secure Shell) is a communication protocol used to securely connect and communicate with remote systems over a network. It is widely used for:

- Remote login to a server (like accessing a Linux server from your local machine)

- Secure file transfer (using tools like scp or sftp)

- Running remote commands or managing systems programmatically

> SSH is very powerful. It even allows you to communicate **directly between two remote servers**, not just with/via your local server or machine.

## How SSH Actually Works.

Below is the SSH snippet to access an AWS EC2 VM from my local server.

```bash
ssh -i "path_to_private_key_file" user@remote-host
```

E.g.

```bash
ssh -i "path_to_private_key_file" ubuntu@ip_of_the_vm
```

The above snippet pattern is the same for every EC2 SSH connection. Below is a list detailing how SSH actually works - from key-pair generation, to connection, and more.

1. Create/Generate a secure key-pair on(or for - I explained 'delegated SSH connections' below) the machine-to-connect-from(client).
2. Add the public key of the machine-to-connect-from(client) into the authorized keys file of the machine-to-connect-to(server).
3. Using the above snippet, add the relevant details(IP and private key file path), then paste and attempt to create a connection on the
machine-to-connect-from(client). 

> Here's what then happens: When an SSH connection is attempted, the client(machine-to-connect-from) uses the provided private key to respond to a cryptographic challenge issued by the server(machine-to-connect-to). The server checks whether the public key(already stored in authorized_keys) matches the private key's response. If the keys match, it then grants access. Else the connection will be rejected. The private key must remain only on the client - it must be kept private.

## Delegated SSH Connections.

From the above, you might already be wondering - "...but when I'm connecting to an AWS VM from my local machine, I never actually created the key-pair on my local machine".

> That's a very valid concern, but that simply explains delegated SSH connections.

SSH is simple and straight-forward. All you actually need, is a pair of keys. The key-pair does not necessarily have to be generated on the machine-to-connect-from(client). **What's important is that the client machines has the private key to provide(and be verified), for(against) a public key which is already on the machine-to-connect-to(server)**.

This simple concept is what makes it possible for two remote machines(even virtual machines(VMs)) to connect to with each other. A very practical example, is how you're able to connect a remote Jenkins(master/controller) server with a remote project-host server, and programmatically perform actions on the project-host server without any direct human actions.

## Practical(Server-Client) Connection Demonstration.

1. Generate the key pair on the client. 

```bash
ssh-keygen -t rsa -b 4096 -C "comment" -f ~/.ssh/name-of-key
```

e.g.

```bash
ssh-keygen -t rsa -b 4096 -C "jenkins@agent" -f ~/.ssh/id_rsa_jenkins_agent # creates the keys, and reveals where to access both the private and public keys.
```

2. Copy the public key.

On Ubuntu(I guess on all other linux distros as well), the public key should be on the paths shown below. Access it, and copy the public key.

```bash
cat /home/ubuntu/.ssh/name-of-key.pub # this reveals the public key.
```

e.g.

```bash
cat ~/.ssh/name-of-key.pub # this reveals the public key.
```

Similarly, the private key should be here:

```bash
cat /home/ubuntu/.ssh/name-of-key # this reveals the private key.
```

e.g.

```bash
cat ~/.ssh/name-of-key # this reveals the private key.
```

3. Head to the remote machine to connect from, and run this.

```bash
nano ~/.ssh/authorized_keys
```

4. Then add the newly copied(public) key on a new line.

5. Save(CTRL o, then ENTER, then CTRL x).

6. Return to the host/client machine that the key was created on, and run:

```bash
ssh -i path-to-private-key ubuntu@ip-of-remote-server-to-connect-to

# E.g.

ssh -i ~/.ssh/id_rsa_jenkins_agent ubuntu@12.54.153.100
```

7. Just like that, you should find yourself inside the shell of the remote EC2 instance.

8. Check all your SSH keys.

```bash
ls ~/.ssh
```

## Explaining The Default Key Filenames.

`id_rsa` is the **default private key filename** used by OpenSSH when generating SSH key pairs.

Here’s a breakdown:

### What is `id_rsa`?

* `id_rsa`: Your **private SSH key**.
* `id_rsa.pub`: The corresponding **public SSH key**.

### Where are they located?

By default, when you run:

```bash
ssh-keygen
```

and accept all the defaults (i.e., press Enter for all prompts), it creates:

* `~/.ssh/id_rsa` – private key
* `~/.ssh/id_rsa.pub` – public key

These are stored in the `~/.ssh/` directory in your home folder.

### Why is `id_rsa` the default?

SSH clients like `ssh`, `scp`, `git`, and Jenkins SSH plugins will automatically look for `~/.ssh/id_rsa` when connecting, unless explicitly told to use another key (via `-i` flag or config file).

E.g.

```bash
ssh -i ~/.ssh/id_rsa_jenkins_agent ubuntu@12.54.153.100
```

### 🔒 Important Notes:

* The **private key (`id_rsa` or equivalent) must be kept secret**.
* The **public key (`id_rsa.pub` or equivalent) can be shared**, for example, by appending it to `~/.ssh/authorized_keys` on a remote server.

Would you like help generating or managing SSH keys?
