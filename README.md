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

## Components

### Core Distribution Workflow

The main workflow (`.github/workflows/distribute_workflows.yml`) handles:

- **Secret Distribution**: Synchronizes the `REPO_DISPATCH_TOKEN` secret across all target repositories
- **Workflow Distribution**: Copies workflow files from the `workflows/` directory to `.github/workflows/` in each target repository
- **Configuration Distribution**: Copies configuration files (like `kicad-config.yml`) to target repositories if they don't already exist

### Distributed Workflows

The system provides several specialized workflows:

#### `generate_kicad_3d_models.yml`
- Automatically generates 3D models (GLB and WRL formats) from KiCad PCB files
- Cleans up old artifacts before generation
- Uploads 3D models as GitHub Actions artifacts
- Triggers downstream workflow in 3D model hosting service

#### `generate_kicad_jobs.yml`
- Generates comprehensive KiCad documentation including:
  - PDF schematics with Gruvbox theme
  - High-quality PCB renders (top, bottom, side, left, right views)
  - Interactive HTML BOM (Bill of Materials)
- Commits generated files to the project's documentation directory
- Triggers downstream workflows for 3D model generation and symbol checking

#### `trigger-symbol-check.yml`
- Monitors for changes in documentation pictures
- Triggers symbol checking in the KiCAD Symbols Generator repository

### Configuration

#### `configs/kicad-config.yml`
Provides customizable parameters for KiCad rendering:
- Zoom factors for different views
- Pan and rotation settings for consistent rendering
- Default values can be overridden per repository

## Target Repositories

The distribution system manages workflows and secrets for the following repositories:

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

## Usage

### Adding New Repositories

To include a new repository in the distribution system:
1. Add the repository name to the `TARGET_REPOS` environment variable in `.github/workflows/distribute_workflows.yml`
2. Ensure the repository has the appropriate permissions for the `WORKFLOW_DISTRIBUTION_TOKEN`

### Triggering Distribution

The distribution workflow is triggered by:
- Pushes to the `distribute` branch
- Manual dispatch through GitHub Actions UI

### Customizing KiCad Rendering

To customize KiCad rendering parameters in your repository, create a `.github/kicad-config.yml` file with values that will override the defaults.

## Benefits

- **Consistency**: All repositories have identical CI/CD processes
- **Maintainability**: Updates to workflows happen in one place and propagate automatically
- **Scalability**: Adding new repositories to the system is straightforward
- **Quality Control**: Centralized workflow ensures best practices across all projects

## Architecture

The system works through a two-stage process:
1. **Secret Distribution**: First, secrets are pushed to all target repositories
2. **Workflow Distribution**: After successful secret distribution, workflow files are copied and branches are synchronized

This architecture ensures that all repositories have the required secrets before attempting to run workflows that depend on them.

