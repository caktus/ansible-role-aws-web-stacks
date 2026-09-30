caktus.aws-web-stacks
======================


v0.4.0
------

* Add support for Ansible 13 (ansible-core 2.19+), which requires conditionals to
  be booleans: template bucket/upload/URL tasks use explicit checks, and
  `template`, `template_body` and `template_url` are omitted when empty (#4)
* Make the generate-inventory conditional return a boolean (#5)
* Make the remove-inventory conditional return a boolean (#6)
* Only print `stack_outputs` when defined, since changeset runs have none (#6)


v0.3.0
------

* Set `public_access` via amazon.aws.s3_bucket module


v0.2.0
------

* Upgrade bucket role to reflect Ansible changes


v0.1.0
------

* Initial release
