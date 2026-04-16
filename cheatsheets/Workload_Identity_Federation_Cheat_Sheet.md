# Workload Identity Federation Cheat Sheet

## Introduction

CI/CD pipelines often rely heavily on long-lived, static credentials (like cloud provider access keys) to deploy infrastructure or applications. These permanent secrets are a common target for attackers because they can easily leak through poorly configured CI/CD tools, exposed repository settings, hardcoded scripts, logging artifacts, or over-permissive runners. If a static credential is compromised, it increases the blast radius of an attack until manually rotated or revoked.

The preferred approach is to **use workload identity federation and OpenID Connect (OIDC)** to establish a trust relationship directly between your CI/CD platform and your cloud provider.

Instead of storing a static cloud API key in the CI/CD platform, the CI/CD runner requests an OIDC token during the job execution. This token mathematically proves the runner's identity (such as the repository name, branch, and workflow environment). The cloud provider validates this token against a pre-configured trust policy and, if approved, strictly issues temporary, short-lived access credentials.

## Benefits of Workload Identity Federation

- **Eliminates permanent secrets:** Access keys do not need to be stored in the CI/CD tool, removing the risk of long-term credential leakage.
- **Short-lived access:** Dynamically issued credentials expire automatically, minimizing the window of opportunity for an attacker.
- **Granular scoping:** Access can be strictly scoped to specific repositories, branches, environments, or even specific workflow actions.
- **Improved auditability:** Cloud audit logs provide complete visibility into token issuance and role assumption, including precisely which CI/CD workflow requested access.

## Best Practices

- **Restrict trust policies tightly:** Bind the identity strictly to specific repositories, environments, and even specific workflow triggers where supported.
- **Apply the principle of least privilege:** The dynamically issued role should only have the exact permissions necessary for the specific deployment job.
- **Separate trust boundaries:** Use different cloud roles and trust configurations for production vs. non-production deployments.
- **Monitor the trust relationship:** Actively alert on and review logs for anomalous token issuances, cloud role assumptions, or token mapping failures.
- **Disable fallback credentials:** Whenever possible, disable or heavily restrict backup long-lived static credentials for the same service account.
- **Periodically review:** Regularly audit trust relationships, subject claim mappings, and the permissions granted to CI/CD workflows.

## Common Pitfalls

- **Overly broad authorization:** Allowing any branch, fork, or environment from your repository to assume the deployment role.
- **Excessive permissions:** Granting administrative or generic deployment access rather than scoped permissions to a workload running via OIDC.
- **Keeping static keys permanently active:** Failing to delete the legacy long-lived credentials after migrating a workflow to use Workload Identity federation.
- **Failing to validate claims:** Insufficiently verifying the OIDC token claims during the trust policy configuration, rendering the federation insecure.
- **Ignoring audit logs:** Not maintaining visibility over token exchanges and relying purely on CI/CD tooling for audit history.

## Example Pipeline Flow

When a deployment pipeline executes, the CI/CD workflow securely exchanges its job-specific OIDC token with the cloud provider's STS (Security Token Service). The STS evaluates the claims within the token, verifies the digital signature, and returns temporary access keys. The pipeline utilizes these temporary credentials to execute deployment operations. Once the job finishes, or the time-to-live expires, the credentials become instantly useless without requiring any deliberate rotation.

In conclusion, workload identity federation should always be preferred over long-lived static CI/CD secrets for deployment automation whenever supported by both the CI/CD and cloud platforms.
