# ironicbadger/ansible-role-proxmox-nag-removal

This role removes the obnoxious Proxmox 'please subscribe' dialog box from a non subscribed server. It will also by default ensure that the enterprise repos are disabled.

Tested and working with Proxmox VE 9.2 (Debian 13 / trixie). Behaviour on Proxmox VE 6 through 8 is unchanged.

## Variables

| Variable | Default | Effect |
| --- | --- | --- |
| `remove_nag` | `True` | Patches `proxmoxlib.js` so the subscription dialog never fires, then restarts `pveproxy`. |
| `remove_enterprise_repo` | `True` | Deletes `pve-enterprise.list` and `pve-enterprise.sources`. |
| `remove_ceph_enterprise_repo` | `False` | Deletes `ceph.list` / `ceph.sources` when they point at the enterprise CDN. |
| `add_no_subscription_repo` | `True` | Configures the `pve-no-subscription` repo for the host's Debian release. |

## Proxmox VE 9

Proxmox VE 9 moved its APT sources to the deb822 format. On Debian 13 and newer the role writes `/etc/apt/sources.list.d/pve-no-subscription.sources` and removes any leftover `pve-no-subscription.list`. On Debian 12 and older it keeps writing the one-line `pve-no-subscription.list`.

The enterprise repo removal deletes both `pve-enterprise.list` and `pve-enterprise.sources`, so it covers either layout.

## Ceph

`remove_ceph_enterprise_repo` is off by default because the Ceph repo is not part of a stock install. `pveceph install` writes it, as a single stanza in `ceph.sources` on Proxmox VE 9 or `ceph.list` on older releases, and the component you picked at install time decides whether it is the enterprise repo. An unsubscribed host left on the enterprise component gets a 401 from `enterprise.proxmox.com` on every `apt update`.

Enabling the variable deletes the file only when it points at `enterprise.proxmox.com`, so a `no-subscription` or `test` Ceph repo is left in place. Deleting it takes Ceph packages away with it; run `pveceph install --repository no-subscription` afterwards if you still want them.
