# Machinery templates

Templates are organised by **CNC control manufacturer** (the system you actually
integrate with over the network), then by **control generation / family**. We
deliberately key on the *control*, not on the machine-tool builder: a Hermle
machining centre running a HEIDENHAIN TNC belongs under `HEIDENHAIN/`, not under
a "Hermle" folder.

Each leaf folder targets a control generation that supports an Ethernet-based
integration protocol (OPC UA, MTConnect, HEIDENHAIN DNC, FANUC FOCAS, EZSocket,
THINC-API, …). To create a template, see
[`.claude/skills/create-edge-template/SKILL.md`](../.claude/skills/create-edge-template/SKILL.md).

> Some manufacturers below (Mazak, Okuma, Haas) also build the machine — but they
> build their **own control**, so the control is still the integration target and
> gets a folder. Pure machine builders that fit a third-party control (Hermle,
> DMG MORI, Grob, Mikron, …) do **not** get a folder; their machines live under
> whichever control they ship with.

| Control manufacturer | Folder | Ethernet integration |
|----------------------|--------|----------------------|
| HEIDENHAIN | `HEIDENHAIN/iTNC530` | HEIDENHAIN DNC over Ethernet |
| HEIDENHAIN | `HEIDENHAIN/TNC620` | HEIDENHAIN DNC; OPC UA Server (option) |
| HEIDENHAIN | `HEIDENHAIN/TNC640` | HEIDENHAIN DNC; OPC UA Server (option) |
| HEIDENHAIN | `HEIDENHAIN/TNC7` | OPC UA Server; HEIDENHAIN DNC |
| HEIDENHAIN | `HEIDENHAIN/MANUALplus620` | HEIDENHAIN DNC over Ethernet |
| HEIDENHAIN | `HEIDENHAIN/CNCpilot640` | HEIDENHAIN DNC; OPC UA Server (option) |
| FANUC | `FANUC/Series0i` | FOCAS2 Ethernet |
| FANUC | `FANUC/Series16i-18i-21i` | FOCAS Ethernet |
| FANUC | `FANUC/Series30i-31i-32i` | FOCAS2 Ethernet; MT-LINKi |
| FANUC | `FANUC/PowerMotion-i` | FOCAS2 Ethernet |
| Siemens SINUMERIK | `Siemens SINUMERIK/Powerline` | Legacy; Ethernet via SINUMERIK Integrate (limited) |
| Siemens SINUMERIK | `Siemens SINUMERIK/Solutionline` | OPC UA (Access MyMachine / SINUMERIK Integrate) |
| Siemens SINUMERIK | `Siemens SINUMERIK/828D` | OPC UA |
| Siemens SINUMERIK | `Siemens SINUMERIK/SINUMERIK-ONE` | OPC UA (native) |
| Mitsubishi Electric CNC | `Mitsubishi Electric/M800-M80` | EZSocket; OPC UA; MTConnect |
| Mitsubishi Electric CNC | `Mitsubishi Electric/M700-M70` | EZSocket; MTConnect |
| Mitsubishi Electric CNC | `Mitsubishi Electric/C80` | EZSocket Ethernet |
| Okuma (OSP) | `Okuma/OSP-P300` | THINC API; MTConnect |
| Okuma (OSP) | `Okuma/OSP-P200` | THINC API; MTConnect |
| Okuma (OSP) | `Okuma/OSP-P500` | THINC API; MTConnect |
| Mazak (Mazatrol) | `Mazak/Smooth` | MTConnect |
| Mazak (Mazatrol) | `Mazak/Matrix` | MTConnect (adapter) |
| Haas | `Haas/NGC` | MTConnect |
| Haas | `Haas/Classic` | MDC / limited (legacy) |

_Protocol availability depends on the control's software version and purchased
options — always verify against the specific machine._
