# NAMD Configuration Generator

This system allows you to generate NAMD configuration files for different stages of molecular dynamics simulations using Jinja2 templates and YAML configuration.

## Files

- `namd_template.j2`: Jinja2 template for NAMD configuration files
- `namd_config.yaml`: YAML configuration with default settings for all simulation stages
- `generate_namd.py`: Python script to generate NAMD files from the template and configuration
- Original NAMD files:
  - `01_Minimization.namd`: Minimization stage configuration
  - `02_Equilibration.namd`: Equilibration stage configuration
  - `03_Production_npt.namd`: Production stage configuration

## Requirements

- Python 3
- PyYAML
- Jinja2

You can install the required packages using pip:

```bash
pip install jinja2 pyyaml
```
or
```bash
conda install jinja2 pyyaml
```

## Usage

To generate NAMD configuration files:

```bash
./generate_namd.py [options]
```

### Options

- `--config`, `-c`: Path to the YAML configuration file (default: namd_config.yaml)
- `--template`, `-t`: Path to the Jinja2 template file (default: namd_template.j2)
- `--output-dir`, `-o`: Directory to save generated NAMD files (default: current directory)
- `--stages`, `-s`: Stages to generate. Choices: minimization, equilibration, production, all (default: all)
- `--temperatures`, `-T`: List of temperatures to generate files for (overrides the temperature in the config file)
- `--temp-prefix`, `-p`: Add temperature as prefix to output directories (T{temp})
- `--no-organize`, `-n`: Disable automatic organization into subfolder based on PSF name

### Examples

Generate all NAMD configuration files:

```bash
./generate_namd.py
```

This will create a folder structure based on the PSF filename:
```
./
└── psf_basename/
    ├── 01_Minimization.namd
    ├── 02_Equilibration.namd
    └── 03_Production_npt.namd
```

Generate only the minimization stage configuration:

```bash
./generate_namd.py --stages minimization
```

Generate multiple specific stages:

```bash
./generate_namd.py --stages minimization equilibration
```

Output files to a different directory:

```bash
./generate_namd.py --output-dir /path/to/output
```

Use a custom configuration file:

```bash
./generate_namd.py --config custom_config.yaml
```

Generate files for multiple temperatures:

```bash
./generate_namd.py --temperatures 300 310 320
```

This will create separate directories for each temperature (T300, T310, T320) and generate the NAMD files in each directory.

Generate files without organizing into a PSF-named subfolder:

```bash
./generate_namd.py --output-dir test --no-organize
```

## File Organization

By default, the script organizes output based on the structure file name:

1. It automatically creates a directory named after the PSF file (without extension) and places all NAMD configuration files inside.

2. To disable this behavior, use the `--no-organize` / `-n` option:

```bash
./generate_namd.py --no-organize
```

3. You can combine with other options:

```bash
./generate_namd.py --temperatures 300 320 --output-dir simulations/
```

This would create a folder structure like:
```
simulations/
├── T300/
│   └── psf_basename/
│       ├── 01_Minimization.namd
│       ├── 02_Equilibration.namd
│       └── 03_Production_npt.namd
└── T320/
    └── psf_basename/
        ├── 01_Minimization.namd
        ├── 02_Equilibration.namd
        └── 03_Production_npt.namd
```

## Multiple Temperature Simulations

To generate configuration files for multiple temperature simulations:

1. Use the `--temperatures` option followed by a list of temperatures:

```bash
./generate_namd.py --temperatures 300 310 320 330
```

2. The script will create separate directories for each temperature (e.g., T300, T310, etc.) and place the corresponding NAMD files inside.

3. You can combine with other options:

```bash
./generate_namd.py --temperatures 300 320 --stages minimization --output-dir simulations/
```

This would create directories like `simulations/T300/psf_basename/` and `simulations/T320/psf_basename/` with only the minimization files.

## Customizing NAMD Parameters

Edit the `namd_config.yaml` file to customize parameters for your simulation. The configuration file is organized into sections:

- `common`: Parameters shared across all stages
- `minimization`: Parameters specific to the minimization stage
- `equilibration`: Parameters specific to the equilibration stage
- `production`: Parameters specific to the production stage

You can add or modify parameters as needed for your specific simulation requirements.

### Using Multiple Parameter Files

The template supports using either a single parameter file or multiple parameter files. To use multiple parameter files:

1. In the YAML configuration, replace `parameters_file` with `parameters_files` as a list:

```yaml
# Using a single parameter file
common:
  parameters_file: "../common/final.params"
  
# Using multiple parameter files
common:
  parameters_files:
    - "../common/par_all36_prot.prm"
    - "../common/par_all36_lipid.prm"
    - "../common/par_all36_cgenff.prm"
```

2. You can define multiple parameter files in the `common` section to use them across all stages, or in specific stage sections to use different parameter files for different stages.

3. If both `parameters_file` and `parameters_files` are defined in the same section, the template will use `parameters_files`.

## Example Configuration

To simulate a different system (e.g., protein in water) at multiple temperatures, update the YAML configuration and use the command line options:

```yaml
# Common settings
common:
  structure_file: "../common/protein_water.psf"
  parameters_files:
    - "../common/par_all36_prot.prm"
    - "../common/par_all36_cgenff.prm"
    - "../common/par_water.prm"
  temperature: 310  # Default temperature (can be overridden by --temperatures)
  # ... other common settings ...

# Minimization stage config
minimization:
  job_description: "Minimization of protein in water"
  coordinates_file: "../common/protein_water.pdb"
  # ... other minimization settings ...
```

Then generate files for multiple temperatures:

```bash
./generate_namd.py --temperatures 300 310 320 330
``` 