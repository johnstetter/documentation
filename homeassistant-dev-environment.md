# 🛠️ Home Assistant Dev/Test Instance with Docker Compose

### *A Quick-and-Dirty Guide for Hassling Less with Blueprints and Automations*

---

## Why Bother?

Because you’re tired of blowing up your main Home Assistant config while testing blueprints, YAML, or automations. This guide gets you a *sandbox* instance—zero risk, totally disposable, version-controlled, and mockable.

---

## 1. Prereqs

* **Docker** (or Docker Desktop) installed
* **Docker Compose** (`docker compose` command, not `docker-compose`)
* A host with decent resources (Linux, macOS, or WSL on Windows)
* Optional: VS Code + Remote Containers, if you like to get fancy

---

## 2. Make a Dev Folder

```sh
mkdir -p ~/ha-dev/config
cd ~/ha-dev
```

---

## 3. `compose.yml` – Your Docker Compose File

```yaml
services:
  homeassistant:
    image: ghcr.io/home-assistant/home-assistant:stable
    container_name: homeassistant-dev
    volumes:
      - ./config:/config
      - /etc/localtime:/etc/localtime:ro
    environment:
      - TZ=America/Chicago    # Change as needed
    restart: unless-stopped
    network_mode: host       # Simplest for local dev, esp. for discovery/mDNS
    privileged: true         # Needed for some integrations (Bluetooth, etc)
```

> **NOTE:**
>
> * On Windows, use `"network_mode: bridge"` and map ports instead.
> * Don’t run this alongside your “real” Home Assistant on the same network if you have physical devices paired, or you might confuse them. See [Caveats & Avoiding Conflicts](#caveats--avoiding-conflicts) below.

---

## 4. Launch That Sucker

```sh
docker compose up -d
```

First run takes a minute. UI will be at [http://localhost:8123](http://localhost:8123) (or your host IP if on another machine).

---

## 5. Create a Separate Git Repo (Optional but Good)

```sh
cd ~/ha-dev
git init
echo "config/.storage/" >> .gitignore    # Don't version the runtime .storage DB
git add .
git commit -m "feat: initial Home Assistant dev setup"
```

> You can keep all your blueprints and automations versioned here and sync to GitHub.

---

## 6. Add Some **Mock Devices**

### Why?

So you can simulate sensors, lights, switches, etc., without risking your real gear.
Home Assistant comes with “integration: `template`” and “integration: `demo`” for exactly this.

### Example: Fake Light, Motion Sensor, and Switch

**`config/configuration.yaml`:**

```yaml
homeassistant:
  name: DevTest
  latitude: 43.0731
  longitude: -89.4012
  elevation: 260
  unit_system: imperial
  currency: USD
  external_url: "http://localhost:8123"

# Demo platform for dummy devices
demo:

# Template sensor as a fake motion sensor
binary_sensor:
  - platform: template
    sensors:
      fake_motion_sensor:
        friendly_name: "Fake Motion"
        device_class: motion
        value_template: "{{ states('input_boolean.fake_motion') == 'on' }}"

# Input booleans for toggling states
input_boolean:
  fake_motion:
    name: Fake Motion
  fake_switch:
    name: Fake Switch

# Template light as a fake lamp
light:
  - platform: template
    lights:
      fake_lamp:
        friendly_name: "Fake Lamp"
        value_template: "{{ states('input_boolean.fake_switch') }}"
        turn_on:
          service: input_boolean.turn_on
          target:
            entity_id: input_boolean.fake_switch
        turn_off:
          service: input_boolean.turn_off
          target:
            entity_id: input_boolean.fake_switch
```

> Now you’ll have “Fake Motion” and “Fake Lamp” entities you can toggle from the UI, or use in your blueprint testing.

---

## 7. Develop and Debug Blueprints

* Drop your blueprints in `config/blueprints/automation/yourname/`
* Use the “Developer Tools” in the UI for real-time template debugging
* Use the mock devices in your automations/blueprints!

---

## 8. Sync Back to GitHub

```sh
git add .
git commit -m "feat: add new blueprint for testing"
git push origin main
```

---

## 9. *Pro Tips*

* **Roll back easily:** Just nuke the container or the config and start over.
* **Keep your “real” instance untouched.**
* **Do NOT use real integrations (like Z-Wave, Zigbee) on both prod and dev at once if you only have one radio/bridge!**
* **Version control is your friend.**

---

## 10. Done! Go Wild

Now you can debug, test, and play with automations without ever pissing off your main Home Assistant.

---

# 🧩 DevContainer Integration (VS Code)

VS Code’s [Dev Containers](https://code.visualstudio.com/docs/devcontainers/containers) extension lets you code *inside* a Docker container. It’s slick for quick editing, version control, and YAML syntax help.

**Requirements:**

* Docker running on your dev machine (the same host you’ll run VS Code on).
* VS Code with the [Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) extension.

## **How To Attach VS Code to Your Running Container**

1. Start your Home Assistant container (`docker compose up -d`).
2. Open VS Code on the *same host*.
3. Press `F1` (or `Cmd+Shift+P`), type: `Dev Containers: Attach to Running Container...`
4. Select `homeassistant-dev`.
5. You now have a VS Code window inside the container!

* **Edit `/config/configuration.yaml` and other files directly**.
* **Use the built-in VS Code terminal**—no SSH needed!
* **Pro tip:** Install YAML/HA extensions in your container’s VS Code window for schema hints and linting.

### If You Want a Full DevContainer Experience

You can also define a `.devcontainer/devcontainer.json` and build your own DevContainer for testing custom HA development (beyond this guide, but I can provide examples on request).

---

# 🧪 Advanced Fake Device Setups

Need more complex testing? Here’s how to get fancy:

## **Additional Template Devices**

### Add a Fake Temperature Sensor:

```yaml
sensor:
  - platform: template
    sensors:
      fake_temperature_sensor:
        friendly_name: "Fake Temp Sensor"
        unit_of_measurement: '°F'
        value_template: "{{ states('input_number.fake_temp') }}"

input_number:
  fake_temp:
    name: Fake Temp
    min: 50
    max: 90
    step: 0.1
    mode: slider
```

### Add a Fake Door/Window Sensor:

```yaml
binary_sensor:
  - platform: template
    sensors:
      fake_door:
        friendly_name: "Fake Door Sensor"
        device_class: door
        value_template: "{{ states('input_boolean.fake_door') == 'on' }}"

input_boolean:
  fake_door:
    name: Fake Door
```

### Add a Fake Switch (Example for Testing Power Automations):

```yaml
switch:
  - platform: template
    switches:
      fake_power_switch:
        friendly_name: "Fake Power Switch"
        value_template: "{{ states('input_boolean.fake_power_switch') == 'on' }}"
        turn_on:
          service: input_boolean.turn_on
          target:
            entity_id: input_boolean.fake_power_switch
        turn_off:
          service: input_boolean.turn_off
          target:
            entity_id: input_boolean.fake_power_switch

input_boolean:
  fake_power_switch:
    name: Fake Power Switch
```

## **Tips**

* Use `input_number`, `input_boolean`, and `input_select` as your "test control panel"—toggle and adjust them from the HA UI.
* Template sensors/entities reference these values, so you can simulate real-world events.

---

# 📁 Example `blueprints/` Directory Structure

Organize your blueprints for sanity and shareability!

```
config/
└── blueprints/
    └── automation/
        ├── johnstetter/
        │   ├── motion_based_lighting_v2_5.yaml
        │   └── ...
        └── homeassistant/
            └── notify_on_door_open.yaml
```

* Place your YAML blueprints here for version control and easy import/export.
* You can create folders by your username/GitHub handle, or group by function.

---

# ⚠️ Caveats & Avoiding Conflicts with Your Main HA

* **Never share your production `/config` with dev—use a different folder or named Docker volume!**
* **Only use fake/template/demo devices in dev.** Never add your *real* Z-Wave/Zigbee/Bluetooth dongles or cloud integrations to your dev container.
* **Disable device discovery** (see sample below) so dev HA doesn’t find & try to control your real devices.
* **Don’t use the same credentials for dev and prod cloud integrations** (Google, Alexa, Nabu Casa, etc.).
* **Run only one HA instance with `network_mode: host` on the same host at a time,** to avoid port conflicts.
* If you want to experiment with discovery, do it on an isolated network or VM.

---

## Sample `configuration.yaml` — **Mock Devices + Discovery Disabled**

```yaml
# Loads default set of integrations. Do not remove.
default_config:

# ======= LOCK DOWN DISCOVERY =======
zeroconf: false     # Disable mDNS/Bonjour discovery
ssdp: false         # Disable network device discovery
# discovery:        # (usually not needed, but don't enable it)

# Load frontend themes from the themes folder
frontend:
  themes: !include_dir_merge_named themes

automation: !include automations.yaml
script: !include scripts.yaml
scene: !include scenes.yaml

# ======= MOCK DEVICES FOR DEV TESTING =======
demo:

binary_sensor:
  - platform: template
    sensors:
      fake_motion_sensor:
        friendly_name: "Fake Motion"
        device_class: motion
        value_template: "{{ states('input_boolean.fake_motion') == 'on' }}"

input_boolean:
  fake_motion:
    name: Fake Motion
  fake_switch:
    name: Fake Switch

light:
  - platform: template
    lights:
      fake_lamp:
        friendly_name: "Fake Lamp"
        value_template: "{{ states('input_boolean.fake_switch') }}"
        turn_on:
          service: input_boolean.turn_on
          target:
            entity_id: input_boolean.fake_switch
        turn_off:
          service: input_boolean.turn_off
          target:
            entity_id: input_boolean.fake_switch

# ---- More Advanced Fake Devices ----
sensor:
  - platform: template
    sensors:
      fake_temperature_sensor:
        friendly_name: "Fake Temp Sensor"
        unit_of_measurement: '°F'
        value_template: "{{ states('input_number.fake_temp') }}"

input_number:
  fake_temp:
    name: Fake Temp
    min: 50
    max: 90
    step: 0.1
    mode: slider

binary_sensor:
  - platform: template
    sensors:
      fake_door:
        friendly_name: "Fake Door Sensor"
        device_class: door
        value_template: "{{ states('input_boolean.fake_door') == 'on' }}"

input_boolean:
  fake_door:
    name: Fake Door

switch:
  - platform: template
    switches:
      fake_power_switch:
        friendly_name: "Fake Power Switch"
        value_template: "{{ states('input_boolean.fake_power_switch') == 'on' }}"
        turn_on:
          service: input_boolean.turn_on
          target:
            entity_id: input_boolean.fake_power_switch
        turn_off:
          service: input_boolean.turn_off
          target:
            entity_id: input_boolean.fake_power_switch

input_boolean:
  fake_power_switch:
    name: Fake Power Switch
```

---
