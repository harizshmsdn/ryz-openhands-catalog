---
name: conda-cross-platform-manager
description: >
  Enforces YAML-first approach for managing conda environments and exports clean conda environments to be used by other users regardless of OS.
version: "1.0"
compatibility: Requires conda and bash. Works on darwin, windows, and linux.
metadata:
  author: ryz
triggers:
  - install package
  - export conda environment
  - save conda environment
  - update environment
  - share environment
  - remove dependency 
---

# Conda Cross-Platform Manager

Primary goal is to maintain a clean `environment.yml` file that is free of platform-specific packages, dependencies, and versions. This allows for easy sharing and updating of conda environments across different operating systems. 

## Core Instructions 

Depending on the user's request, follow the approriate workflow to either export a clean environment, update an existing `environment.yml`, or remove unnecessary dependencies.

### Workflow A: Installing, Updating, or Removing Packages (YAML-first approach)

When asked to install, update, or remove a package, use the following workflow:

  1. **Never use imperative commands** like `conda install`, `conda update`, or `conda remove` directly. Instead, always modify the `environment.yml` file first.

  2. **Edit the YAML directly:** Open the `environment.yml` file and add, update, or remove the package from the `dependencies` section. For example, to add a package, include it in the list:
    ```yaml
    dependencies:
      - numpy
      - pandas
      - matplotlib
    ```

  3. **Sync the environment:** Apply the updated YAML file to the existing environment by using a prune flag
    ```bash
    conda env update --file environment.yml --prune
    ```

### Workflow B: Exporting a Clean Environment (OS-agnostic)

When asked to export, save, or capture the current conda environment, use the following workflow:

  1. **Use history:** Never run a raw `conda env export > environment.yml` command. Always use the --from-history flag:
    ```bash
    conda env export --from-history > environment.yml
    ```
  2. **Remove local paths:** After exporting, open the `environment.yml` file and remove the prefix: /path/to/local/env line at the very bottom of the file
  3. **Restore pip packages:** If the project uses pip packages, ensure that the `pip` section is included in the `environment.yml` file. If not, add it manually:
    ```yaml
    dependencies:
      - pip
      - pip:
        - package1
        - package2
    ```



