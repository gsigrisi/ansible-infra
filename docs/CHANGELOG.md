# Changelog

## 2026-11-03

### Fixed

- `playbooks/infra/create_host_ovirt.yml` failed on a second survey run with
  `ovirtsdk4.Error: Fault reason is "Operation Failed". Fault detail is "[MAC Address 56:6f:30:6e:c8:68 is already in use by VM or snapshot: ..."`.
  Survey mode names each guest `<prefix>-rhel<os>-<pet>-NN`, and the pet word changes on every run.
  The MAC was seeded only from `<prefix>-rhel<os>-<index>` (the default prefix `lab`, RHEL 9, index 1 always produced `56:6f:30:6e:c8:68`).
  oVirt rejected the new NIC because the previous VM or one of its snapshots still owned that address.
  The playbook now stores the instance name first, then seeds `community.general.random_mac` from that full name, so each new guest gets its own MAC.
  The VM that already holds the old address can stay.
- `playbooks/aap/update-host-list.yml` failed while updating job templates with
  `Failed to update survey: Default value 0 in survey question 0 expected to be integer.`
  The first question in `template_surveys/create-host-ovirt.j2` is an integer question (`rhel8_instances`), and its default was the string `"0"`.
  The same string default was set for the RHEL 9 and RHEL 10 counts.
  The controller accepts an integer default only when the JSON value is a number.
  Those three defaults are now `0`.
