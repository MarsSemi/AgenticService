# AgenticService GitHub Release

This directory contains the release assets and deployment materials for `AgenticService`.

Language versions:

- [ENG](./README.md)
- [CHINESE](./README_cht.md)
- [JAPANESE](./README_jp.md)

`AgenticService` is a local-first Go service focused on agentic orchestration. It combines:

- structured tool execution
- multi-step task workflows
- local shell and file operations
- task-backed long-running jobs
- local RAG retrieval over knowledge folders
- skill-driven workflow activation
- a built-in web console for sessions, planning, debugging, skills, and RAG

## Release Scope

This release currently targets:

- Linux x86_64

The following platform is intentionally **not** part of the public release notes:

- `linux_arm32_sf`

## Release Assets

Recommended GitHub Release assets:

- `AgenticService_linux_x86_minimal.tar.gz`
- `deploy_files/`
- release notes

## Asset Layout

### `deploy_files/`

This is the release-ready deployment folder used to produce the minimal package.

Typical contents:

```text
deploy_files/
├── AgenticService
├── agent.sample.properties
├── run.sh
├── run_bg.sh
├── stop.sh
├── history/
├── provider/
├── skill/
└── website/
```

### Platform binaries

This `dist/` directory may also contain platform build outputs such as:

- `linux_x86_64/`
- `linux_arm64/`
- `macos_arm64/`

They are build artifacts, but `deploy_files/` is the deployment entry point for GitHub release publishing.

## Main Features

This release bundle supports the following service capabilities:

- chat-based task execution
- plan and tool trace visualization
- shell, file, search, and task tools
- skill-based execution constraints and workflow injection
- local RAG retrieval with scoped file search
- built-in web UI for operational use

## Configuration Rules

This release intentionally does **not** include:

- `agent.properties`
- host-specific credentials
- host-specific logs
- private runtime artifacts that should not be shared

Use the included template:

- `agent.sample.properties`

Recommended rule:

- if `agent.properties` already exists on the target machine, keep it
- otherwise copy `agent.sample.properties` to `agent.properties`

Example:

```bash
cp ./agent.sample.properties ./agent.properties
```

## SSL / HTTPS Configuration

`AgenticService` can enable HTTPS when the SSL settings in `agent.properties` are configured.

Supported patterns:

- PEM certificate + private key
- PKCS#12 / PFX bundle

### PEM mode

Use:

- `ssl_key` for the certificate file path
- `ssl_key_file` for the private key file path
- `ssl_key_password` can be left empty

Example:

```json
"https_port" : 443,
"ssl_key" : "./certs/server.crt",
"ssl_key_file" : "./certs/server.key",
"ssl_key_password" : ""
```

### P12 / PFX mode

Use:

- `ssl_key` for the `.p12` or `.pfx` bundle path
- `ssl_key_password` for the bundle password
- `ssl_key_file` is not required in this mode

Example:

```json
"https_port" : 443,
"ssl_key" : "./certs/server.p12",
"ssl_key_file" : "",
"ssl_key_password" : "your-password"
```

### HTTPS validation

When HTTPS is enabled, test it with:

```bash
curl -k https://127.0.0.1:PORT/talk/hello
curl -k https://127.0.0.1:PORT/api/hello
```

Replace `PORT` with the configured `https_port`.

## Quick Deploy

### Option 1: Deploy from `deploy_files/`

```bash
chmod +x ./AgenticService ./run.sh ./run_bg.sh ./stop.sh
cp ./agent.sample.properties ./agent.properties
./run_bg.sh
```

### Option 2: Deploy from tarball

Package:

```bash
tar -czf AgenticService_linux_x86_minimal.tar.gz -C deploy_files .
```

Deploy:

```bash
mkdir -p /opt/AgenticService
tar -xzf AgenticService_linux_x86_minimal.tar.gz -C /opt/AgenticService
cd /opt/AgenticService
cp ./agent.sample.properties ./agent.properties
chmod +x ./AgenticService ./run.sh ./run_bg.sh ./stop.sh
./run_bg.sh
```

### Option 3: Use the repository deployment helper

The repository includes a remote deployment helper:

- [deploy_linux_x86_remote.sh](/Users/vader/Codes/Go/AgenticService2/deploy/deploy_linux_x86_remote.sh)

This script uses the same deployment layout as `deploy_files/`.

## Post-Deploy Validation

After startup, validate with:

```bash
curl http://127.0.0.1:PORT/talk/hello
curl http://127.0.0.1:PORT/api/hello
```

Replace `PORT` with the actual service port configured in `agent.properties`.

Expected outcomes:

- the web UI opens normally
- `/talk/hello` responds
- `/api/hello` responds
- sessions can be created
- tool-based workflows can run

## Publishing Notes

Before publishing to GitHub:

- review any bundled `skill/` contents
- avoid shipping private login or runtime credential files
- keep `agent.properties` out of release assets
- keep host-generated runtime state out of the release package when possible

## Suggested Release Notes Structure

Recommended GitHub Release note sections:

1. What is included in this release
2. Supported platform
3. Deployment steps
4. Configuration notes
5. Known limitations

## Known Limitations

- This release document currently focuses on Linux x86_64 deployment.
- Some runtime content in `skill/` may need review before public publishing.
- Host-local runtime files should not be treated as portable release assets.
