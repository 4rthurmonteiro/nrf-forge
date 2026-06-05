# Project template — NCS v3.3.0, nRF54LM20B

Exact starter file contents. Replace `app_name` tokens.

## CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.20.0)
find_package(Zephyr REQUIRED HINTS $ENV{ZEPHYR_BASE})
project(app_name)

target_sources(app PRIVATE src/main.c)
# Add further modules: target_sources(app PRIVATE src/sensors.c src/ble.c ...)
```

## prj.conf

```ini
# --- Logging (RTT + deferred mode) ---
CONFIG_LOG=y
CONFIG_LOG_MODE_DEFERRED=y
CONFIG_USE_SEGGER_RTT=y
CONFIG_LOG_BACKEND_RTT=y

# --- Reboot/diagnostics ---
CONFIG_REBOOT=y

# --- Keep main stack comfortable while prototyping ---
CONFIG_MAIN_STACK_SIZE=2048
```

## sysbuild.conf

```ini
# Bootloader off for the first build; nrf-dfu skill enables it later:
# SB_CONFIG_BOOTLOADER_MCUBOOT=y
```

## VERSION

```
VERSION_MAJOR = 0
VERSION_MINOR = 1
PATCHLEVEL = 0
VERSION_TWEAK = 0
EXTRAVERSION =
```

## boards/nrf54lm20dk_nrf54lm20b_cpuapp.overlay

```dts
/ {
	/* Project hardware aliases go here, e.g.:
	 * aliases { iaq-sensor = &bme688; };
	 */
};

/* Enable/configure peripherals per feature work (see nrf-devicetree skill) */
```

## src/main.c

```c
#include <zephyr/kernel.h>
#include <zephyr/logging/log.h>

LOG_MODULE_REGISTER(main, LOG_LEVEL_INF);

int main(void)
{
	LOG_INF("app_name booted (NCS v3.3.0, nRF54LM20B)");

	while (1) {
		k_sleep(K_SECONDS(60));
	}
	return 0;
}
```

## .gitignore

```
build/
*.pyc
.vscode/.cortex-debug.*
```

## Build & flash commands

```bash
# from the app directory, inside the NCS v3.3.0 toolchain env
west build -b nrf54lm20dk/nrf54lm20b/cpuapp --sysbuild
west flash            # add --erase for full-chip erase
west build -t pristine  # clean rebuild
```
