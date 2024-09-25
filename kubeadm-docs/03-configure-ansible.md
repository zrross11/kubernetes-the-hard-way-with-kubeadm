# Setup ssh and configure ansible

From this point on, the steps are *exactly* the same for VirtualBox and Apple Silicon as it is now about configuring Kubernetes itself on the Linux hosts which you have now provisioned.

Begin by logging into `controlplane01` using `vagrant ssh` for VirtualBox, or `multipass shell` for Apple Silicon.

## Access all VMs

Here we create an SSH key pair for the user who we are logged in as (this is `vagrant` on VirtualBox, `ubuntu` on Apple Silicon). We will copy the public key of this pair to the other controlplane and both workers to permit us to use password-less SSH (and SCP) go get from `controlplane01` to these other nodes in the context of the user which exists on all nodes.

Generate SSH key pair on `controlplane01` node:

[//]: # (host:controlplane01)

```bash
ssh-keygen
```

Leave all settings to default by pressing `ENTER` at any prompt.

Add this key to the local `authorized_keys` (`controlplane01`) as in some commands we `scp` to ourself.

```bash
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

Copy the key to the other hosts. You will be asked to enter a password for each of the `ssh-copy-id` commands. The password is:
* VirtualBox - `vagrant`
* Apple Silicon: `ubuntu`

The option `-o StrictHostKeyChecking=no` tells it not to ask if you want to connect to a previously unknown host. Not best practice in the real world, but speeds things up here.

`$(whoami)` selects the appropriate user name to connect to the remote VMs. On VirtualBox this evaluates to `vagrant`; on Apple Silicon it is `ubuntu`.

```bash
ssh-copy-id -o StrictHostKeyChecking=no $(whoami)@controlplane02
ssh-copy-id -o StrictHostKeyChecking=no $(whoami)@loadbalancer
ssh-copy-id -o StrictHostKeyChecking=no $(whoami)@node01
ssh-copy-id -o StrictHostKeyChecking=no $(whoami)@node02
```

For each host, the output should be similar to this. If it is not, then you may have entered an incorrect password. Retry the step.

```
Number of key(s) added: 1
```

Verify connection

```
ssh controlplane01
exit

ssh controlplane02
exit

ssh node01
exit

ssh node02
exit
```

## Install Ansible, clone repo, and run

```bash
# on controlplane01
sudo apt install ansible
git clone https://github.com/zrross11/kubernetes-the-hard-way-with-kubeadm.git
cd kubernetes-the-hard-way-with-kubeadm/ansible
ansible-playbook -i inventory playbook.yaml
```