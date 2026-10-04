# TOFIX

Findings from a code scan on 2026-10-04.

## High

- `show_all_instances.yaml:7` and `show_all_my_instances.yaml:7` - `ec2_facts` no longer exists; `ansible-playbook --syntax-check` fails with "couldn't resolve module/action 'ec2_facts'" (verified). Use `amazon.aws.ec2_metadata_facts` for the per-host facts demo, and `amazon.aws.ec2_instance_info` (which is what actually accepts `filters:`, `show_all_my_instances.yaml:8-10`) run against `localhost` for the "list my instances" demo.
- `get_one_spot.yaml:23-34` - uses the removed `ec2` module (not present in the installed amazon.aws collection, verified with `ansible-doc -l`) through the deprecated mapping form of `local_action` (syntax-check prints a deprecation: removed in ansible-core 2.23). Rewrite with `amazon.aws.ec2_instance` (spot options via `instance_market_options`) and `delegate_to: localhost`/`connection: local` instead of `local_action: {module: ...}`.

## Medium

- `rsconstruct.toml:21-22` - only yamllint checks the playbooks, which is why the broken modules above went unnoticed. Add a `script` processor running `ansible-playbook --syntax-check` (or `ansible-lint`, declared in the `dev` group of `pyproject.toml`) over the three playbooks.
- `get_one_spot.yaml:12-18` and `get_one_spot.yaml:30` - hardcoded account-specific values from a former employer's AWS account (subnet `subnet-1b54456d`, AMI `ami-ccf297fc`, owner `mark@twiggle.com`, previous-generation `t1.micro`); also `show_all_my_instances.yaml:10`. Move them to variables (`vars:`/`vars_prompt`/an example vars file) so the demo can run in any account.

## Low

- `get_one_spot.retry` - Ansible retry file (run artifact) committed to git; delete it.
- `README.md:1` - README is only a title; add what each playbook demonstrates and how to run it (required collection `amazon.aws`, AWS credentials).
