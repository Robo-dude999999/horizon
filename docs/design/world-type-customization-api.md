# World Type and Customization API

Status: proposal for `world_type/customization_api`

## Goals

Provide a general extension point for native mods to register world types and customize their creation settings without hardcoding any mod into Horizon. A registered type may contribute a customization screen. The framework must preserve settings across world creation and into the save, with stable mod-owned identifiers and schema versions.

## Registration

Registration is performed during the mod registration phase and frozen before world creation UI is built. Each world type has a namespaced identifier, display name/description, and optional customization provider. Duplicate IDs fail registration with a clear diagnostic. Built-in world presets remain supported and are not forced through this extension mechanism.

## Customization controls

The customization screen API must let mods declare these controls:

- **Boolean**: label, stable setting key, default value; presented as a checkbox/toggle.
- **Number**: label, stable setting key, default, minimum, maximum, optional step and integer/decimal behavior; validate and clamp/reject values at commit according to documented policy.
- **Text**: label, stable setting key, default, optional maximum length and validation; keep user input editable and validate before committing.

Each control has a stable key scoped to the mod and world-type registration. Labels are localized keys (with a readable fallback). Controls are ordered by registration order. The API must allow reading typed values from a customization context and must reject type mismatches instead of silently coercing.

## UI behavior

The Create World screen shows registered world types with built-in presets. Customize is enabled only when the selected type supplies a customization provider. The customization UI is a shared Horizon-rendered screen, not a mod-owned arbitrary UI, so all controls are consistent and accessible. Opening it loads draft values from the current creation session or defaults; Save commits the draft, Cancel discards it. Reopening retains committed draft values.

## Persistence

Persist mod settings under a namespaced custom world metadata compound, keyed by mod ID and world-type ID, with schema version. Save the selected world type and settings in the world creation result. Keep unknown entries when rewriting; missing mods do not erase data. Warn and use safe defaults for malformed or unsupported entries. Mods can provide migration callbacks for schema changes. Settings should be persisted in world metadata (and exposed in the generation context) rather than being transient UI-only state.

## Generator contract

The selected world-type registration receives a generation context containing seed, dimension, chunk coordinates, validated world-generation settings, world scale (if applicable), and its typed customization values. Built-in generation remains unchanged unless a registered custom type is selected. Diagnostics should identify the world type and settings without dumping sensitive text fields by default.

## Validation and lifecycle

Freeze registration before UI use. Require namespaced IDs and keys; reject duplicates. Number bounds must be finite and valid; defaults must fall within bounds. Text defaults and current values must obey limits and validators. All data is validated when changed and again before saving/loading. Errors should be shown to the player in the customization screen and logged with mod/setting identifiers.

## Implementation plan

1. Inspect existing native-mod registration, screen routing, world-creation state and save format.
2. Add registry types, typed control descriptors, registration and validation tests.
3. Add draft/commit/cancel state and shared boolean/number/text widgets.
4. Add selected world-type state, customization button and screen routing.
5. Persist namespaced typed values in world metadata and pass them to generation.
6. Add a sample mod and docs; test defaults, edits, cancellation, reload, migration, missing mod, malformed values, duplicate keys, and vanilla regression.

This document is design-only; implementation is not yet committed.
