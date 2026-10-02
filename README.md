# Reniway Edge Templates

Ready-to-use flow templates for **[Reniway Edge](https://reniver.eu)**, the industrial edge platform by [Reniver](https://reniver.eu). Use them to connect CNC machines and industrial equipment to databases, OPC UA and MQTT / Unified Namespace (UNS) in minutes, without building every data flow yourself.

Each template is a complete Reniway **flow** exported as a single YAML file. A flow runs from the machine connector (e.g. FANUC FOCAS, HEIDENHAIN DNC, MTConnect, OPC UA) through a data mapper to an enterprise connector such as PostgreSQL / TimescaleDB or the built-in Reniway OPC UA Server. Import a template, fill in a few labels (machine name, IP address, …) and the machine is connected.

We're building a growing library of templates for equipment, machinery and sensors so customers can get started quickly.

📖 **Documentation:** [docs.reniver.eu/reniway](https://docs.reniver.eu/reniway) · **Templates guide:** [docs.reniver.eu/reniway/flows/templates](https://docs.reniver.eu/reniway/flows/templates)
🌐 **Website:** [reniver.eu](https://reniver.eu)

## Available templates

| Control | Protocol | Target | Template |
|---------|----------|--------|----------|
| FANUC Series 30i / 31i / 32i | FANUC FOCAS2 | PostgreSQL | [FANUC_FOCAS_To_PostgreSQL](Machinery/FANUC/Series30i-31i-32i/FANUC_FOCAS_To_PostgreSQL_template_1.0.yaml) |
| HEIDENHAIN iTNC 530 | HEIDENHAIN DNC | PostgreSQL | [HEIDENHAIN_iTNC530_DNC_To_PostgreSQL](Machinery/HEIDENHAIN/iTNC530/HEIDENHAIN_iTNC530_DNC_To_PostgreSQL_template_1.0.yaml) |
| HEIDENHAIN iTNC 530 | HEIDENHAIN DNC | OPC UA Server | [HEIDENHAIN-iTNC530_DNC_To_OPC-UA-Server](Machinery/HEIDENHAIN/iTNC530/HEIDENHAIN-iTNC530_DNC_To_OPC-UA-Server_template_1.1.yaml) |
| HEIDENHAIN TNC 640 | HEIDENHAIN DNC | PostgreSQL | [HEIDENHAIN_TNC640_To_PostgreSQL](Machinery/HEIDENHAIN/TNC640/HEIDENHAIN_TNC640_To_PostgreSQL_template_1.0.yaml) |
| Siemens SINUMERIK 840D sl (Solutionline) | OPC UA | PostgreSQL | [SIEMENS_SolutionLine840d_To_PostgreSQL](Machinery/Siemens%20SINUMERIK/Solutionline/SIEMENS_SolutionLine840d_To_PostgreSQL_Database.yaml) |
| Okuma OSP-P200 | MTConnect | PostgreSQL | [OKUMA_OSP-P200_MTConnect_To_PostgreSQL](Machinery/Okuma/OSP-P200/OKUMA_OSP-P200_MTConnect_To_PostgreSQL_template_1.0.yaml) |
| Mazak Smooth | MTConnect | PostgreSQL | [Mazak_Smooth_MTConnect_To_PostgreSQL](Machinery/Mazak/Smooth/Mazak_Smooth_MTConnect_To_PostgreSQL_template_1.0.yaml) |

The PostgreSQL templates all write to the same standard CNC schema (machine state, mode, alarms, overrides, program name, cycle time), so machines from different brands show up in the same dashboards.

## Planned controls

Folders already exist for these controls, and templates will be added over time:

- **FANUC**: Series 0i, Series 16i / 18i / 21i, Power Motion i
- **HEIDENHAIN**: TNC 620, TNC7, MANUALplus 620, CNC PILOT 640
- **Siemens SINUMERIK**: 828D, SINUMERIK ONE, Powerline
- **Mitsubishi Electric**: M800 / M80, M700 / M70, C80
- **Okuma**: OSP-P300, OSP-P500
- **Mazak**: Mazatrol Matrix
- **Haas**: NGC, Classic

See [Machinery/README.md](Machinery/README.md) for how the library is organised and which Ethernet protocols each control supports. Need a control that isn't listed? [Contact Reniver](https://reniver.eu).

## Using a template

1. Download the template `.yaml` file for your control.
2. In Reniway Edge, open the **Flows** page and choose **Upload** from the menu.
3. Fill in the labels the template asks for, such as machine name and IP address / hostname.
4. Start the flow.

Full instructions are in the [Reniway templates documentation](https://docs.reniver.eu/reniway/flows/templates).

> Protocol availability depends on the control's software version and purchased options. Always check against the specific machine.

## About Reniway Edge

Reniway Edge connects machines, PLCs, CNCs and industrial equipment. It processes and standardises machine data at the edge and makes it available to MES, ERP, cloud and AI applications through OPC UA and MQTT / Unified Namespace. Supported connectivity includes OPC UA, MQTT, Modbus, MTConnect, HEIDENHAIN DNC, EtherNet/IP, TwinCAT ADS, FANUC FOCAS, SINUMERIK, SQL and InfluxDB.

Learn more at [reniver.eu](https://reniver.eu).
