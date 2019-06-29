qubes.SshKeyInstall
===================

## Description

This is an Qrexec3 RPC service for Qubes OS.

It can be used to install an SSH key on a target VM; similar to `ssh-copy-id(1)`.

The primary use case is to send keys from a VM where they are stored to another VM that wants to use a key. For example, you may have your SSH keys stored in an offline VM (*vault-ssh*) and find yourself repeatedly copying them over to a DisposableVM whenever they are needed.

### Legend

| System | Role        |
| :----- | :---------- |
| Source VM | SSH key storage |
| Target VM | SSH client |

The RPC service is invoked from the source VM and executed by the target VM.

### Contents

| File | Description |
| :--- | :---------- |
| qubes.SshKeyInstall.rpc | Script that is executed as RPC service |
| qubes.SshKeyInstall.policy | Permissions (ACL) for RPC service |

Currently it is just a backend service that needs to be invoked via the `qrexec-client-vm` utility, and writes an output file from key data passed on STDIN. A notification is sent from the target VM when the key is successfully installed, or if an error would occur.

Some kind of fancy wrapper script is on the roadmap.

## Installation

### Service

The RPC service should be installed on the target VM:s that is to receive keys.

In most cases, this will be an AppVM or DisposableVM so configuration needs to be done in the corresponding TemplateVM (otherwise our changes are discarded between restarts). In the case of a StandaloneVM it can be configured directly as it will keep changes between restarts.

Copy the service file to your TemplateVM/StandaloneVM and install it:

```
# cp qubes.SshKeyInstall.rpc /etc/qubes-rpc/qubes.SshKeyInstall
# chown root:root /etc/qubes-rpc/qubes.SshKeyInstall
# chmod 755 /etc/qubes-rpc/qubes.SshKeyInstall
```

_Please note that the destination file must have no .rpc file extension._

### Policy

Then copy the policy file to dom0 and install it:

```
# cp qubes.SshKeyInstall.policy /etc/qubes-rpc/policy/qubes.SshKeyInstall
# chown root:root /etc/qubes-rpc/policy/qubes.SshKeyInstall
# chmod 644 /etc/qubes-rpc/policy/qubes.SshKeyInstall
```

_Please note that the destination file must have no .policy file extension._

By default, the policy is configured to ask every time the RPC service is invoked.

This can be changed by editing the policy file, for example to allow or deny specific VM:s automatically. Please see the official documentation for more details: [Command execution in VMs - Qrexec3 | Qubes OS](https://www.qubes-os.org/doc/qrexec3/#qubes-rpc-administration).

## Usage

To send an SSH key to a VM that has the RPC service enabled, invoke the Qubes OS qrexec client with your SSH key passed on STDIN.

### Example

Install key file `~/.ssh/id_key` on VM `disp5830`:

```
$ qrexec-client-vm disp5830 qubes.SshKeyInstall < ~/.ssh/id_key
```

The remote key will be stored in the SSH directory of the target user, named with an `id_` prefix followed by a random string. Example:

`~/.ssh/id_qubesplDztu`


If you want to specify a given name for the key file, pass it as an RPC argument (delimited by a plus sign):

```
$ qrexec-client-vm disp5830 qubes.SshKeyInstall+id_secret < ~/.ssh/id_key
```

_NOTE: File names will be mangled according to how RPC arguments are encoded/escaped by Qubes OS._
