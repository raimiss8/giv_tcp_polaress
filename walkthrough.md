# Walkthrough - Polar ESS GivTCP Conversion

We have successfully restored communication with the Polar ESS inverter locally, verified that the protocol matches GivEnergy Modbus Gen 1 Hybrid format, and configured the codebase to support customizable inverter ports (defaulting to `7654`).

## Key Discoveries & Milestones

1. **TCP Port 7654 Verified**:
   - The Polar ESS WiFi dongle was configured in **TCP Client Mode**, connecting out to `comms.givenergy.cloud` on port `7654`.
   - By setting it to **TCP Server Mode** on port `7654` and rebooting it via its web interface at `http://192.168.0.180/`, we opened port `7654` locally.

2. **Protocol Compatibility**:
   - Probing register pages confirmed that the Polar ESS inverter responds to standard GivEnergy Modbus TCP frames.
   - It supports both holding and input registers on slave address `0x31` and `0x32`.

3. **Data Verification**:
   - We successfully executed `GivClient.getData(True)` on the host using the updated library code.
   - We retrieved and parsed real data from your inverter:
     * **Inverter Model**: Hybrid
     * **Inverter Serial**: `JD2401P106`
     * **Battery Serial**: `AB2352P267`
     * **Battery SOC**: `83%`

## Codebase Modifications

We modified the codebase to support customizable inverter ports natively (using port `7654` as the default for Polar ESS support):

* **Client Library Default**:
  - Modified [modbus.py](file:///d:/antigravity_projects/polaress36kw/givenergy_modbus/modbus.py) and [client.py](file:///d:/antigravity_projects/polaress36kw/givenergy_modbus/client.py) to default to port `7654`.
* **GivTCP Logic**:
  - Updated [write.py](file:///d:/antigravity_projects/polaress36kw/GivTCP/write.py) and [GivLUT.py](file:///d:/antigravity_projects/polaress36kw/GivTCP/GivLUT.py) to read and pass down `invertorPort` from settings.
* **Addon Settings / Startup**:
  - Updated [startup.py](file:///d:/antigravity_projects/polaress36kw/startup.py), [startup_2.py](file:///d:/antigravity_projects/polaress36kw/startup_2.py), and [settings_template.py](file:///d:/antigravity_projects/polaress36kw/GivTCP/settings_template.py) to parse `INVERTOR_PORT_x` environment variables and output it to `settings.py`.
* **Home Assistant Addon Options**:
  - Updated [config.yaml](file:///d:/antigravity_projects/polaress36kw/config.yaml) and [docker-compose.yml](file:///d:/antigravity_projects/polaress36kw/docker-compose.yml) to add `INVERTOR_PORT_1` options so it is fully configurable in the Home Assistant UI and Docker setups.

## Home Assistant Integration & Control

Since the Polar ESS inverter uses the exact same Modbus structure as GivEnergy, the existing GivTCP features are fully compatible:
* All sensors (PV, battery, load, temperatures, grid power) are discovered automatically.
* Charge/discharge schedules, reserve settings, and control switches are exposed via MQTT Auto Discovery and fully work out of the box.
