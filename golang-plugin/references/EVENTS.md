# Storyden Platform Events Reference

Guide to event subscriptions and handlers for Storyden plugins.

## Canonical References

- Event RPC docs: https://storyden.org/docs/extending/api/host-to-plugin/event
- SDK handlers: https://github.com/Southclaws/storyden/blob/main/sdk/go/storyden/handlers.go
- Event payload structs: https://github.com/Southclaws/storyden/blob/main/lib/plugin/rpc/rpc.go

Use those as source of truth when in doubt.

## Overview

Plugins receive only the events listed in manifest `events_consumed`.

Example:

```yaml
events_consumed:
  - EventThreadReplyCreated
  - EventThreadPublished
```

If an event is not listed, it is not delivered.

## Go Handler Pattern

```go
plugin.OnThreadReplyCreated(func(ctx context.Context, event *rpc.EventThreadReplyCreated) error {
	// Handle event
	return nil
})
```

Handlers:

- receive `context.Context`
- receive a typed event payload struct
- return `error`

## Available Handler Methods (Current SDK)

From `sdk/go/storyden/handlers.go`:

- `OnThreadPublished`
- `OnThreadUnpublished`
- `OnThreadUpdated`
- `OnThreadDeleted`
- `OnThreadReplyCreated`
- `OnThreadReplyDeleted`
- `OnThreadReplyUpdated`
- `OnThreadReplyPublished`
- `OnThreadReplyUnpublished`
- `OnPostLiked`
- `OnPostUnliked`
- `OnPostReacted`
- `OnPostUnreacted`
- `OnCategoryUpdated`
- `OnCategoryDeleted`
- `OnMemberMentioned`
- `OnNodeCreated`
- `OnNodeUpdated`
- `OnNodeDeleted`
- `OnNodePublished`
- `OnNodeSubmittedForReview`
- `OnNodeUnpublished`
- `OnAccountCreated`
- `OnAccountUpdated`
- `OnAccountSuspended`
- `OnAccountUnsuspended`
- `OnReportCreated`
- `OnReportUpdated`
- `OnActivityCreated`
- `OnActivityUpdated`
- `OnActivityDeleted`
- `OnActivityPublished`
- `OnSettingsUpdated`

## Example: Thread Reply Event

```go
plugin.OnThreadReplyCreated(func(ctx context.Context, event *rpc.EventThreadReplyCreated) error {
	slog.Info("reply created",
		slog.String("reply_id", event.ReplyID.String()),
		slog.String("thread_id", event.ThreadID.String()),
	)
	return nil
})
```

`EventThreadReplyCreated` includes fields like:

- `ReplyID`
- `ThreadID`
- `ReplyAuthorID`
- `ThreadAuthorID`
- optional `ReplyToTargetID`
- optional `ReplyToAuthorID`

## Filtering Inside Handlers

Example: ignore replies made by the thread author.

```go
plugin.OnThreadReplyCreated(func(ctx context.Context, event *rpc.EventThreadReplyCreated) error {
	if event.ReplyAuthorID == event.ThreadAuthorID {
		return nil
	}
	return processReply(ctx, event)
})
```

Example: process only replies that are direct replies to another post.

```go
plugin.OnThreadReplyCreated(func(ctx context.Context, event *rpc.EventThreadReplyCreated) error {
	if _, ok := event.ReplyToTargetID.Get(); !ok {
		return nil
	}
	return processNestedReply(ctx, event)
})
```

## Reliability Guidance

Treat handlers as:

- idempotent (safe if triggered more than once)
- resilient to transient API/network errors
- quick to return when possible

For API calls, always use context timeouts.

```go
const apiTimeout = 10 * time.Second

plugin.OnThreadReplyCreated(func(ctx context.Context, event *rpc.EventThreadReplyCreated) error {
	timeoutCtx, cancel := context.WithTimeout(ctx, apiTimeout)
	defer cancel()

	client, err := plugin.BuildAPIClient(timeoutCtx)
	if err != nil {
		return err
	}

	_, err = client.PostReactAddWithResponse(timeoutCtx, openapi.PostIDParam(event.ReplyID.String()), openapi.PostReactAddJSONRequestBody{Emoji: "\U0001F525"})
	return err
})
```
