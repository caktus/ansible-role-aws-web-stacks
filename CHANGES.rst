caktus.aws-web-stacks
======================


Unreleased
----------

* Make the remove-inventory conditional return a boolean (required by ansible-core 2.19+)
* Only print `stack_outputs` when defined (changeset runs have none)


v0.3.0
------

* Set `public_access` via amazon.aws.s3_bucket module


v0.2.0
------

* Upgrade bucket role to reflect Ansible changes


v0.1.0
------

* Initial release
