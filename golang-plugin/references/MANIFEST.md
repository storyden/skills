# Plugin Manifest Reference

Reference for Storyden plugin manifest files.

Canonical docs:

- https://storyden.org/docs/extending/manifest
- https://storyden.org/docs/extending/model
- https://storyden.org/docs/extending/security

## Overview

Every plugin must provide a manifest (`manifest.yaml` or `manifest.json`) describing:

- Plugin identity (`id`, `name`, `author`, `version`)
- Runtime entrypoint (`command`, optional `args`)
- Event subscriptions (`events_consumed`)
- Optional API identity request (`access`)
- Optional runtime config schema (`configuration_schema`)

## Required Fields

### id

- Type: `string`
- Pattern: `^[a-zA-Z0-9](?:[a-zA-Z0-9-]*[a-zA-Z0-9])?$`
- Example: `my-plugin`

Unique plugin identifier.

### name

- Type: `string`
- Example: `My Plugin`

Display name shown in UI.

### author

- Type: `string`
- Pattern: `^[a-zA-Z0-9](?:[a-zA-Z0-9-]*[a-zA-Z0-9])?$`
- Example: `your-name`

Plugin author label.

### description

- Type: `string`
- Example: `Reacts with a fire emoji to new replies.`

Human-readable plugin description.

### version

- Type: `string`
- Example: `1.0.0`

Display version.

### command

- Type: `string`
- Examples: `./myplugin`, `./myplugin.exe`, `python`, `node`

Runtime entrypoint used by supervised plugins.

Examples:

```yaml
# Linux/macOS host
command: "./myplugin"
```

```yaml
# Windows host
command: "./myplugin.exe"
```

```yaml
# Interpreter-based plugin
command: "python"
args: ["main.py"]
```

## Optional Fields

### args

- Type: `array<string>`

Arguments passed to `command`.

### events_consumed

- Type: `array<string>`
- Example:

```yaml
events_consumed:
  - EventThreadReplyCreated
  - EventThreadPublished
```

Only listed events are delivered.

See:

- https://storyden.org/docs/extending/api/host-to-plugin/event
- [EVENTS.md](./EVENTS.md)

### access

Optional API access request. When present, Storyden can provision a plugin account + access key.

#### access.handle

- Type: `string`
- Required when `access` is present
- Example: `my-plugin-bot`

#### access.name

- Type: `string`
- Required when `access` is present
- Example: `My Plugin Bot`

#### access.permissions

- Type: `array<string>`
- Required when `access` is present

Example:

```yaml
access:
  handle: reactbot
  name: React Bot
  permissions:
    - CREATE_REACTION
```

Use valid permission names only. See [PERMISSIONS.md](./PERMISSIONS.md).

#### access.bio / access.links / access.metadata

Optional profile metadata for the provisioned account.

### configuration_schema

Defines admin-editable runtime config fields.

```yaml
configuration_schema:
  fields:
    - id: webhook_url
      label: Webhook URL
      description: Where to send events
      type: string
    - id: enabled
      label: Enabled
      description: Toggle behavior
      type: boolean
    - id: retry_count
      label: Retry count
      description: Retry attempts for failed requests
      type: number
```

Supported field types are currently:

- `string`
- `number`
- `boolean`

## Minimal Example

```yaml
id: reactbot
name: React Bot
author: you
description: React to every new thread reply.
version: 1.0.0
command: "./reactbot"
events_consumed:
  - EventThreadReplyCreated
access:
  handle: reactbot
  name: React Bot
  permissions:
    - CREATE_REACTION
```
