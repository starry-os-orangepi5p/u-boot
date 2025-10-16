# OrangePi 5 Plus U-Boot Environment Configuration

## Overview

This document describes how to enable persistent environment storage for the OrangePi 5 Plus U-Boot using SPI Flash.

## Background

By default, the OrangePi 5 Plus U-Boot configuration does not support saving environment variables. The environment exists only in RAM and is lost when the board is powered off. This configuration enables the `saveenv` command to persist environment variables to SPI Flash.

## Configuration Changes

The following configurations were added to `configs/orangepi-5-plus-rk3588_defconfig`:

```
CONFIG_CMD_ENV_EXISTS=y
CONFIG_CMD_SAVEENV=y
CONFIG_ENV_IS_IN_SPI_FLASH=y
```

### Configuration Details

| Option | Description |
|--------|-------------|
| `CONFIG_CMD_SAVEENV=y` | Enables the `saveenv` command to save environment variables |
| `CONFIG_CMD_ENV_EXISTS=y` | Enables the `env exists` command to check if variables exist |
| `CONFIG_ENV_IS_IN_SPI_FLASH=y` | Configures environment storage location to SPI Flash |

## Default Environment Parameters

The Rockchip platform provides default values for SPI Flash environment storage:

| Parameter | Value | Description |
|-----------|-------|-------------|
| `CONFIG_ENV_OFFSET` | 0x140000 (1.25MB) | Offset from the start of SPI Flash |
| `CONFIG_ENV_SIZE` | 0x2000 (8KB) | Size of environment storage area |
| `CONFIG_ENV_SECT_SIZE` | 0x2000 (8KB) | SPI Flash sector/erase size |

These defaults are defined in `env/Kconfig` for `ARCH_ROCKCHIP` when `ENV_IS_IN_SPI_FLASH=y`.

## SPI Flash Layout

```
0x000000 ┌─────────────────────┐
         │                     │
         │   U-Boot SPL        │
         │                     │
0x060000 ├─────────────────────┤
         │                     │
         │   U-Boot proper     │
         │                     │
0x140000 ├─────────────────────┤
         │   Environment       │  ← 8KB (CONFIG_ENV_SIZE)
0x142000 ├─────────────────────┤
         │                     │
         │   Available space   │
         │                     │
         └─────────────────────┘
```

## Usage

### Building U-Boot

After making these configuration changes, rebuild U-Boot:

```bash
make orangepi-5-plus-rk3588_defconfig
make -j$(nproc)
```

### Using saveenv Command

Once U-Boot is built and flashed, you can save environment variables:

```
# Set an environment variable
setenv myvar "hello world"

# Save to SPI Flash
saveenv

# Verify it persists after reboot
reset
printenv myvar
```

### Checking Environment

```
# Check if a variable exists
env exists myvar

# Print all environment variables
printenv

# Print specific variable
printenv bootcmd
```

## SPI Flash Requirements

The OrangePi 5 Plus configuration already includes SPI Flash support:

- `CONFIG_ROCKCHIP_SPI_IMAGE=y` - SPI image support
- `CONFIG_SPI_FLASH_SFDP_SUPPORT=y` - SFDP support
- `CONFIG_SPI_FLASH_XMC=y` - XMC SPI Flash support
- `CONFIG_ROCKCHIP_SFC=y` - Rockchip SPI Flash Controller
- `CONFIG_SF_DEFAULT_BUS=5` - SPI bus 5
- `CONFIG_SF_DEFAULT_SPEED=24000000` - 24MHz

## Alternative: MMC/eMMC Storage

If you prefer to store the environment in MMC/eMMC instead of SPI Flash, use:

```
CONFIG_ENV_IS_IN_MMC=y
```

With Rockchip defaults:
- Offset: 0x3F8000 (4MB - 32KB)
- Size: 0x8000 (32KB)

## References

- U-Boot environment configuration: `env/Kconfig`
- OrangePi 5 Plus defconfig: `configs/orangepi-5-plus-rk3588_defconfig`
- U-Boot documentation: `doc/usage/environment.rst`
