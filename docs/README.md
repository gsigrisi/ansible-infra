# ansible-infra — Documentation

A collection of Ansible roles, playbooks, inventories and supporting assets used to
**post-configure and operate a Red Hat lab environment**: Red Hat Satellite 6,
Ansible Automation Platform (AAP) / Automation Hub, Red Hat Virtualization (RHV/oVirt)
and VMware. Everything is written to run both from the CLI (via `ansible-navigator` /
execution environments) and from Ansible Automation Platform (job templates + surveys).

> Author: Gianfranco Sigrisi — licensed GPL v3 or later.
> This is a lab toolkit for product setup, host provisioning, and demos.

---

## What kind of code is in here

| Area | Location | Language / type |
|------|----------|-----------------|
| Playbooks | [playbooks/](../playbooks/) | Ansible YAML |
| Roles | [roles/](../roles/) | Ansible roles (mostly vendored/`pinguozzo.*`) |
| Inventories | [inventory/](../inventory/) | Static YAML inventories (dc1, oVirt, AAP, remote sites) |
| Variables | [vars/](../vars/) | Per-OS and per-domain var files |
| Vaults | [vaults/](../vaults/) | Ansible Vault encrypted secrets (per-environment) |
| Config data | [configs/](../configs/) | Controller/Teams/Users config-as-code |
| Survey templates | [template_surveys/](../template_surveys/) | Jinja2 survey/host-list templates |
| Dependencies | [collections/requirements.yml](../collections/requirements.yml), [roles/requirements.yml](../roles/requirements.yml), [pip3-requirements.txt](../pip3-requirements.txt) | Galaxy collections / roles, Python deps |
| Execution env | [ansible-navigator.yml](../ansible-navigator.yml), [.devcontainer/](../.devcontainer/) | Navigator + container/EE config |

---

## Capabilities — what it lets you do

### Red Hat Satellite 6 ([playbooks/satellite/](../playbooks/satellite/))
Setup and content operations for a Satellite server. [config/all.yml](../playbooks/satellite/config/all.yml) imports each step when the matching `set_*` flag in the inventory is true. [config.yml](../playbooks/satellite/config.yml) is a shorter entry point: repositories, third-party products, and global parameters, selected with the tags `enable_repos`, `3rdparty`, and `global_parameters`.

- **Initial setup**: apply the subscription manifest, configure locations, global parameters, and storage/LVM. The installer playbook passes the initial organization as `--foreman-initial-organization`. There is no separate organization playbook. `satellite-installer.yml` and `disconnected.yml` are imported with `tags: never`. The `set_installer` and `set_disconnected` conditions in `config/all.yml` are commented out, so those plays run only when the `installer` or `disconnected` tag is selected.
- **Content management**: manage repositories, sync repos, sync plans, activation keys, lifecycle environments, and third-party products. `disconnected.yml` mounts installation ISOs for an air-gapped prep.
- **Content Views** ([content/](../playbooks/satellite/content/)): create, publish, promote, and clean up Content View versions. `cv_version_create.yml` applies the `redhat.satellite.content_views` role. The older `redhat.satellite.content_view` tasks in that file are commented out.
- **Host operations**: register/delete hosts, host collections, boot-disk & iPXE / image-based
  provisioning ([provisioning/](../playbooks/satellite/provisioning/)), guest-id lookups.
- **VMware golden images**: create, configure, template-ize and clean up VMware golden images
  associated with Satellite.

### Ansible Automation Platform & Automation Hub ([playbooks/aap/](../playbooks/aap/), [playbooks/ah/](../playbooks/ah/), [playbooks/ee_utilities/](../playbooks/ee_utilities/))
- **Execution Environments**: build custom EEs (`ee_builder`), and push/upload them.
- **Configuration-as-code**: manage Controller teams, users and host lists ([configs/](../configs/)).
- **Automation Hub**: registry sync and repository sync.
- **Installers**: download the AAP installer ([playbooks/aap_download.yml](../playbooks/aap_download.yml)), prepare a host ([playbooks/infra/aap-pre.yml](../playbooks/infra/aap-pre.yml)), and use the containerized AAP 2.5 setup files under [installers/](../installers/).

### Red Hat Virtualization / oVirt ([playbooks/rhv/](../playbooks/rhv/), [playbooks/infra/](../playbooks/infra/), [playbooks/ovirt-rename-disks.yml](../playbooks/ovirt-rename-disks.yml))
- Create a host on oVirt (`playbooks/infra/create_host_ovirt.yml`).
- Rename VM disks with [playbooks/ovirt-rename-disks.yml](../playbooks/ovirt-rename-disks.yml) and [roles/pinguozzo.ovirt-rename-disks](../roles/pinguozzo.ovirt-rename-disks/).
- Back up the Hosted Engine (`playbooks/infra/ovirt-he-backup.yml`) and shut an environment down (`playbooks/rhv/ovirt-ansible-shutdown-env.yml`).

### VMware ([playbooks/vmware/](../playbooks/vmware/))
- Golden-image workflow: create VMs, upload VMDKs, add swap, associate to Satellite, add to inventory, fetch blueprints, and post-install.
- Datastore cleanup is [playbooks/infra/vmware_datastore_cleanup.yml](../playbooks/infra/vmware_datastore_cleanup.yml).

### General infrastructure / host post-configuration ([playbooks/infra/](../playbooks/infra/))
- Create users, deploy SSH keys, register systems, install Cockpit, run OpenSCAP (`playbooks/infra/run-oscap.yml`), update Cloudflare DNS records, install GitLab, and run small helpers (ping, date, find-files). RHEL patching and the patch report are `playbooks/patching.yml`.
- Install and initialize HashiCorp Vault on RHEL (`install-hc.yml`, `init-hc.yml`, `init-hc-multi*.yml`) and read secrets with AppRole (`get_hc_secrets.yml`). User playbooks can create Linux accounts from those secrets.

---

## Roles ([roles/](../roles/))

| Role | Purpose |
|------|---------|
| [ldelouw.rh-satellite](../roles/ldelouw.rh-satellite/) | Red Hat Satellite configuration (upstream role) |
| [pinguozzo.create-users](../roles/pinguozzo.create-users/) | Manage Linux users |
| [pinguozzo.ansible-sshkeys](../roles/pinguozzo.ansible-sshkeys/) | Deploy public SSH keys to remote hosts |
| [pinguozzo.gitlab](../roles/pinguozzo.gitlab/) | GitLab setup |
| [pinguozzo.callback](../roles/pinguozzo.callback/) | Callback configuration |
| [pinguozzo.ovirt-rename-disks](../roles/pinguozzo.ovirt-rename-disks/) | Rename oVirt VM disks |
| [pinguozzo.ovirt-he-backup](../roles/pinguozzo.ovirt-he-backup/) | Back up the oVirt Hosted Engine |

Additional Galaxy roles/collections are declared in
[roles/requirements.yml](../roles/requirements.yml) and
[collections/requirements.yml](../collections/requirements.yml).

---

## How to run

### Prerequisites
```bash
# Python deps
pip install -r pip3-requirements.txt          # dnspython, jmespath, petname, requests, pyvmomi, ovirt-engine-sdk-python

# Ansible content
ansible-galaxy collection install -r collections/requirements.yml
ansible-galaxy role install -r roles/requirements.yml
```

### From the CLI (via ansible-navigator + execution environment)
The repo ships an [ansible-navigator.yml](../ansible-navigator.yml) configured to use a Podman execution environment. Typical Satellite config run:
```bash
ansible-navigator run playbooks/satellite/config/all.yml -i inventory/all.yml
```

### From Ansible Automation Platform
Playbooks are designed as job templates. Many read per-environment secrets from `vaults/{{ env }}-vault.yml`. Steps in `playbooks/satellite/config/all.yml` follow inventory `set_*` variables. Survey inputs live in [template_surveys/](../template_surveys/).

---

## Key conventions

- **Environment secrets**: encrypted with Ansible Vault under [vaults/](../vaults/), selected
  by the `env` host variable (e.g. `mgmt`) → `vaults/mgmt-vault.yml`.
- **Feature toggles**: `playbooks/satellite/config/all.yml` runs each imported step when its inventory flag is true (`set_manifest`, `set_locations`, `set_global`, `set_lvm`, `set_repos`, `set_sync`, `set_sync_plan`, `set_environments`, `set_cv`, `set_ak`, `set_3rdparty`). `set_installer` and `set_disconnected` are set in `inventory/all.yml`, and the matching `when` lines in `config/all.yml` are commented out. Those two plays use `tags: never`.
- **Per-OS vars**: `vars/RedHat_6.yml` … `RedHat_10.yml` hold OS-specific settings.
- **Global parameters**: structured `global_parameters` (json/array/scalar) drive Satellite
  global parameters and system-role settings — see the example in the top-level
  [README.md](../README.md).

---

## Directory reference

```
playbooks/
  aap/          AAP config: EE build/upload, teams, users, host lists
  ah/           Automation Hub: registry & repository sync
  ee_utilities/ Execution Environment builders
  satellite/    Satellite lifecycle (config/, content/, provisioning/)
  rhv/          oVirt/RHV operations
  vmware/       VMware golden-image workflow
  infra/        Host post-config: users, Vault, sshkeys, oscap, patching, register, gitlab, VMware datastore cleanup
  image-builder/ Install Image Builder (osbuild-composer, composer-cli, cockpit-composer)
  debug/        Fact/var inspection helpers
roles/          Reusable roles (see table above)
inventory/      Static inventories (all.yml, dc1/, oVirt, AAP, remote-sites)
vars/           Per-OS and per-domain variables
vaults/         Vault-encrypted per-environment secrets
configs/        Controller/Teams/Users config-as-code
template_surveys/ Jinja2 survey & host-list templates
installers/     Containerized AAP 2.5 setup assets
```
