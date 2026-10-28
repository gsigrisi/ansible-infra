# ansible-infra
#

## About
This is a collection of Ansible roles and playbooks for lab work: Ansible Automation Platform, Red Hat Virtualization, VMware, host post-configuration, and Red Hat Satellite. It covers execution environments, controller configuration, virtual machines, Vault-backed users, and the product features I want to try without doing the setup by hand.

Playbooks are written to run from the CLI and as Automation Platform job templates.

## What the code can do

### Ansible Automation Platform and Automation Hub (`playbooks/aap/`, `playbooks/ah/`, `playbooks/ee_utilities/`)
- Configure Controller users and teams after installation, and refresh a job-template survey from the current host list.
- Build custom execution environments and push images to execution nodes.
- Download the AAP installer and prepare a host for installation. Containerized AAP 2.5 setup files live under `installers/`.
- Sync Automation Hub repositories and the execution-environment registry.

### Virtualization
- **Red Hat Virtualization / oVirt** (`playbooks/rhv/`, `playbooks/infra/`, `playbooks/ovirt-rename-disks.yml`): create a host, rename VM disks, back up the Hosted Engine, and shut an environment down.
- **VMware** (`playbooks/vmware/`): upload and convert a VMDK, create a template, create a VM from that template, add the host to inventory, run post-install, add swap, associate the VM with a compute resource, and clean leftover datastore files.

### Host and lab tasks (`playbooks/infra/` and top-level playbooks)
- Create Linux users, including users whose secrets come from HashiCorp Vault, and register or migrate a host.
- Install and initialize Vault on RHEL (KV v2 engines, team paths, AppRole checks) and install the RHEL web console.
- Patch RHEL through host collections and build a patch report. Run an OpenSCAP scan.
- Update Cloudflare DNS A records, install GitLab, and install Image Builder (`osbuild-composer`).

### Red Hat Satellite 6 (`playbooks/satellite/`)
- Post-configure a Satellite server: repositories, third-party products, global parameters, installer, manifest, locations, lifecycle environments, activation keys, and sync plans.
- Search, list, and sync repositories, and manage content-view versions.
- Register, delete, and group hosts, and provision from an image, iPXE, or a VMware boot disk.
- Build golden images and turn them into templates, including a KVM template path.

Reusable roles under `roles/` cover Satellite configuration (`ldelouw.rh-satellite`), Linux users, SSH keys, GitLab, callbacks, oVirt disk rename, and Hosted Engine backup. Galaxy collections are listed in `collections/requirements.yml`. A longer directory reference is in [docs/README.md](docs/README.md).

## Author, Copyright and License
This repo contains code created by Gianfranco Sigrisi.\
It is free software licensed under the terms of the GNU General Public License GPL v3 or later.

## Motivation
I've wrote this Playbook because I do a lot of experiments in my LAB.\
On a daily base, I use Red Hat Satellite, Ansible Automation Platform and Red Hat Virtualization to try new stuff and
sometimes it breaks.\
\
Manual installation takes a lot of time and all I want to do is *automate*.

## Scope
This playbook is not meant for daily tasks such as creating and provision new hosts.\
It can be useful in case specific operations are needed after the initial base OS installation.

## How to use
The code has been created to be used both from cli and Ansible Automation Platform.

### Variables Examples

```
global_parameters:
  - parameter_type: 'json'
    name: "kernel_settings_sysctl"
    value: [{ name: fs.file-max, value: 400000 },{ name: kernel.threads-max, value: 65536},{ name: vm.swappiness,value: 40}]
  - parameter_type: 'json'
    name: timesync_ntp_servers
    value: [{ hostname: 82.197.164.46, iburst: true },{ hostname: 82.197.188.130, iburst: true}]
  - parameter_type: 'array'
    name: "resolv_search"
    value: [domain1.com, domain2.com]
  - { name: ansible_roles_check_mode, value: false  }
  - { name: bootloader-append, value: nofb }
  - { name: disable-firewall, value: 'false' }
  - { name: enable-epel, value: false }
  - { name: enable-luks, value: false }
  - { name: enable-puppet5, value: false }
  - { name: enable-remote-execution-pull, value: false }
```

## Contributions
Contributions are welcome, just send me a pull-request.
