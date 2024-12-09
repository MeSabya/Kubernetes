## You are an administrator of a Kubernetes cluster running a couple of existing Pods. It's your job to inspect the containers defined by the Pods for immutability. Delete all Pods that do not follow typical immutability best practices.

A container can be considered mutable if it exhibits characteristics that allow it to change during its lifecycle.
***Below characteristics will make a container mutable***

### Writable Filesystem:

- Filesystem is not read-only (readOnlyRootFilesystem: false or omitted)
- Applications write to internal container paths instead of external storage.

### Ephemeral Runtime Configuration:
Runtime configurations are hardcoded or modified within the container instead of being injected via environment variables, ConfigMaps, or Secrets.

### Writable Volume Mounts:
Volume mounts (e.g., /tmp, /var) are writable instead of being explicitly read-only.

### hostNetwork: true:
Container shares the host’s network namespace, exposing it to potential security risks and host-level mutability.

### securityContext.runAsUser: 0 (Root User):
The container runs as the root user, increasing the risk of privilege escalation and unintended modifications.

### securityContext.privileged: true:
Privileged containers have full access to the host, allowing modifications to the host system.

### Absence of readOnlyRootFilesystem:
The root filesystem is writable, enabling modifications at runtime.

### Capabilities Added:
Elevated capabilities (e.g., SYS_ADMIN) are added, granting the container more privileges than necessary.

### Host Path Mounts:
The container mounts host paths directly, allowing modification of host files or access to sensitive directories.
