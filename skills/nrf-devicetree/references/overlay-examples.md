# Overlay patterns — nRF54LM20B (nrf54lm20dk)

Adapt instance numbers and pins to the DK schematic / custom board. Always pair instances with pinctrl.

## I2C sensor (e.g., IAQ/temperature sensor)

```dts
&pinctrl {
	i2c21_default: i2c21_default {
		group1 {
			psels = <NRF_PSEL(TWIM_SDA, 1, 8)>, <NRF_PSEL(TWIM_SCL, 1, 9)>;
		};
	};
	i2c21_sleep: i2c21_sleep {
		group1 {
			psels = <NRF_PSEL(TWIM_SDA, 1, 8)>, <NRF_PSEL(TWIM_SCL, 1, 9)>;
			low-power-enable;
		};
	};
};

&i2c21 {
	status = "okay";
	pinctrl-0 = <&i2c21_default>;
	pinctrl-1 = <&i2c21_sleep>;
	pinctrl-names = "default", "sleep";
	clock-frequency = <I2C_BITRATE_FAST>;

	iaq: bme688@76 {
		compatible = "bosch,bme680";
		reg = <0x76>;
	};
};
```

C side (inside the sensors module only):

```c
static const struct device *iaq_dev = DEVICE_DT_GET(DT_NODELABEL(iaq));
/* device_is_ready(iaq_dev) before first use; then
   sensor_sample_fetch() + sensor_channel_get(SENSOR_CHAN_...) */
```

## ADC channel (e.g., illuminance via analog photodiode, battery voltage)

```dts
/ {
	zephyr,user {
		io-channels = <&adc 0>;
	};
};

&adc {
	status = "okay";
	#address-cells = <1>;
	#size-cells = <0>;

	channel@0 {
		reg = <0>;
		zephyr,gain = "ADC_GAIN_1_4";
		zephyr,reference = "ADC_REF_INTERNAL";
		zephyr,acquisition-time = <ADC_ACQ_TIME_DEFAULT>;
		zephyr,input-positive = <NRF_SAADC_AIN1>;
		zephyr,resolution = <12>;
	};
};
```

C: use `ADC_DT_SPEC_GET_BY_IDX(DT_PATH(zephyr_user), 0)` + `adc_read_dt`.

## GPIO (interrupt from sensor INT pin)

```dts
/ {
	sensor_int: sensor-int {
		compatible = "gpio-keys";
		int0: int_0 {
			gpios = <&gpio1 10 (GPIO_ACTIVE_LOW | GPIO_PULL_UP)>;
		};
	};
};
```

C: `GPIO_DT_SPEC_GET(DT_NODELABEL(int0), gpios)` + `gpio_pin_interrupt_configure_dt` + callback that only signals a work item.

## PDM microphone (sound pressure level)

```dts
&pinctrl {
	pdm20_default: pdm20_default {
		group1 {
			psels = <NRF_PSEL(PDM_CLK, 1, 12)>, <NRF_PSEL(PDM_DIN, 1, 13)>;
		};
	};
};

&pdm20 {
	status = "okay";
	pinctrl-0 = <&pdm20_default>;
	pinctrl-names = "default";
};
```

Use the Zephyr `dmic` API (`CONFIG_AUDIO_DMIC=y`); compute SPL/dBA in a dedicated module (good FLPR/Edge-AI candidate later).

## Verifying

- `west build -t menuconfig` → confirm driver symbols active.
- Inspect `build/<app>/zephyr/zephyr.dts` for the final merged tree.
- Runtime: `device_is_ready()` must be checked for every device; log one real measurement.
