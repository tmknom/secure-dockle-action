# secure-dockle-action

Run [Dockle][dockle] in an isolated Docker container to securely lint container images.

<!-- actdocs start -->

## Description

This action securely runs Dockle in a Docker container to lint container images.
It helps enforce [Dockerfile Best Practices][best_practices] and checks compliance with [CIS Benchmarks][cis].

This action reduces security risks associated with software supply chain attacks, such as compromised third-party tools or tampered container images.
To achieve this, it enforces strict container isolation, disables network connections, and drops unnecessary privileges.

## Usage

### Default

```yaml
  steps:
    - name: Secure Dockle
      uses: tmknom/secure-dockle-action@v0
```

### Custom

```yaml
  steps:
    - name: Secure Dockle
      uses: tmknom/secure-dockle-action@v0
      with:
        dockerfile-path: ./subdir/Dockerfile.dev
        ignore: CIS-DI-0005 CIS-DI-0006
        exit-level: info
```

## Inputs

| Name | Description | Default | Required |
| :--- | :---------- | :------ | :------: |
| dockerfile-path | The path to the Dockerfile for security linting. | `./Dockerfile` | no |
| exit-level | Severity level (`fatal`, `warn`, or `info`) at which violations cause the action to fail. | `warn` | no |
| ignore | Space-separated list of Dockle checkpoint codes to ignore. | n/a | no |

## Outputs

N/A

<!-- actdocs end -->

## Permissions

N/A

## FAQ

### What is Dockle?

[Dockle][dockle] is a linter for container images.
It detects errors, security issues, and best practice violations, helping you maintain safer and more reliable container images.

### Why should I use this action instead of directly using Dockle?

Running third-party tools directly in your environment may expose your repository and credentials to compromised or malicious code.
This action significantly reduces such risks by strictly isolating the environment:

- Network access is completely disabled (`--network none`)
- All unnecessary Linux capabilities are dropped (`--cap-drop all`)
- Privilege escalation is explicitly disabled (`--security-opt no-new-privileges`)
- The action runs as a non-root, restricted user  (`--user 1111:1111`)
- The filesystem is strictly read-only (`--read-only`)
- The repository directory is mounted as read-only (`--volume "${volume}:${volume}:ro"`)

### What specific security risks does this action protect against?

This action specifically prevents threats related to software supply chain security (attacks targeting third-party software or tools used in workflows), such as:

- Unauthorized outbound connections from runners, significantly reducing the risk of data leakage
- Malicious updates or compromised tools exploiting elevated privileges or unrestricted network access

### How does this action ensure the Docker image used for Dockle is secure?

This action explicitly specifies the Docker image using its digest (SHA256).
Using a digest ensures that exactly the intended and verified image is used every time, eliminating the risk of malicious updates or image tampering.

### Are network connections permitted inside the Docker container used by this action?

No. Network connections are completely disabled within the container.
Even if the tool were compromised or contained malicious code, disabling network access effectively prevents communication with external attackers, significantly reducing the risk of data leaks.

### Does this action run with elevated (root) privileges?

No. The action runs as a non-root, restricted user without privilege escalation.


### Should I pin this action using a commit SHA?

**Yes, strongly recommended.**

To further protect your workflows from unintended or malicious modifications, it's a best practice to pin the action to a specific commit SHA (commit hash).
Doing so ensures the immutability of both the action’s code and any resources it references, such as Docker images, further reducing the risk of software supply chain attacks.

**Recommended (more secure):**

```yaml
- uses: tmknom/secure-dockle-action@de64904abbaee35da28dd45cfa7a31e084347ed3 # v0.2.0
```

**Not recommended:**

```yaml
- uses: tmknom/secure-dockle-action@v0
```

### What value should I specify for the `dockerfile-path` input?

Specify the path to your Dockerfile, relative to the repository root.

```yaml
- uses: tmknom/secure-dockle-action@v0
  with:
    dockerfile-path: ./path/to/Dockerfile
```

**Examples:**

- `./Dockerfile` (Default)
- `./docker/Dockerfile` (Subdirectory)
- `./Dockerfile.dev` (Alternative name)

### Can I customize the Dockle parameters?

Yes. You can customize Dockle parameters through the inputs described in the [Inputs](#inputs) section.
See [Custom Usage](#usage) for an example.

This action intentionally limits arbitrary customization to ensure secure, isolated, and predictable execution.
If you require further customization, consider creating your own fork of this action.

### Why does this action build the Docker image every time?

This action builds the image from scratch each time to ensure a clean, reproducible, and secure environment for linting.
While this slightly increases build time, it prevents reuse of potentially compromised cached layers.

### Does using this action significantly impact my CI/CD performance?

No. The impact is minimal, primarily due to the overhead of pulling and executing a small Docker container.
Typically, this added overhead is negligible, and the security improvements provided by the action justify its use.

### Can I run this action without Docker?

No. This action strictly requires Docker to be installed on your GitHub Actions runner.
Without Docker, the action will fail to execute, as it relies on Docker’s isolation mechanisms to run securely.

## Related projects

N/A

## Release notes

See [GitHub Releases][releases].

[dockle]: https://github.com/goodwithtech/dockle
[best_practices]: https://docs.docker.com/develop/develop-images/dockerfile_best-practices/
[cis]: https://www.cisecurity.org/cis-benchmarks/
[releases]: https://github.com/tmknom/secure-dockle-action/releases
