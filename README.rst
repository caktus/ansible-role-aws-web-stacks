caktus.aws-web-stacks
======================

Ansible role to automate AWS CloudFormation stack provisioning with
`aws-web-stacks <https://github.com/caktus/aws-web-stacks>`_.


License
~~~~~~~~~~~~~~~~~~~~~~

This Ansible role is released under the BSD License.  See the `LICENSE
<https://github.com/caktus/ansible-role-aws-web-stacks/blob/master/LICENSE>`_
file for more details.

Development sponsored by `Caktus Consulting Group, LLC
<http://www.caktusgroup.com/services>`_.


Requirements
~~~~~~~~~~~~~~~~~~~~~~

* awscli (required to restrict bucket policy)


Installation
------------

1. Add to your ``requirements.yml``:

.. code-block:: yaml

    ---
    # file: deployment/requirements.yml

    - src: https://github.com/caktus/ansible-role-aws-web-stacks
      name: caktus.aws-web-stacks


2. Copy aws-web-stacks YAML template (develop branch) to
``playbooks/files/ec2-nat.yml``.


Example
~~~~~~~~~~~~~~~~~~~~~~


Playbooks
----------------------

Playbooks to provision and deprovision environments:

.. code-block:: yaml

    ---
    # file: deployment/playbooks/provision.yml

    - name: create/update a cloudformation stack
      hosts: aws
      gather_facts: false
      roles:
        - role: caktus.aws-web-stacks

    ---
    # file: deployment/playbooks/deprovision.yml

    - name: create/update a cloudformation stack
      hosts: aws
      gather_facts: false
      roles:
        - role: caktus.aws-web-stacks
          vars:
            cloudformation_stack_state: absent

Run playbook example::

  ansible-playbook -vv -i deployment/environments/dev deployment/playbooks/provision.yml


After provisioning, you can graph your inventory::

    ansible-inventory -vv -i deployment/environments/dev --graph


Post-provision playbook to ping bastion host:

.. code-block:: yaml

    ---
    # file: deployment/playbooks/ping.yml

    - name: test ping
      hosts: role_bastion
      tasks:
        - ping:

Run playbook example::

    ansible-playbook -vv --user ubuntu -i deployment/environments/dev deployment/playbooks/ping.yml


Inventory
----------------------

Simple inventory for local connections:

.. code-block:: ini

    # envs/dev/aws

    [aws]
    aws.amazon.com ansible_connection=local ansible_python_interpreter='/usr/bin/env python3'


Vars
----------------------

The following example use `Ansible's variable order of precedence`_ to allow
overriding specific variables per-environment.

.. _Ansible's variable order of precedence: https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#variable-precedence-where-should-i-put-a-variable

Project (shared across environments) variables:

.. code-block:: yaml

    ---
    # playbooks/groups_vars/all/caktus.aws-web-stacks.yml

    # The following 4 variables could be in another variables file if
    # desired, but are included here for simplicity.
    project_name: myproject
    stack_name: '{{ project_name }}-{{ env_name }}'
    aws_profile: myproject
    aws_region: us-east-1

    # ----------------------------------------------------------------------------
    # caktus.aws-web-stacks: Provision a stack with AWS CloudFormation.
    # ----------------------------------------------------------------------------

    default_cloudformation_stack:

      profile: "{{ aws_profile | mandatory }}"
      region: "{{ aws_region | mandatory }}"
      stack_name: "{{ stack_name | mandatory }}"
      template_bucket: "aws-web-stacks-{{ project_name }}"

      template_parameters:
        AMI: ami-026c8acd92718196b  # Ubuntu 18.04
        AssetsBucketAccessControl: "Private"
        AssetsCloudFrontCertArn: ""
        AssetsUseCloudFront: "false"
        UseAES256Encryption: "true"
        AdministratorIPAddress: "0.0.0.0/0"
        BastionType: "SSH"
        BastionAMI: ami-026c8acd92718196b  # Ubuntu 18.04
        BastionKeyName: "{{ ssh_keypair_name }}"
        CertificateValidationMethod: "(none)"
        ContainerInstanceType: t3.small
        CustomAppCertificateArn: ""
        DatabaseClass: db.t2.small
        DatabaseEngineVersion: "11"
        DatabaseUser: "{{ project_name }}_{{ env_name }}"
        DatabasePassword: "{{ SECRET_DATABASE_PASSWORD }}"
        DatabaseName: "{{ project_name }}_{{ env_name }}"
        DatabaseParameterGroupFamily: postgres11
        DesiredScale: 2
        DomainNameAlternates: ""
        KeyName: "{{ ssh_keypair_name }}"
        MaxScale: 2
        PrimaryAZ: "{{ aws_region }}a"
        SecondaryAZ: "{{ aws_region }}b"
        SecretKey: "{{ SECRET_KEY }}"
      tags:
        Environment: "{{ env_name }}"

    # Combine default stack parameters with customizations
    cloudformation_stack: >-
      {{ default_cloudformation_stack |
      combine(cloudformation_stack_overrides, recursive=True)
      }}


Per-environment variables:

.. code-block:: yaml

    ---
    # envs/dev/group_vars/all/vars.yml
    env_name: dev
    domain: dev.myproject.com

    cloudformation_stack_overrides:

      template_parameters:
        ContainerInstanceType: t3.micro
        DomainName: '{{ domain }}'
