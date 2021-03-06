# bluebutton-web-deployment
Ansible Configuration and Playbooks for Blue Button API.

## Keeping variables safe
Configuration variables and sensitive values are now stored in this repository
using ansible-vault. hhs_Ansible uses a cascading set of variable files:

- ./vars/envs/common.yaml: For frequently used variables across all environments.
- ./vars/all-var.yml: All variables used across any platform. Environment specific
variables are embedded inside the variable defined in this file. Environment specific
variables are prefixed with "env_".
- ./vars/env/{environment_name}/env.yml: Non-sensitive environment specific variables are
stored in this file. Sensitive environment variables are embedded within env_{variable_name} variables
and are prefixed with "vault_".

Variable files can't embed other variable files as includes. Therefore the
playbook must load the variables files as includes. A typical include section
in a playbook would be:

Where the playbook is found in: ./playbook/{role}/playbook.yml

    ```
    vars_files:
      - "./../../vars/common.yml"
      - "./../../vault/env/{{ env }}/vault.yml"
      - "./../../vars/env/{{ env }}/env.yml"
      - "./../../vars/all_var.yml"

    ```  
{env} is a variable passed at run time to the playbook using
--extra-vars env=dev | test | impl | prod

for example:
In all_var.yml:
aws_secret_key: "{{ env_aws_secret_key }}"

In ./vars/env/{{ env }}/env.yml:
env_aws_secret_key: "{{ vault_env_aws_secret_key }}"

In ./vault/env/{{ env }}/vault.yml:
vault_env_aws_secret_key: "what_ever_the_secret_should_be"


## Installation (Redhat / Centos / Fedora)

To enable ec2 support you must install python-boto:

    sudo yum install -y python-pip
    sudo pip install --upgrade pip
    sudo yum -y install git
    sudo yum -y install ansible
    sudo yum -y install python-boto

NOTE: if FIPS is enabled add an additional parameter to pip command:
-i https://pypi.org/simple/

Install hhs_ansible:

    mkdir /hhs_ansible
    cd /hhs_ansible
    git clone https://github.com/CMSGov/bluebutton-web-deployment.git

Updating the Application Load Balancers requires a newer version of awscli.
Install updated version as follows:

    sudo /bin/bash
    cd /root
    pip install --upgrade --user awscli -i https://pypi.org/simple/

this will install the updated version to

    /root/.local/bin

Then add your hosts to

    /etc/ansible/hosts

Edit the config file:

    /etc/ansible/ansible.cfg

The public keys from ec2-user (id_rsa.pub and id_ecdsa.pub) need to be
generated on the Management Server so that they can be copied to the
remote machines in the base_patch role. As the **ec2-user** on the Management
Server generate two sets of keys with no passphrase:

    ssh-keygen -t rsa
    ssh-keygen -t ecdsa

- Do not enter a passphrase. Hit enter to step pass the prompt
- Do not change the default filenames id_rsa and id_ecdsa.


As a minimum, if you are using AWS you will probably want to change:
    ' #remote_user = root'

to the remote user account used to connect to a server.

## Host Configuration

Hosts fall into the following groups:
 - mgmtservers (The ansible management server)
 - appservers
 - dbservers

When creating a MGMT server instance in ec2 to run ansible add the following
tags to the instance:

 - Managed = "BB-MANAGED-{{ env|upper }}"
 - Environment = {{ env|upper }} eg. DEV | TEST | IMPL | PROD
 - Layer = "MGMT"

These fields are added to each ec2 instance to be managed. These instances
will automatically be grouped in to the relevant group in /etc/anasible/hosts
based upon their Layer setting (MGMT | APP | DATA)

more information about hhs_ansible is here: [./documentation.md]
