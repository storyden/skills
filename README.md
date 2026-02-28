# Storyden Skills

This repository contains a set of [Agent Skills](https://agentskills.io/) for use with coding agents such as Codex, Claude Code, OpenCode and more.

## Scope

These skills help you and your coding agent build:

- Plugins
- Custom frontends/clients
- Integrations

## Available Skills

### [golang-plugin](golang-plugin/)

Build, package, and deploy Storyden plugins using Go and the official Storyden Go SDK.

**Use when:**
- Creating new Storyden plugins
- Implementing event handlers for platform events
- Requesting and using API access from plugins
- Packaging plugins for supervised runtime deployment
- Setting up local development workflow

**Features:**
- Complete plugin development workflow
- Event handler patterns and examples
- API client usage and authentication
- Manifest configuration guide
- Production packaging instructions
- Working example (reactbot)

## Usage

Using `skills` CLI tool:

```shell
npx skills add storyden/skills
```
