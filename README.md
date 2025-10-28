# Workflow Distributor

A central system for distributing GitHub Actions workflows and secrets to multiple repositories in the KiCad-based hardware projects ecosystem.

## Overview

The Workflow Distributor is a centralized system that automatically propagates GitHub Actions workflows and secrets across multiple target repositories. This approach ensures consistency, reduces maintenance overhead, and allows for centralized updates to CI/CD processes across all projects in the ecosystem.

## Features

- **Centralized Management**: Define workflows and configurations in one place
- **Multi-Repository Distribution**: Automatically propagate changes to multiple target repositories
- **Secret Synchronization**: Securely distribute secrets to all target repositories
- **KiCad Project Support**: Specialized workflows for generating documentation, 3D models, and BOMs for KiCad projects
- **Branch Synchronization**: Keep documentation branches in sync across repositories
- **Configurable Rendering**: Support for custom KiCad rendering parameters via YAML configuration
- **3D Model Generation**: Automated creation of 3D models in GLB and WRL formats
- **Footprint Management**: Advanced control over 3D model visibility and positioning for specific components

## Components

### Core Distribution Workflow

The main workflow (`.github/workflows/distribute_workflows.yml`) handles the bulk of the distribution work with two primary stages: secrets distribution and workflow distribution. The workflow is triggered by pushes to the `distribute` branch or manual dispatch through GitHub Actions UI. It has the following features and capabilities:

- **Configurable Repository List**: Uses `TARGET_REPOS` environment variable with format `repo_name|secrets|workflows` to specify which repositories receive secrets and/or workflows (true/false flags)
- **Secret Distribution**: Synchronizes the `REPO_DISPATCH_TOKEN` secret across all target repositories where the secrets flag is enabled
- **Workflow Distribution**: Copies workflow files from the `workflows/` directory to `.github/workflows/` in each target repository where the workflows flag is enabled
- **Configuration Distribution**: Copies configuration files (like `kicad-config.yml`) to target repositories if they don't already exist
- **Script Distribution**: Copies utility scripts (like `kicad_footprint_manager.py`) to target repositories to support workflow operations
- **Branch Synchronization**: Maintains synchronization between the main branch and a `doc_workflow` branch in target repositories
- **Comprehensive Logging**: Provides detailed success/failure reporting with summary statistics for all operations

### Distributed Workflows

The system provides several specialized workflows for KiCad project automation and documentation generation:

#### `generate_kicad_3d_models.yml`
- Automatically generates 3D models (GLB and WRL formats) from KiCad PCB files
- Cleans up old artifacts before generation to prevent accumulation of outdated files
- Uploads 3D models as GitHub Actions artifacts with temporary retention
- Triggers downstream workflow in the Docker_3D_Models_Hosting service via repository dispatch
- Uses a dedicated KiCad Docker image (ionutms/kicad_9:V1.1) for consistent rendering environment

#### `generate_kicad_jobs.yml`
- Generates comprehensive KiCad documentation including:
  - PDF schematics with Gruvbox theme
  - High-quality PCB renders (top, bottom, side, left, right, front, back views)
  - Interactive HTML BOM (Bill of Materials)
- Commits generated files to the project's documentation directory organized by project name
- Integrates with the kicad-config.yml configuration for customizable rendering parameters
- Uses the kicad_footprint_manager.py script to control 3D model visibility and positioning for specific components during rendering
- Triggers downstream workflows for 3D model generation and symbol checking
- Handles multiple project files with detection and error handling
- Creates project-specific output directories to avoid conflicts in multi-project repositories

### Utility Scripts

#### `scripts/kicad_footprint_manager.py`
A Python utility for managing KiCad footprint properties in .kicad_pcb files with the following capabilities:
- **Extract footprint code**: Retrieve the complete footprint definition by reference designator
- **Hide/show 3D models**: Add/remove `(hide yes)` flag from 3D model definitions to control visibility during rendering
- **Offset 3D models**: Apply coordinate offsets to 3D model positions for visualization improvements
- **Batch operations**: Process multiple footprints with different operations as specified in configuration files

### Configuration Files

#### `configs/kicad-config.yml`
Provides customizable parameters for KiCad rendering with a flexible view system. Key features include:
- **Predefined Views**: 8 different views (side, top, bottom, left, right, front, back, test) with configurable parameters
- **Customizable Parameters**: Each view can have individual settings for zoom, rotation, panning, lighting, and side selection
- **View Prefixes/Suffixes**: Configurable naming conventions for output files (e.g., '1_', '2_', etc.)
- **Default Values**: Fallback parameters that can be overridden at the view level
- **Advanced Footprint Control**: Support for hiding/showing specific footprints and applying 3D model offsets on a per-view basis

## Target Repositories

The distribution system manages workflows and secrets for the following repositories listed in the distribution workflow configuration (TARGET_REPOS environment variable):

- ionutms/Minimal_AD74416H
- ionutms/Minimal_AD74416H_Stack_Adapter
- ionutms/AD74416H_Power_Interface_Module
- ionutms/Modular_AD74416H_PLC
- ionutms/Minimal_ADP1074
- ionutms/Minimal_ADPL76030
- ionutms/Minimal_ADP1032
- ionutms/Minimal_MAX14906
- ionutms/Minimal_AD74413R
- ionutms/Modular_Software_Configurable_IO_PLC
- ionutms/Minimal_ADIN1110
- ionutms/Minimal_LTC9111
- ionutms/Minimal_MAX17761
- ionutms/Minimal_LT8304
- ionutms/Minimal_MAX32650
- ionutms/Minimal_ADP1031
- ionutms/Minimal_MAX17570
- ionutms/Docker_3D_Models_Hosting (secrets only, no workflow distribution)

## Usage

### Adding New Repositories

To include a new repository in the distribution system:
1. Add the repository name to the `TARGET_REPOS` environment variable in `.github/workflows/distribute_workflows.yml` using the format `owner/repo|secrets|workflows` (e.g., `ionutms/new_repo|true|true`)
2. Ensure the repository has the appropriate permissions for the `WORKFLOW_DISTRIBUTION_TOKEN` with write access to workflows and secrets
3. For repositories that only need secrets (like the Docker_3D_Models_Hosting service), set the workflows flag to false (e.g., `ionutms/Docker_3D_Models_Hosting|true|false`)

### Triggering Distribution

The distribution workflow is triggered by:
- Pushes to the `distribute` branch
- Manual dispatch through GitHub Actions UI with optional flags to control secret and workflow distribution separately

### Customizing KiCad Rendering

To customize KiCad rendering parameters in your repository, create a `.github/kicad-config.yml` file with values that will override the defaults. The configuration file supports all the same parameters as the default config but allows for project-specific adjustments. The system will use your repository's config file if present, falling back to the default config from the distributor if none exists. The kicad-footprint-manager script allows for advanced rendering control by hiding specific components or adjusting 3D model positions during rendering operations on a per-view basis.

### Advanced Footprint Control

The system supports advanced footprint control through the kicad-config.yml configuration file with the following features per view configuration:
- `hide_footprints`: List of reference designators to hide during rendering of this specific view (e.g., components that interfere with the view)
- `show_footprints`: List of reference designators to force show during rendering (for components normally hidden)
- `offset_footprints`: Array of objects with `reference`, `x`, `y`, `z` values to apply coordinate offsets to specific 3D models during this view's rendering

## Benefits

- **Consistency**: All repositories have identical CI/CD processes
- **Maintainability**: Updates to workflows happen in one place and propagate automatically
- **Scalability**: Adding new repositories to the system is straightforward
- **Quality Control**: Centralized workflow ensures best practices across all projects
- **Customization**: Flexible configuration options allow for project-specific adjustments without breaking standardization
- **Error Handling**: Comprehensive error handling and reporting to identify and resolve issues quickly
- **Resource Management**: Automatic cleanup of old artifacts and proper resource handling

## Architecture

The system works through a two-stage process with dependency management:
1. **Secret Distribution**: First stage distributes secrets to all target repositories where enabled, with success/failure reporting and early exit if any secrets fail to distribute
2. **Workflow Distribution**: Second stage runs after successful secret distribution (or is skipped if secrets are disabled), copying workflow files, configuration, and scripts to target repositories where enabled, and synchronizing branches

This architecture ensures that all repositories have the required secrets before attempting to run workflows that depend on them, providing a more reliable distribution system. The workflow distribution process also handles both main branch updates and doc_workflow branch synchronization, ensuring that documentation-related workflows remain functional across all target repositories.


