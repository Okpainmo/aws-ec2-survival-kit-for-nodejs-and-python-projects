# SSH Server Communications.

DevOps as a whole, is a craft that involves a lot of remote interactions between servers. The main protocol for communicating/interaction between two or
more servers remains SSH(Secure Shell).

SSH(Secure Shell) is a communication protocol used to securely connect and communicate with remote systems over a network. It is widely used for:

- Remote login to a server (like accessing a Linux server from your local machine)

- Secure file transfer (using tools like scp or sftp)

- Running remote commands or managing systems programmatically

> SSH is very powerful. It even allows you to communicate **directly between two remote servers**, not just with/via your local computer or machine.

## How SSH Actually Works.

Below is the SSH snippet to access an AWS EC2 VM from my local machine(computer).

```bash
ssh -i "path_to_private_key_file" user@remote-host
```

E.g.

```bash
ssh -i "path_to_private_key_file" ubuntu@ip_of_the_vm
```

The above snippet pattern is not unique to AWS. It is the same for every linux SSH connection. It is simply a default command for gaining shell access into remote VMs via SSH.

Below is a list detailing how SSH actually works - from key-pair generation, to connection, and more.

1. Create/Generate a secure key-pair(the private and public keys) on(or for - I explained 'delegated SSH connections' below) the machine-to-connect-from(client).
2. Add the public key of the machine-to-connect-from(client) into the authorized keys file of the machine-to-connect-to(server).
3. Using the above snippet, add the relevant details(IP and private key file path), then copy and attempt to create a connection on the
machine-to-connect-from(client). 

> Here's what then happens: When an SSH connection is attempted, the client(machine-to-connect-from) uses the private key it has, to respond to a cryptographic challenge issued by the server(machine-to-connect-to). The server checks whether the public key(that is already stored in authorized_keys) matches the private key's response. If the keys match, it then grants access. Else the connection will be rejected. The private key must remain only on the machine-to-connect-from(client) - it must be kept private.

## Delegated SSH Connections.

From the above, you might already be wondering - "...but when I'm connecting to an AWS VM from my local machine, I never actually created the key-pair on my local machine".

> That's a very valid concern, but that simply explains delegated SSH connections.

SSH is simple and straight-forward. All you actually need, is a pair of keys. The key-pair does not necessarily have to be generated on the machine-to-connect-from(client). **What's important is that the machine-to-connect-from(client) machine has the private key to provide(and be verified), for/against a public key which is already on the machine-to-connect-to(server)** - irrespective of whether the key-pair was generated on it(the machine-to-connect-from/client) or not.

This simple concept is what makes it possible for two remote machines(even virtual machines(VMs)) to connect to with each other. A very practical example, is how you're able to connect a remote Jenkins(master/controller) server with a remote project-host server, and programmatically perform actions on the project-host server without any direct human actions. You simply extract the keys from where ever they were created(whether directly on the Jenkins server or not), add the public key to 'authorised keys' on the machine-to-connect-to, then add the private key to Jenkins. Thus permitting/**DELEGATING** Jenkins to authenticate VIA SSH and perform actions on your behalf.

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

or

```bash
cat ~/.ssh/name-of-key.pub # this reveals the public key.
```

Similarly, the private key should be here:

```bash
cat /home/ubuntu/.ssh/name-of-key # this reveals the private key.
```

or

```bash
cat ~/.ssh/name-of-key # this reveals the private key.
```

3. Head to the remote machine to connect from, and run this.

```bash
nano ~/.ssh/authorized_keys
```

4. Then add the newly copied(public) key on a new line.

5. Save(CTRL o, then ENTER, then CTRL x).

6. Return to the host/client machine that now has the private key, and run:

```bash
ssh -i path-to-private-key ubuntu@ip-of-remote-server-to-connect-to

# E.g.

ssh -i ~/.ssh/id_rsa_jenkins_agent ubuntu@12.54.153.100
```

7. Just like that, you should find yourself inside the remote shell of the other VM.

**Check all your SSH keys**.

```bash
ls ~/.ssh
```

## Explaining The Default Key-pair Filenames.

By default, when you run:

```bash
ssh-keygen
```

and accept all the defaults (i.e., press Enter for all prompts), it creates:

* `~/.ssh/id_rsa` – private key
* `~/.ssh/id_rsa.pub` – public key

`id_rsa` is the **default key-pair filename** used by OpenSSH when generating SSH key pairs.

Here’s a breakdown:

* `id_rsa`: Your **private SSH key**.
* `id_rsa.pub`: The corresponding **public SSH key**.

### Where are they located?

Of course, these are stored in the `~/.ssh/` directory in your home folder.

e.g. Access them:

```bash
cat ~/.ssh/d_rsa # this reveals the private key.
```

```bash
cat ~/.ssh/d_rsa.pub # this reveals the public key.
```


### `id_rsa` is simply a default.

By default, SSH clients like `ssh`, `scp`, `git`, and Jenkins SSH plugins will automatically look for `~/.ssh/id_rsa` when connecting, unless explicitly told to use another key (via `-i` flag or config file).

E.g.

```bash
ssh -i ~/.ssh/id_rsa_jenkins_agent ubuntu@12.54.153.100
```

### 🔒 Important Notes:

* The **private key (`id_rsa` or equivalent) must be kept secret** - only add to a remote client when you want to delegate access to it.
* The **public key (`id_rsa.pub` or equivalent) can be shared**, for example, by appending it to `~/.ssh/authorized_keys` on a remote server.

