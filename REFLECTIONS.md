# Reflections - costctl CLI Challenge

## 1. `clean --apply` Blast Radius

If `clean --tag Environment=dev --apply` is accidentally run in a shared account, the blast radius is massive. To limit damage, we should have the following safeguards in place:

- **Strict IAM permissions**: Restrict deletion/termination capabilities via fine-grained IAM policies (e.g., denying terminate actions on critical/production instances or resources tagged with `System=Critical`).
- **Resource termination protection**: Enable termination protection on critical EC2 instances and database instances, preventing deletion without explicitly disabling the setting first.
- **Tag-based boundary checks**: Prevent bulk cleanups on general keys like `Environment=dev` unless a more specific key (e.g., `GroupOwner=G1`) is also supplied.
- **Interactive multi-stage prompts**: When `--apply` is specified, require double confirmation including typing the group ID or tag value to verify intent before executing destructive calls.
- **Dry-run log generation**: Keep a local and centralized audit trail (e.g., in CloudTrail/S3) of what was scanned and planned for deletion before running the active execution.

## 2. AI Assistance Reflection

- **Fraction of Code**: Approximately 80% of the core command boilerplate was assisted by AI tools to align with standard boto3 documentation and paginated formats.
- **Active Modifications**:
  - We modified S3 bucket deletion logic to strictly catch bucket-not-empty conditions beforehand rather than blindly calling delete.
  - We added merged tag logic for S3 tagging, ensuring that get_bucket_tagging is executed first to prevent overwriting other pre-existing bucket tags.
  - We handled ClientError in all CLI command flows to format human-readable error messages on stdout instead of standard Python tracebacks.
