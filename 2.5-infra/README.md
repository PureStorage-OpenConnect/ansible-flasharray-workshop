# Exercise 2.5 - Configuring common infrastructire settings on a FlashArray

## Table of Contents

- [Objective](#objective)
- [Guide](#guide)
- [Playbook Output](#playbook-outbook)
- [Solution](#solution)
- [Verifying the Solution](#verifying-the-solution)
- [Going Further](#going-further)

# Objective

Demonstrate the use of multiple FlashArray modules to configure common infrastructure settings on a FlashArray.

In this example we will set the the NTP servers using the [purefa_ntp module](https://docs.ansible.com/ansible/latest/collections/purestorage/flasharray/purefa_ntp_module.html), the DNS servers using the [purefa_dns module](https://docs.ansible.com/ansible/latest/collections/purestorage/flasharray/purefa_dns_module.html) and the SYSLOG server using the [purefa_syslog module](https://docs.ansible.com/ansible/latest/collections/purestorage/flasharray/purefa_syslog_module.html).

# Guide

## Step 1:

Using the text editor, create a new file called `purefa-infra.yml`.

## Step 2:

Enter the following play definition into `purefa-infra.yml`:

``` yaml
---
- name: INFRASTRUCTURE SETTINGS
  hosts: localhost
  connection: local
  gather_facts: true
  vars:
    url: 10.34.56.233
    api: 89a9356f-c203-d263-8a89-c229486a13ba
```

- The `---` at the top of the file indicates that this is a YAML file.
- The `hosts: localhost`, indicates the play is run on the current host.
- `connection: local` tells the Playbook to run locally (rather than SSHing to itself)
- `gather_facts: true` enables facts gathering.  
- The `vars:` parameter is a group of parameters to be used in the playbook.
- `url: 10.34.56.233` is the management IP address of your source FlashArray - change this reflect your local environment.
- `api: 89a9356f-c203-d263-8a89-c229486a13ba` is the API token for a user on the source FlashArray - change this reflect your local environment.

## Step 3:

Next, add the following `tasks` to the playbook. These tasks will use three different modules to configure the DNS, NTP and SYSLOG settings on the FlashArray.

``` yaml
  tasks:
    - name: CONFIGURE NTP
      purestorage.flasharray.purefa_ntp:
        ntp_servers:
          - 0.pool.ntp.org
          - 1.pool.ntp.org
        fa_url: "{{ url }}"
        api_token: "{{ api }}"

    - name: CONFIGURE DNS
      purestorage.flasharray.purefa_dns:
        nameservers:
          - 8.8.8.8
          - 8.8.4.4
        fa_url: "{{ url }}"
        api_token: "{{ api }}"

    - name: CONFIGURE SYSLOG
      purestorage.flasharray.purefa_syslog:
        name: acme_syslog
        address: syslog.acme.com
        port: 514
        protocol: tcp
        fa_url: "{{ url }}"
        api_token: "{{ api }}"
```

- `name: CONFIGURE NTP | DNS | SYSLOG` is a user defined description that will display in the terminal output.
- `purefa_*:` tells the task which module to use.
- The other parameters tells the modules what values to use for the appropriate configuration values. More details can be found in the online Ansible Collection documentation.
- The `fa_url: "{{url}}"` parameter tells the module to connect to the FlashArray Management IP address, which is stored as a variable `url` defined in the `vars` section of the playbook. This makes this array the source array in the replication pair.
- The `api_token: "{{api}}"` parameter tells the module to connect to the FlashArray using this API token, which is stored as a variable `api` defined in the `vars` section of the playbook.

Save the file and exit out of the editor.

## Step 4:

Run the playbook - Execute the following:

```
[student1@ansible ~]$ ansible-playbook purefa-infra.yml
```

# Playbook Output

The output will look as follows.

```yaml
[student1@ansible ~]$ ansible-playbook purefa-infra.yml

PLAY [INFRASTRUCTURE SETTINGS] ******************************************************************************************

TASK [Gathering Facts] **************************************************************************************************
ok: [localhost]

TASK [CONFIGURE NTP] ****************************************************************************************************
changed: [localhost]

TASK [CONFIGURE DNS] ****************************************************************************************************
changed: [localhost]

TASK [CONFIGURE SYSLOG] *************************************************************************************************
changed: [localhost]

PLAY RECAP **************************************************************************************************************
localhost                  : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

# Solution

The finished Ansible Playbook is provided here: [purefa-infra.yml](https://github.com/PureStorage-OpenConnect/ansible-workshop/blob/master/2.5-infra/purefa-infra.yml).

# Verifying the Solution

Login to the source Pure Storage FlashArray with your web browser using the management IP address you set in your YAML file.

Navigate to the Settings -> System window to see the array configuration. There are sub-windows specifically for Array Time, where the NTP servers are listed, and Syslog Servers.

To see the DNS domain and nameserver settings navigate to Settings -> Network page and look at the DNS Settings sub-window.

# Going Further

Configuring the infrastructure settings is only the tip of the iceberg for FlashArray. Using these and other infrastructure related modules for the FlashArray you can build very complex playbooks to configure and maintain your critical configurations.

For more details on this control, also referred to as Drift control, you can check out this [blog post](http://theansibleguy.com/manage-drift-with-ansible/) that describes in greater detail the methodology and utilises the concept of Ansible 'roles'.
