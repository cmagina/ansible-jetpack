# ansible-jetpack

## Supported Systems

### Orin

#### Target Platforms

* agx
  * devkit
  * 32g
  * industrial
* nx
  * 16g
  * 8g
* nano
  * devkit
  * 8g
  * 4g

#### Software Versions

* 36.3.0 (JetPack 6.0)
* 36.3.1 (IGX SW 1.0)
* 36.4.0 (JetPack 6.1)

## Requirements

### Ansible

Install the Ansible requirements by running this command (requirements.yml is in the ansible-jetpack repo):

```bash
ansible-galaxy install -r requirements.yml
```

## Flash

### Configuration

#### Copy and fill out the sample inventory file

``` inventory.yml
flash_hosts:
  hosts:
    x86_host:
      ## flash config
      jetpack_platform_module: <agx | nx | nano>
      # jetpack_platform_module_type: <devkit | 32g | 16G | 8g | 4g | industrial>
      # jetpack_flash_type: <full | boot > # boot is the default
      # optional target path to extract the jetpack packages to, uses a tempdir by default
      # jetpack_work_path: /path/to/workdir

jetsons:
  hosts:
    agxorin:
      ## adds each user to the video group
      # additional_jetpack_users:
      #  - test

all:
  vars:
    ## target jetpack release
    jetpack_software_release: 36.4.0
```

#### flash_hosts

Each host in this list needs to be x86_64 and connected to a Jetson's flash port. The Jetson driver package and all dependencies will be installed to each host and flash will be run from there. The supported operating systems for these hosts can be either Fedora 39+ (currently for a boot flash only) or Ubuntu 20.04 and 22.04 (both boot and full flash).

#### jetsons

A list of the jetson hosts to install NVIDIA JetPack 6 to.

## Playbook Roles

* flash
* jetpack

### flash

Setup each flash host with the specified firmware and run the configured flash command for the target jetson.

### jetpack

Builds and installs the NVIDIA OOT kernel modules, pulled from their respective git repos. Downloads and extracts the required tools, libraries and services from the Debian packages pulled from the NVIDIA provided Debian respositories. Updates are made to the userspace components to support them being installed into a RHEL OS versus Ubuntu. And finally installs the NVIDIA Container Toolkit runtime and adds a service to generate the container device inferface (CDI) config on boot. This allows NVIDIA containers to be launched using podman on RHEL with full access to the GPU.

##### Red Hat Subscription Manager Credentials

The first step for the jetpack role is to register the RHEL system so that it can download software dependencies. If the system has already been registered or does not need to be registered to download the needed software, not providing the credentials will cause the step to be skipped. If providing the credentials is needed, create a file, `rhsm_credentials.enc`, with these contents:

``` rhsm_credentials.enc
rhsm_username: <user@email.com>
rhsm_password: password1234
```

Then encrypt it using Ansible Vault like this:

```bash
ansible-vault encrypt rhsm_credentials.enc
```

To provide the credentials to the ansible playbook, add these arguments to the command:

```bash
--ask-vault-password --extra-vars @/path/to/rhsm_credentials.enc
```

## Run the playbooks

### Flash

```bash
ansible-playbook --ask-become-pass --ask-pass --inventory /path/to/inventory.yml /path/to/flash.yml
```

### Jetpack

```bash
ansible-playbook --ask-become-pass --ask-pass --inventory /path/to/inventory.yml /path/to/jetpack.yml
```
