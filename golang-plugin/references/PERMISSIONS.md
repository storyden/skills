# Storyden API Permissions Reference

Guide to API permissions for Storyden plugins.

## Canonical References

- Plugin access model: https://storyden.org/docs/extending/security#api-access-identity-model
- Manifest `access` field: https://storyden.org/docs/extending/manifest#access
- Source enum (authoritative names): https://github.com/Southclaws/storyden/blob/main/app/resources/rbac/rbac_enum_gen.go

## Overview

Plugins that call Storyden APIs must declare `access.permissions` in manifest.

```yaml
access:
  handle: my-plugin
  name: My Plugin
  permissions:
    - CREATE_REACTION
```

Missing permissions result in authorization failures (for example HTTP `403`).

## Current Permission Names

From `app/resources/rbac/rbac_enum_gen.go`:

- `CREATE_POST`
- `READ_PUBLISHED_THREADS`
- `CREATE_REACTION`
- `MANAGE_POSTS`
- `MANAGE_CATEGORIES`
- `CREATE_INVITATION`
- `READ_PUBLISHED_LIBRARY`
- `MANAGE_LIBRARY`
- `SUBMIT_LIBRARY_NODE`
- `UPLOAD_ASSET`
- `MANAGE_EVENTS`
- `LIST_PROFILES`
- `READ_PROFILE`
- `CREATE_COLLECTION`
- `LIST_COLLECTIONS`
- `READ_COLLECTION`
- `MANAGE_COLLECTIONS`
- `COLLECTION_SUBMIT`
- `USE_PERSONAL_ACCESS_KEYS`
- `MANAGE_SETTINGS`
- `MANAGE_SUSPENSIONS`
- `MANAGE_ROLES`
- `MANAGE_REPORTS`
- `VIEW_ACCOUNTS`
- `ADMINISTRATOR`

## Common Permission Sets

### Reaction bot

```yaml
access:
  permissions:
    - CREATE_REACTION
```

### Reply/post bot

```yaml
access:
  permissions:
    - CREATE_POST
    - READ_PUBLISHED_THREADS
```

### Library automation

```yaml
access:
  permissions:
    - READ_PUBLISHED_LIBRARY
    - MANAGE_LIBRARY
```

### Profile lookup bot

```yaml
access:
  permissions:
    - LIST_PROFILES
    - READ_PROFILE
```

## Best Practices

1. Request least privilege only.
2. Keep permission list aligned with actual API calls.
3. Handle `403` responses explicitly in code.
4. Document each requested permission in your plugin README.

Example error handling:

```go
resp, err := client.PostReactAddWithResponse(ctx, postID, body)
if err != nil {
	return fmt.Errorf("api call failed: %w", err)
}

if resp.StatusCode() == 403 {
	return fmt.Errorf("missing CREATE_REACTION permission")
}
```
