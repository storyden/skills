# React Bot Example

This is a complete Storyden plugin example in Go.

## What it does

React Bot automatically adds a fire emoji reaction to each newly created thread reply.

## Files

- [reactbot.go](reactbot.go) - plugin implementation
- [manifest.yaml](manifest.yaml) - plugin manifest

## Key Concepts Demonstrated

### 1. Plugin Initialization

```go
plugin, err := storyden.New(ctx)
if err != nil {
	logger.Error("failed to create plugin", slog.String("error", err.Error()))
	os.Exit(1)
}
```

### 2. Event Handler Registration

```go
plugin.OnThreadReplyCreated(bot.onThreadReplyCreated)
```

### 3. API Client Usage

```go
client, err := r.plugin.BuildAPIClient(timeoutCtx)
if err != nil {
	return fmt.Errorf("build api client: %w", err)
}

resp, err := client.PostReactAddWithResponse(timeoutCtx, postID, openapi.PostReactAddJSONRequestBody{
	Emoji: fireEmoji,
})
```

### 4. Graceful Shutdown

```go
ctx, stop := signal.NotifyContext(context.Background(), os.Interrupt)
defer stop()

defer func() {
	if err := plugin.Shutdown(); err != nil && !errors.Is(err, context.Canceled) {
		logger.Warn("plugin shutdown returned error", slog.String("error", err.Error()))
	}
}()
```

## Running This Example

### External Mode (Development)

1. Create a fresh folder and copy these files into it.
2. Initialize module and dependencies:

```bash
go mod init example.com/reactbot
go mod tidy
```

3. In Storyden Admin -> Plugins, add an **External** plugin.
4. Paste `manifest.yaml`.
5. Copy the `STORYDEN_RPC_URL=...` value from the Connection tab.
6. Run:

```bash
export STORYDEN_RPC_URL='ws://localhost:8000/rpc?token=...'
go run reactbot.go
```

### Supervised Mode (Packaging)

1. Build binary for your Storyden host OS/arch.

Linux example:

```bash
export CGO_ENABLED=0
export GOOS=linux
export GOARCH=amd64
go build -o reactbot reactbot.go
```

2. Convert manifest to JSON:

```bash
yq -o=json manifest.yaml > manifest.json
```

3. Create archive:

```bash
zip reactbot.zip manifest.json reactbot
```

4. Upload `reactbot.zip` in Storyden Admin -> Plugins as a **Supervised** plugin.

## Related Documentation

- Storyden plugin docs: https://storyden.org/docs/extending
- Go plugin tutorial: https://storyden.org/docs/extending/tutorials/go-plugin
- Manifest reference: https://storyden.org/docs/extending/manifest
- API/RPC reference: https://storyden.org/docs/extending/api
