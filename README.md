# DIY CO2 sensor

A small, 3D-printed, WiFi-connected CO2/temperature/humidity sensor built on an ESP32 and a Sensirion SCD41.  
Data is exposed via [ESPHome](https://esphome.io/) and scraped with [Prometheus](https://prometheus.io/) so it can be visualized in [Grafana](https://grafana.com/).

![Running sensor](img/real_running.jpg)

## Features

- CO2, temperature, and humidity readings from an SCD41
- Small OLED display cycling through CO2 / temperature / humidity
- WiFi-connected
- Metrics exposed via Prometheus for easy Grafana dashboards
- Fully 3D-printed case

## Hardware requirements

- ESP32-C3 board with SSD1306-based OLED display, 72x40, I2C
- [Sensirion SCD41](https://sensirion.com/products/catalog/SCD41) CO2/temperature/humidity sensor
- 3D printer to print the [case](case.3mf) and [lid](lid.3mf)

## Case

The case and lid are provided as `.3mf` files, ready to slice and print.

### Sketch

![Sketch](img/sketch.png)

### 3D case

| [Case](case.3mf) | [Lid](lid.3mf) |
| --- | --- |
| ![Case top](img/case_top.png) | ![Lid top](img/lid_top.png) |
| ![Case ortho](img/case_ortho.png) | ![Lid ortho](img/lid_ortho.png) |
| | ![Lid bottom](img/lid_bottom.png) |

### Assembled sensors

| | | |
| --- | --- | --- |
| ![Closed](img/real_closed.jpg) | ![Open](img/real_open.jpg) | ![Multiple](img/real_multiple.jpg) |

## Wiring

The SCD41 and the OLED display both sit on the same I2C bus, but they've different addresses.  
Internally, the I2C bus is wired to the GPIO5 & GPIO6 pins.

Connect the SCD41 to the ESP like this:

| Signal | ESP32-C3 pin |
| --- | --- |
| SDA | GPIO5 |
| SCL | GPIO6 |

## ESPHome setup

1. Install [ESPHome](https://esphome.io/guides/getting_started_command_line.html).
2. Create the required secrets, such as:
   - `wifi_ssid`: The WiFi SSID
   - `wifi_password`: The WiFi password / passphrase
   - `api_encryption_key`: The encryption key for the API
   - `ota_update_password`: The password for the OTA updates
   - `fallback_hotspot_password`: The password for the WiFi fallback hotspots
3. Create a new device and use the [`esphome_sensor.yml`](`esphome_sensor.yml`) as a template
4. Build and flash the firmware image to the ESP32

The device exposes a `/metrics` endpoint (via the ESPHome [Prometheus component](https://esphome.io/components/prometheus/)) that a Prometheus server can scrape.

> [!important]
> Prometheus scraps the metrics directly from the ESP32 and **not** from the web-based [ESPHome Device Builder](https://esphome.io/guides/getting_started_command_line/#bonus-esphome-device-builder).
> Thus ESPHome is only used to build & flash the image and the «Device Builder» isn't required for operations.  
> 
> However, maintaining the devices is much easier via «Device Builder» because:
> 
> - You've ESPHome in a central instance, i.e. in a simple Docker container
> - It's much more convenient, as you've a simple management interface for all of your devices
> - ESPs can be flashed from your machine via your browser (thanks to [Web Serial API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Serial_API))
> - If you use your browser, you don't have to run the dockerised ESPHome with critical permissions, and you don't have to pass the USB device into the container

## Grafana dashboard

The [`grafana_dashboard.yml`](grafana_dashboard.yml) contains a dashboard definition for visualizing CO2, temperature, and humidity over time, fed by the Prometheus metrics above.

![Grafana dashboard](img/grafana.png)

> [!NOTE]
> Import the [`grafana_dashboard.yml`](grafana_dashboard.yml) into Grafana under Dashboards → Import.
