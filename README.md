# Role ansible_galaxy_role.package

The role ansible_galaxy_role.package will manage packages.

## Requirements

Supported package managers:
* dnf
* yum
* apt

---
## Role Parameters

| Parameter | Choices | Description |
|---|---|---|
| `package_list` | [] <-- | **Required**<br>List of packages |
| `package_state` | `present` <--<br>`absent` | Install (`present`) packages<br>or remove (`absent`) packages|
| `package_update_cache` | `yes` <--<br>`no` | Update package manager cache<br>i.e. run: `apt update`|


---
## `package_list` details

| Variable     | Required | Description                                               |
|--------------|---|-----------------------------------------------------------|
| `name`       | yes | Package name                                              |
| `state`      | no | If not defined then it defaults to role's `package_state` |
| `dnf_name`   | no | Dnf package name if different than `name`                 |
| `yum_name`   | no | Yum package name if different than `name`                 |
| `apt_name`   | no | Apt package name if different than `name`                 |
| `dnf_ignore` | no | Ignore package for dnf
| `yum_ignore` | no | Ignore package for yum                                    |
| `apt_ignore` | no | Ignore package for apt                                    |

The following `package_list` come predefined with the role:
* `packages_homeco`<br>
List of package the HomeCo team installs on every system.

###### Example `package_list`

```yaml
packages_homeco:
  - name: net-tools
  - name: screen
  - name: mlocate
    apt_ignore: yes
  - name: mail
    dnf_name: mailx
    yum_name: mailx
    apt_name: mailutils
```
The `ansible_galaxy_role.package` role will install `net-tools` and `screen` on yum (CentOS) and apt (Ubuntu) systems.<br>
Package `mlocate` will be installed only on yum (CentOS) systems.<br>
For package definition `mail` the role will install package `mailx` on yum systems and `mailutils` on apt systems<br>

```yaml
packages_remove:
  - name: screen
    state: absent
  - name: tmux
    state: absent
    yum_ignore: yes
```
Package `screen` will be removed from any system.<br>
Package `tmux` will be removed from apt (Ubuntu) systems.

***NOTE:*** The variable `package_list` can have a mix of packages defined in different states (present, or absent),
but should be used in rare cases, like one time runs and ad-hoc runs. To make the code easier to understand,
the `package_list` variable should not have the state defined.<br>
The state should be defined as role variable and it will apply to all packages defined in the `package_list`:
```yaml
- { role: ansible_galaxy_role.package, package_list: '{{packages_old}}', package_state: absent }
```

---
## Example Playbook

**Note:** By default we ignores tasks tagged with `with_packages`.<br>
For regular playbook run, that caries a few configuration changes, there is no need to wait for cache to be updated and packages status to be confirmed.<br>
To override this default setting and install the packages run with: `--with_packages`


* ### Install packages as defined in the list
```yaml
- name: PlaybookName
  hosts: all
  roles:
    - { role: ansible_galaxy_role.package, package_list: '{{ packages_homeco }}', tags: with_packages }
```
The above line is longer than 80 columns causing some editors to wrap the line, which will make the code hard to read, and it's not good practice.<br>
Instead do this:
```yaml
- name: PlaybookName
  hosts: all
  roles:
    - role: ansible_galaxy_role.package
      tags: with_packages
      vars:
        package_list: '{{ packages_homeco }}'
```

* ### Remove packages
```yaml
- role: ansible_galaxy_role.package
  tags: with_packages
  vars:
    package_state: absent
    package_list: '{{ packages_old }}'
```
**Note:** Variable `package_state` is defined for the whole list. There is no need to specify the state of each package
in the list. The variable `packages_old` can be just a list of names:<br>
```yaml
packages_old:
  - name: net-tools
  - name: mlocate
  - name: tmux
```

* ### `package_list` definition with the role
```yaml
- role: ansible_galaxy_role.package
  tags: with_packages
  vars:
    package_list:
      - name: screen
        state: absent
      - name: tmux
        apt_ignore: yes
```

---




