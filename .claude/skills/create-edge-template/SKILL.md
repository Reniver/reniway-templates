---
name: create-edge-template
description: Create or edit Reniway Edge flow templates (the YAML files in this repo). Use when asked to add support for a new machine/CNC/controller, map a connector to the standard PostgreSQL schema, add replacement labels to a template, or generate a new template by analogy with an existing one (e.g. HEIDENHAIN, Mazak, FANUC).
---

# Creating Reniway Edge templates

## What a template is

A Reniway Edge **template** is a complete **flow** (a data pipeline) exported as a single YAML
file. A flow connects a **field connector** (the machine — e.g. a HEIDENHAIN, FANUC or Mazak CNC)
through a **data mapper** (transformation logic) into an **enterprise connector** (usually a
TimescaleDB / PostgreSQL database). Once a flow works for one machine, exporting it as a template
lets a customer re-import it to connect many similar machines in seconds.

Templates are produced by **exporting an existing flow** from the Reniway Edge UI (Flows page menu →
download), then **manually editing** the YAML to generalise it — most importantly by adding
**replacement labels** (see below). Import is the reverse (Flows page menu → upload).

Reference docs: https://docs.reniver.eu/reniway/flows/templates

## Your goal

Generate **new templates for machines that are not yet supported**, by analogy with the templates
already in `Machinery/`. Today the objective is almost always: take a new machine's field connector
and map its mode/state, alarms, overrides, program name and cycle time onto the **same standard
PostgreSQL schema** the existing templates use, so the data lands in the same dashboards regardless
of machine brand.

That database schema is the **standard CNC schema** and is the target for most current templates.
In the future, templates will target other sinks too — mapping machine data onto an **OPC UA address
space**, publishing to an **MQTT unified namespace**, streaming to the **Reniway Cloud**, etc. The
template mechanics (connectors + data mappers + replacement labels) are identical; only the
**enterprise connector** and the **transformation outputs** change. When the target isn't the CNC
database, replace the SQL connector with the appropriate enterprise connector and shape the
transformation `value` to that sink instead.

When asked for a new machine:
1. Learn **exactly which properties the field connector produces** and their **names, types and value
   vocabulary** — best from a flow exported from a Reniway Edge instance connected to that machine, or
   from an existing template that uses the same connector.
2. Use the connector's exact `type` string as it appears in an exported flow or existing template
   (e.g. `HEIDENHAIN`, `MTConnect`, `FanucFOCASBridge`).
3. Copy the structure of the closest existing `*_To_PostgreSQL_template_*.yaml` and rewrite the field
   connector + data-mapper transformations for the new machine. Keep the enterprise (SQL) connector
   and its query strings identical so the target schema stays the same.

## File layout

Templates are organised by **CNC control manufacturer**, then by **control
generation/family** — key on the control you integrate with, not the machine-tool
builder (a Hermle machine running a HEIDENHAIN TNC goes under `HEIDENHAIN/`, not
`Hermle/`). `Machinery/README.md` holds the full manufacturer → generation →
Ethernet-protocol taxonomy; many generation folders are scaffolded with a stub
`README.md` and await their first template.

```
Machinery/<Control manufacturer>/<Control generation>/<MANUFACTURER>_<Model>_To_<Target>_template_<ver>.yaml
```

Examples: `Machinery/HEIDENHAIN/TNC640/HEIDENHAIN_TNC640_To_PostgreSQL_template_1.0.yaml`,
`Machinery/FANUC/Series30i-31i-32i/FANUC_FOCAS_To_PostgreSQL_template_1.0.yaml`.

## Top-level structure

```yaml
remove: false          # true would DELETE matching flow nodes on import; keep false for templates
labels:                # replacement labels — see below
  - Machine Name : "Enter a unique name for your machine"
connectors:            # list of connectors (field + enterprise). In the model these are "bridges".
dataMappers:           # list of data mappers
opcua:                 # (optional) OPC UA address-space nodes — only present if the flow used them
```

A template has three logical pieces (see the docs):
- **connectors** — field connectors (the machine) and enterprise connectors (DB/MQTT/OPC UA server).
- **dataMappers** — logical machine entities whose properties either receive a connector value or run
  a JavaScript transformation.
- **opcua** — optional OPC UA server node hierarchy.

## Replacement labels (the key feature, not yet in the public docs)

A `labels:` block at the top of the file declares placeholders that the importer prompts the user to
fill in. Each entry is a single-key map: `Label Name : "helper text shown in the UI"`.

Anywhere in the YAML you then write `${Label Name}` and, on import, the UI replaces **every**
occurrence of `${Label Name}` with the user-entered value before the template is deserialised. The
matching is literal text replacement on the YAML string (regex `\$\{\s*Label Name\s*\}`), so labels
can appear in connector names, setting values (host, port, user, password, database), and inside
data-mapper transformation code (e.g. `'Serial': '${Machine Name}'`).

Conventional labels used across the existing templates:

| Label                  | Used for                                              |
|------------------------|-------------------------------------------------------|
| `Machine Name`         | unique machine name → connector names + DB `Serial`/`Name` |
| `Machine Model`        | DB `model`                                            |
| `Machine Manufacturer` | DB `manufacturer` (omit if a fixed brand)             |
| `Machine Url`          | field connector host/IP setting                       |
| `Machine Port`         | field connector port setting (when applicable)        |
| `Database Url`         | SQL connector `host`                                   |
| `Database Name`        | SQL connector `database`                               |
| `Database User`        | SQL connector `user`                                   |
| `Database Password`    | SQL connector `password`                               |

Validation in the UI requires every declared label to be given a non-empty value, so only declare
labels you actually reference.

## The standard CNC PostgreSQL target schema

Keep the enterprise (SQL) connector identical to the existing templates. It writes to these tables
via output properties whose `queryString` is fixed; only the **data-mapper transformations feeding
them** change per machine:

- `machines (serial, name, protocol, model, manufacturer)` — upsert (`ON CONFLICT (serial) DO UPDATE`)
- `observation_log (machine_serial, time, mode, state)` —
  `INSERT INTO observation_log VALUES (@Serial, NOW(), @Mode::Mode, @State::State);`
- `override_log (machine_serial, time, feed, speed, rapid)` — feed/speed/rapid are `smallint`
- `program_data_log (machine_serial, time, program_name, program_state, cycle_time)` —
  `cycle_time` is `integer`
- `insert_alarms(machine_serial_in text, alarms jsonb)` — pass alarms as a JSON array of
  `{Code, Message, Source, Severity, Time}`. The function de-duplicates active alarms and emits a
  deactivation row when an alarm clears, so just send the **currently-active** alarm set each cycle.

**Allowed enum values** (case-sensitive — your transformations must
output exactly these):
- `machine_mode`: `Manual`, `MDI`, `RFP`, `SingleStep`, `Automatic`, `Other`, `Handwheel`
- `machine_state`: `NotConnected`, `Connected`, `Booted`, `Initializing`, `Available`, `ShuttingDown`
- `program_state`: `Idle`, `Running`, `Completed`, `Error`, `Stopped`, `Interrupted`, `Finished`,
  `Not Selected`

> Cast naming note: the schema names the enum types `machine_mode` / `machine_state`, but every
> existing template (and therefore the deployed databases they run against) casts with `@Mode::Mode`
> and `@State::State`. Match the **existing templates** verbatim so a new template interoperates with
> the same customer databases; do not "fix" the casts to `::machine_mode` unless the live schema is
> confirmed to use those names. `program_state` is consistent in both.

Your job per machine is to translate that machine's native vocabulary into these enums inside the
data-mapper transformation code.

## How connectors, mappers and IDs wire together

All `id` values in a template are **template-local** integers. On import they are stripped and
remapped to real IDs; only the relationships matter, so keep them internally consistent and unique.

- **Field connector property** → a value read from the machine.
- **Data-mapper input property** links to a field connector property via
  `propertyLinks: [{ sourceBridgePropertyId: <field prop id> }]`.
- **Data-mapper transformation property** has a `transformationCodeBody` (JavaScript) and an **empty**
  `propertyLinks`. It reads its inputs through `msg.payload[<data-mapper property id>]`. The importer
  scans the code for `msg.payload[<id>]` / `msg.timestamps[<id>]`, remaps each id, and auto-creates
  the transformation's source links — so the only thing wiring a transformation to its inputs is the
  `msg.payload[...]` ids in the code.
- **Enterprise (SQL) output property** links to the transformation that feeds it via
  `propertyLinks: [{ sourceDataMapperPropertyId: <transformation prop id> }]`, plus its fixed
  `queryString` and `parameters`.

`transformationCodeBody` return shape: `return { value: { Param1: ..., Param2: ... } }` where the keys
match the `parameters` of the SQL output property it feeds.

> Tip: write transformation code as a YAML literal block scalar (`|-`) so newlines are preserved
> verbatim. The UI export uses folded scalars (`>-`) with blank lines between every line — valid but
> hard to read; `|-` is cleaner and equivalent.

### Keep the wires from crossing (property order vs. declaration order)

A property's vertical position in a flow node has no effect on data flow, but if the two ends of a
wire sit at different heights the wire renders **crossed** on import. The renderer draws nodes in
separate sections (connector: normal props then `isSystemProperty: true` pinned to the **bottom**;
data mapper: **input** props, then **transformation** props in a section below). **Crucially, what
controls a section's order differs between node types — and this is a sharp edge:**

- **Connectors (field & enterprise) and data-mapper *inputs*** keep their `order:` field on import
  (added as a batch), so their vertical order = `order:`.
- **Data-mapper *transformations*** (props with `transformationCodeBody`) have their `order`
  **reassigned on import** to insertion sequence — i.e. the **order they are declared in the YAML
  array**. Their `order:` value is ignored.

So to keep `mapper transformations → SQL outputs` uncrossed, the **declaration order of the
transformation blocks in the YAML** must match the **`order:` sequence of the SQL output properties**
they feed. The classic bug: `Program Data Log` is declared *last* among the transformations (it often
has the highest id, so a UI export writes it last) while the `ProgramDataLog` SQL output has `order`
that puts it 2nd — that one wire crosses. Fix either end: move the transformation block in the YAML,
**or** (easier, lower-risk) renumber the SQL outputs' `order:` so `ProgramDataLog` is last, matching
its transformation. The other templates in this repo put the SQL outputs in the sequence
Machine → Observation → Override → Alarm → ProgramData to line up with that declaration order.

For `field → mapper inputs`, both ends use `order:`, so just give the mapper inputs the same
top-to-bottom sequence as the field-connector properties they link to. Because the field connector's
`Status` is a *system* property (bottom), the mapper's `Status` input must be the **last** input
(highest `order`).

Verify with: for each wire, sort by the source-side position and confirm the target-side positions
also increase — but compute the transformation positions from **declaration order**, not `order:`.

## ⚠️ Critical, connector-specific gotcha: how a field connector matches its properties

Different field connectors match incoming machine data to template properties **differently**. Get
this wrong and the connector silently creates duplicate properties and nothing flows:

- **MTConnect**: matches by `dataItemId` (property carries `dataItemId: mode`).
- **HEIDENHAIN**: matches by `address` (property carries `address: ExecutionMode`).
- **FANUC FOCAS**: matches by **property `name`**. The connector flattens its data classes into names
  like `StateData_Mode`, `StateData_Override_FeedOverride`, `ProductionData_Executing_Name`,
  `ProductionData_Timers_CycleTimeMs`, `Alarms`. The field-connector property `name` in the template
  **must equal that generated name exactly** — do **not** prefix it with `${Machine Name}` and do not
  rename it. You may still give the data-mapper properties friendly names — only the field
  connector's property names are constrained.

Always check an exported flow or existing template for the same connector before writing the field
connector block, to confirm the match key, the property names/types, and the settings keys.

## Checklist before finishing

- [ ] `labels:` declares every `${...}` you use, and nothing you don't.
- [ ] Field connector `type` matches the connector's exact type string; `connectorCategoryType: Field`.
- [ ] Field connector settings keys match those of an exported flow for that connector (host/port/feature flags).
- [ ] Field connector property names match the connector's match key (name / address / dataItemId).
- [ ] Every transformation produces only the **allowed enum values** for Mode/State/program_state.
- [ ] SQL connector query strings + parameters are unchanged from the reference template.
- [ ] All `msg.payload[<id>]` ids point at real data-mapper property ids; all `propertyLinks` ids are
      consistent and unique.
- [ ] File saved under `Machinery/<Manufacturer>/<Model>/..._To_PostgreSQL_template_<ver>.yaml`.
