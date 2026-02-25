# GitHub Actions Secrets & Variables Synchronization Tool - Design Documentation

## Overview
This tool is a Taskfile-based solution for synchronizing GitHub Actions secrets and variables between repositories. It provides functionality to list, synchronize, and export these configuration elements using the GitHub CLI (`gh`) and Taskfile tooling.

## Architecture
The tool is built around:
- **Taskfile.yml**: Contains all task definitions and workflows
- **GitHub CLI (gh)**: Used for interacting with GitHub repositories to list/get/set variables and secrets
- **Shell scripting**: Implements the core logic for synchronization operations

## Core Components

### 1. Variables Synchronization
- **List variables**: Uses `gh variable list --repo <repo>` to retrieve all variables from a source repository
- **Synchronize variables**: Uses `gh variable get` to fetch each variable's value and `gh variable set` to apply them to the target repository

### 2. Secrets Synchronization
- **List secrets**: Uses `gh secret list --repo <repo>` to retrieve all secret names from a source repository
- **Export secrets**: Generates template files with secret names but no values (due to security restrictions)
- **Synchronize secrets**: Reads from secret files and uses `gh secret set` to apply secrets to target repositories

### 3. Task Operations
The tool provides several tasks:
- `check-gh`: Validates that the GitHub CLI is installed and authenticated
- `var:ls`: Lists variables from a repository
- `var:sync`: Synchronizes variables between repositories or files
- `secret:ls`: Lists secret names from a repository
- `secret:sync`: Synchronizes secrets using a secret file
- `secret:export`: Exports secret names to a template file for manual value entry
- `sync`: Main synchronization task that combines variables and secrets

## Design Patterns

### Security Approach
Due to security restrictions, the tool cannot retrieve secret values from GitHub repositories. Instead:
1. Secret names are exported to template files
2. Users manually fill in the secret values
3. The tool synchronizes secrets using these pre-filled files

### Flexibility Design
The tool supports multiple input methods:
- Direct repository-to-repository synchronization
- File-based variable synchronization (for automation)
- Interactive mode with choice selection for secret handling

## Workflow Implementation

### Main Synchronization Process
1. List and synchronize all variables from source to target repository
2. Handle secrets in two modes:
   - Manual mode: Export secret names to template file for user input
   - Environment mode: Use existing secret file with values

### Error Handling
- Validates prerequisites (gh CLI and authentication)
- Checks for required parameters in all tasks
- Provides clear error messages when files or repositories are not found

## Usage Patterns

### Interactive Mode
```bash
task sync source=<owner>/<repo> target=<other>/<repo>
```
- Prompts user to choose between manual or environment secret modes

### Non-Interactive Mode
```bash
task sync source=<owner>/<repo> target=<other>/<repo> secret_file=<secret_file>
```
- Uses existing secret file for synchronization

### Manual Secret Mode
```bash
task sync source=<owner>/<repo> target=<other>/<repo> mode=manual
```
- Exports secret names to template file for manual editing

### Variable-only Mode
```bash
task var:sync source=<owner>/<repo> target=<other>/<repo>
```
- Synchronizes only variables without secrets

### Secret-only Mode
```bash
task secret:sync source=<owner>/<repo> target=<other>/<repo> secret_file=<secret_file>
```
- Synchronizes only secrets from a file

## Technical Details

### Environment Requirements
- GitHub CLI (`gh`) installed and authenticated
- Taskfile CLI (`task`) installed
- Shell environment with basic utilities (echo, read, etc.)

### File Format Requirements
- Secret files must follow the format: `SECRET_NAME=secret_value`
- Variable files can be used for file-based synchronization in the format: `VAR_NAME=VAR_VALUE`

### Security Considerations
- Secret values are never read by the tool directly from GitHub (by design)
- Secret files should be kept secure and not committed to repositories
- The tool provides a template system to avoid direct secret handling in code

## Design Decisions

### 1. Use of Taskfile
- Leverages Taskfile for cross-platform task management and consistent CLI interface
- Allows easy installation and usage across different operating systems

### 2. Two-Step Secret Process
- Separates secret name listing from value setting to comply with GitHub security policies
- Provides clear separation between template creation and actual secret application

### 3. Flexible Input Methods
- Supports both repository-to-repository and file-based variable synchronization
- Enables automation through file-based workflows while maintaining interactive usability

### 4. Modular Task Structure
- Each operation (variables, secrets, listing, synchronization) has dedicated tasks
- Main sync task composes these operations into a complete workflow