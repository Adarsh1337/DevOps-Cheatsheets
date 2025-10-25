# 🧭 K9s Cheatsheet

[⬅️ Back to Root](../../README.md)

## Core Concepts
- TUI for Kubernetes: navigate resources with keyboard
- Contexts/namespaces, pod logs, exec, port-forward
- Filters, labels, favorites, hotkeys

## Launch & Context
```bash
k9s          # current context
k9s --context myctx --namespace dev
```

## Navigation Basics
- : to open command prompt
- 0-9 to switch views (pods, deployments, services, etc.)
- / to filter, Ctrl-a to clear filter
- Arrow keys / j k to move, Enter to drill in
- Esc to go back, ? for help

## Common Shortcuts
- g: goto favorites
- s: switch context/namespace
- l: logs (L for multi)
- d: describe
- e: edit manifest (opens editor)
- x: exec shell
- f: port-forward
- r: restart (rollout)
- y: view YAML
- Shift-0..9: view hotkeys

## Pod Actions
- x: exec into container
- l: tail logs, Shift-l for previous logs
- p: show ports, f to forward (e.g., 8080:80)
- t: top (metrics)

## Filters
- /key=value to label-filter
- /namepart to fuzzy match names

## Best Practices
- Use read-only kubeconfigs in production
- Limit edit actions to least-privileged contexts
- Customize skin and hotkeys in ~/.k9s
