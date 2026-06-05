---
name: golang-plugin
description: Build, package, and deploy Storyden plugins using Go. Use when creating new Storyden plugins, implementing event handlers, requesting API access, or packaging plugins for distribution.
metadata:
  author: southclaws
compatibility: Requires Go 1.24+, access to a Storyden instance, and Administrator permissions in Storyden
---

# Storyden Go Plugin Development

This skill helps you build, test, and deploy Storyden plugins using Go and the official Storyden Go SDK.

## When to use this skill

Use this skill when you need to:

- Create a new Storyden plugin from scratch
- Implement event handlers for Storyden platform events
- Request and use API access from within a plugin
- Package plugins for supervised runtime deployment
- Set up local development workflow for plugin development
- Define plugin manifests with correct capabilities and permissions

## Prerequisites

Before starting, ensure you have:

1. Go 1.24+ installed
2. Access to a running Storyden instance (local or remote)
3. Admin access to the Storyden instance (Admin -> Plugins)
4. Basic Go knowledge
5. The `sd` CLI installed and authenticated (see storyden-cli skill)

The `sd` CLI (`sd plugin --help`) is the primary tool for scaffolding, running, packaging, installing, and managing plugins. Always consult the installed CLI for authoritative flag and argument docs:

```bash
sd plugin --help
sd plugin dev --help
sd plugin dev run --help
```

## Quick Start: Build a Simple Plugin

This quick start builds a minimal plugin that reacts to new thread replies with a fire emoji.

### Step 1: Set up a new plugin project

Scaffold with the `sd` CLI (creates `manifest.yaml` pre-filled with sane defaults):

```bash
sd plugin dev new reactbot
cd reactbot
go mod init example.com/reactbot
```

Or start from a fresh, empty directory manually:

```bash
mkdir reactbot
cd reactbot
go mod init example.com/reactbot
```

### Step 2: Create the manifest

Create `manifest.yaml`:

```yaml
id: reactbot
name: React Bot
author: your-name
description: React to every new thread reply with a fire emoji.
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

If your Storyden host is Windows, use `command: "./reactbot.exe"`.

### Step 3: Implement the plugin

Create `main.go`:

```go
package main

import (
	"context"
	"errors"
	"fmt"
	"log/slog"
	"os"
	"os/signal"
	"time"

	"github.com/Southclaws/storyden/app/transports/http/openapi"
	"github.com/Southclaws/storyden/lib/plugin/rpc"
	"github.com/Southclaws/storyden/sdk/go/storyden"
)

const (
	fireEmoji      = "\U0001F525"
	apiCallTimeout = 10 * time.Second
)

func main() {
	logger := slog.New(slog.NewTextHandler(os.Stdout, &slog.HandlerOptions{Level: slog.LevelInfo}))
	slog.SetDefault(logger)

	ctx, stop := signal.NotifyContext(context.Background(), os.Interrupt)
	defer stop()

	plugin, err := storyden.New(ctx)
	if err != nil {
		logger.Error("failed to create plugin", slog.String("error", err.Error()))
		os.Exit(1)
	}
	defer func() {
		if err := plugin.Shutdown(); err != nil && !errors.Is(err, context.Canceled) {
			logger.Warn("plugin shutdown returned error", slog.String("error", err.Error()))
		}
	}()

	// Register event handlers before starting the runtime loop.
	plugin.OnThreadReplyCreated(func(ctx context.Context, event *rpc.EventThreadReplyCreated) error {
		timeoutCtx, cancel := context.WithTimeout(ctx, apiCallTimeout)
		defer cancel()

		client, err := plugin.BuildAPIClient(timeoutCtx)
		if err != nil {
			return fmt.Errorf("build api client: %w", err)
		}

		postID := openapi.PostIDParam(event.ReplyID.String())
		resp, err := client.PostReactAddWithResponse(timeoutCtx, postID, openapi.PostReactAddJSONRequestBody{
			Emoji: fireEmoji,
		})
		if err != nil {
			return fmt.Errorf("create reaction: %w", err)
		}
		if resp.StatusCode() != 200 {
			return fmt.Errorf("create reaction returned status %d", resp.StatusCode())
		}

		return nil
	})

	// Run connects to Storyden RPC and starts handling incoming messages/events.
	if err := plugin.Run(ctx); err != nil {
		if errors.Is(err, context.Canceled) {
			return
		}
		logger.Error("plugin stopped", slog.String("error", err.Error()))
		os.Exit(1)
	}
}
```

Run `go mod tidy` once after writing your code:

```bash
go mod tidy
```

### Step 4: Run as external plugin (development)

`sd plugin dev run` handles external plugin registration and injects `STORYDEN_RPC_URL` automatically — no manual UI steps required:

```bash
sd plugin dev run
```

It reads `manifest.yaml`, registers or updates the plugin as an external plugin on the current instance, then executes the manifest's `command` with `STORYDEN_RPC_URL` set.

To override the manifest command (e.g. run with `go run .` during development):

```bash
sd plugin dev run -- go run .
```

Useful flags:
- `--manifest/-m <path>` — path to manifest YAML (default: `manifest.yaml`)
- `--no-update` — skip updating the existing external plugin manifest before running
- `--instance-id <id>` — target a specific existing plugin installation

### Step 5: Package and install as supervised plugin

**Validate** the manifest and package structure before distributing:

```bash
sd plugin dev validate
```

**Install directly** to the connected Storyden instance (builds the package and uploads it in one step):

```bash
sd plugin dev install
```

If a supervised plugin with the same manifest `id` already exists it is updated in place; otherwise a new one is created. Use `--dir` if your project is not in the current directory.

To produce a portable zip for manual distribution instead:

```bash
sd plugin dev package
# or specify an output path:
sd plugin dev package --output reactbot.zip
```

The zip contains `manifest.json` (converted from `manifest.yaml`) and the binary. If you need to cross-compile first:

```bash
export CGO_ENABLED=0
export GOOS=linux
export GOARCH=amd64
go build -o reactbot main.go
sd plugin dev package --output reactbot.zip
```

If target host is Windows, build `reactbot.exe` and set manifest `command` to `./reactbot.exe`.

## Managing Installed Plugins with sd

All plugin management commands use a `<plugin-instance-id>` which is the UUID assigned when the plugin is registered. Use `sd plugin list` or `sd plugin get` to find it.

```bash
# List all plugins on the current instance
sd plugin list
sd plugin list --format json
sd plugin list --wide          # includes version and description columns

# Inspect a specific plugin
sd plugin get <plugin-instance-id>

# Start / stop supervised plugins
sd plugin activate <plugin-instance-id>
sd plugin deactivate <plugin-instance-id>

# Stream live logs from a supervised plugin
sd plugin logs <plugin-instance-id>

# Delete a plugin
sd plugin delete <plugin-instance-id>

# Rotate the static RPC token for an external plugin
sd plugin token rotate <plugin-instance-id>
```

> Always run `sd plugin --help` or `sd plugin <subcommand> --help` to get the version-matched docs for your installed CLI.

## Event Handlers

Event handler methods are generated on `*storyden.Plugin` (for example `OnThreadReplyCreated`, `OnNodeCreated`, `OnReportUpdated`).

Canonical references:

- Handler methods: https://github.com/Southclaws/storyden/blob/main/sdk/go/storyden/handlers.go
- Event payload types: https://github.com/Southclaws/storyden/blob/main/lib/plugin/rpc/rpc.go
- RPC event docs: https://storyden.org/docs/extending/api/host-to-plugin/event

## Configuration Schema (Optional)

Plugins can define runtime configuration that admins edit in the UI:

```yaml
configuration_schema:
  fields:
    - id: webhook_url
      label: Webhook URL
      description: Where to send event notifications
      type: string
    - id: enabled
      label: Enable notifications
      description: Toggle notification sending
      type: boolean
    - id: retry_count
      label: Retry attempts
      description: Number of retries for failed requests
      type: number
```

## Common Mistakes to Avoid

- Requesting permissions you do not use in `access.permissions`
- Using non-existent permission names (check references/PERMISSIONS.md)
- Listing events in `events_consumed` that your SDK handler code does not implement
- Forgetting `manifest.json` in supervised plugin archives
- Assuming external RPC URLs include `plugin_id` (external mode uses token-based URL)

## Documentation

- Plugin guide: https://storyden.org/docs/extending
- Go plugin tutorial: https://storyden.org/docs/extending/tutorials/go-plugin
- Manifest reference: https://storyden.org/docs/extending/manifest
- API and RPC overview: https://storyden.org/docs/extending/api
- Security model: https://storyden.org/docs/extending/security
