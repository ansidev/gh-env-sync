# GitHub Actions Secrets & Variables Synchronization Tool

## Overview

A Taskfile.dev-based tool to synchronize GitHub Actions secrets and variables from one repository to another quickly.

### Features

- List secrets/variables from source repository.
- Synchronize variables (with values).
- Export variables (with values) from a GitHub repository to a template file.
- Synchronize secrets (with user-provided values via a secret file).
- Synchronize variables (with user-provided values via a template file).

### Supported levels

- [x] Repository.

## Files Structure

```plaintext
.
├── Taskfile.yml  # Main task definitions
├── .env.example  # Secrets template example
```

## Manual

### Prerequisites

- `gh` CLI installed and authenticated
- `task` CLI installed (Taskfile.dev)

On macOS you can run the command below to get both tools

```bash
brew install gh go-task
```

### Tasks Specification

| Task            | Description                                                                       |
| --------------- | --------------------------------------------------------------------------------- |
| `check-gh`      | Verify gh CLI installed/authenticated                                             |
| `var:ls`        | List variables from source repo                                                   |
| `var:export`    | Export variables from source repo to template file (new feature)                  |
| `var:sync`      | Synchronize variables from source to target                                       |
| `secret:ls`     | List secret names from source repo                                                |
| `secret:sync`   | Synchronize secret from source to target using entries from the given secret file |
| `secret:export` | Export secret name from a GitHub repository to a secret file template             |
| `sync`          | Main entry point - synchronize secrets & variables                                |

### Usage Flow

#### Check gh CLI status

```bash
task check-gh
```

#### Interactive Mode (Prompts for secrets)

```bash
task sync source=<owner>/<repo> target=<other>/<repo>
```

#### Non-Interactive Mode (using secrets from a secret file)

```bash
task sync source=<owner>/<repo> target=<other>/<repo> secret_file=<secret_file>
```

#### Non-Interactive Mode (only synchronize variables and export secret names to a secret file only)

```bash
task sync source=<owner>/<repo> target=<other>/<repo> secret_file=<secret_file> mode=manual
```

#### Only Synchronize Variables (No secrets)

##### Synchronize all variables from source repository to target repository

```bash
task var:sync source=<owner>/<repo> target=<other>/<repo>
```

##### Synchronize all variables from source variable file to target repository

```bash
task var:sync source=<source_variable_file> source_type=file target=<other>/<repo>
```

##### Export variables from source repository to template file

```bash
task var:export source=<owner>/<repo> output_file=.env.vars
```

#### Only Synchronize Secrets (No variables)

```bash
task secret:sync source=<owner>/<repo> target=<other>/<repo>
```

#### Only Export Secrets (No variables)

```bash
task secret:export source=<owner>/<repo> output_file=<secret_file>
```

#### List variables from source repository

```bash
task var:ls source=<owner>/<repo>
```

#### List secrets from source repository

```bash
task secret:ls source=<owner>/<repo>
```

### How does it works?

#### Variables Synchronization

- Use `gh variable list --json name,value` to get all variables
- Use `gh variable set <name> --body <value> --repo <target>` to set each

#### Variable Export Workflow

We cannot retrieve variable values via gh CLI due to security reasons.

1. Export variables to template file, user manually fills values then:
   - Use `gh variable list --repo <repo> --json name,value --jq '.[].name'` to get all variable names.
2. Synchronize variables to the target repository using the template file:
   - Read variable entries from the template file and use `gh variable set <key> --repo <target> --body <value>` to set each.

## Contact

Le Minh Tri (a.k.a [@ansidev](https://ansidet.xyz/about)).

## License

[MIT](./LICENSE)

Copyright © 2026-present Le Minh Tri.