# GitLab Runner service

Wodby 2 service manifest for GitLab Runner, based on the official
[GitLab Runner Helm chart](https://docs.gitlab.com/runner/install/kubernetes/).

## Runtime contract

The service runs one GitLab Runner manager with the Kubernetes executor. It creates an isolated pod for each CI job in
the app instance namespace and can execute up to four jobs concurrently. The manager is linked to a GitLab service and
uses that service's primary HTTPS route as its coordinator URL.

Before enabling the service, create a runner in GitLab and copy its authentication token into the required
`runner-token` setting. Authentication tokens begin with `glrt-`; legacy registration tokens are not supported by this
service. Runner tags, protected-branch access, and whether untagged jobs may run are configured on the runner in GitLab.

## Security defaults

Job containers are unprivileged and cannot request privilege escalation. The Runner's Kubernetes service account is
limited to the app instance namespace and receives only the API permissions needed to create, observe, attach to, and
clean up job pods and their supporting Secrets and Services.

Privileged Docker-in-Docker is deliberately not enabled. Build container images with an unprivileged builder such as
BuildKit rootless, Buildah, or Kaniko, or deploy a separately isolated privileged runner after evaluating the cluster
security boundary. A privileged runner can provide cluster-level code execution to anyone authorized to submit jobs.

## Capacity and cache

The default job container limit is 2 CPU and 2 GiB of memory. Helper and service containers have smaller independent
limits. Adjust the service manifest for a different shared runner profile before publication; individual CI jobs cannot
raise these limits.

Distributed CI cache is not configured. Job artifacts still use GitLab's configured object storage, but reusable cache
entries should be treated as unavailable. Add a dedicated S3-compatible cache configuration when repeated dependency
downloads become material.

The manager is intentionally a single replica. GitLab controls job concurrency, while additional manager replicas
would introduce a separate availability and registration lifecycle that should be designed and tested explicitly.
