# GPUI Skills

[![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)](./skills/.claude-plugin/plugin.json)
[![License](https://img.shields.io/badge/license-Apache--2.0-green.svg)](https://github.com/zed-industries/zed)

Claude Code skills for building cross-platform desktop UI applications with the [GPUI](https://gpui.rs) framework in Rust.

## About GPUI

[GPUI](https://gpui.rs) is a high-performance UI framework developed by [Zed Industries](https://zed.dev) for building native desktop applications in Rust. Key features include:

- **High Performance**: GPU-accelerated rendering with efficient state management
- **Cross-Platform**: Build for macOS, Linux, and Windows from a single codebase
- **Type-Safe**: Leverages Rust's type system for reliable UI code
- **Modern Patterns**: Entity-based state management with async/await support
- **Tailwind-like Styling**: Familiar utility-first styling approach

## Installation

Copy the `skills` folder to `.claude/skills` in your GPUI project:

```bash
# Clone or copy this repository
git clone <repository-url> gpui-skills

# Copy to your project
cp -r gpui-skills/skills your-project/.claude/skills
```

Your project structure should look like:

```
your-project/
├── .claude/
│   └── skills/
│       ├── .claude-plugin/
│       │   └── plugin.json
│       ├── gpui-init/
│       ├── gpui-fundamentals/
│       └── ... (other skills)
├── src/
└── Cargo.toml
```

See [Claude Code Skills documentation](https://docs.anthropic.com/en/docs/claude-code/skills) for more details.

## Skills Overview

### Getting Started

| Skill | Description | When to Use |
|-------|-------------|-------------|
| [gpui-init](./skills/gpui-init/SKILL.md) | Project scaffolding and setup | "Create a new GPUI app" |

### Core Development

| Skill | Description | When to Use |
|-------|-------------|-------------|
| [gpui-fundamentals](./skills/gpui-fundamentals/SKILL.md) | Core concepts: contexts, windows, entities, elements | Learning GPUI basics, understanding architecture |
| [gpui-entities](./skills/gpui-entities/SKILL.md) | Entity lifecycle, subscriptions, state management | Managing component state, entity patterns |
| [gpui-elements](./skills/gpui-elements/SKILL.md) | UI element creation and composition | Building element trees, creating components |
| [gpui-styling](./skills/gpui-styling/SKILL.md) | Tailwind-like utility styling | Applying styles, layouts, colors |
| [gpui-actions](./skills/gpui-actions/SKILL.md) | Actions and event handling | Implementing interactions, keyboard shortcuts |
| [gpui-async](./skills/gpui-async/SKILL.md) | Async patterns and concurrency | Background tasks, async operations |
| [gpui-patterns](./skills/gpui-patterns/SKILL.md) | Common UI patterns | Modals, lists, forms, dropdowns |

### Quality & Debugging

| Skill | Description | When to Use |
|-------|-------------|-------------|
| [gpui-testing](./skills/gpui-testing/SKILL.md) | Testing GPUI applications | Writing tests, testing async code |
| [gpui-troubleshooting](./skills/gpui-troubleshooting/SKILL.md) | Common errors and solutions | Debugging build errors, runtime panics |
| [gpui-code-quality](./skills/gpui-code-quality/SKILL.md) | Best practices and guidelines | Code review, refactoring |

## Usage Examples

### Create a New Project

```
User: Create a new GPUI app with a counter button
Claude: [Uses gpui-init to scaffold project, gpui-fundamentals for structure]
```

### Implement a Feature

```
User: Add a modal dialog for settings
Claude: [Uses gpui-patterns for modal, gpui-styling for appearance]
```

### Fix an Error

```
User: Getting "already mutably borrowed" error
Claude: [Uses gpui-troubleshooting to identify nested entity update issue]
```

### Add Async Data Loading

```
User: Fetch data from an API without blocking the UI
Claude: [Uses gpui-async for background tasks, gpui-patterns for loading states]
```

## What You Can Build

With these skills, Claude can help you:

- Initialize new GPUI projects with proper structure
- Create UI elements with flexbox layouts
- Style components with Tailwind-like utilities
- Handle user interactions and keyboard shortcuts
- Manage application state with entities
- Implement async operations safely
- Build common UI patterns (modals, forms, lists)
- Write proper tests for GPUI applications
- Debug and fix common errors

## Example Applications

GPUI is used in production applications including:

- **[Zed](https://zed.dev)** - High-performance code editor
- **[Longbridge Pro](https://longbridge.com/desktop)** - Financial trading platform (built with [gpui-component](https://github.com/longbridge/gpui-component))

## Key Concepts

### Entities

Entities are GPUI's core state management primitive:

```rust
let counter = cx.new(|_| Counter { count: 0 });
counter.update(cx, |counter, cx| {
    counter.count += 1;
    cx.notify();
});
```

### Elements

Build UI with composable elements:

```rust
div()
    .flex()
    .flex_col()
    .gap_4()
    .child("Hello, GPUI!")
    .child(Button::new("Click Me"))
```

### Async Operations

Handle background work safely:

```rust
cx.spawn(async move |this, cx| {
    let data = fetch_data().await;
    this.update(&mut *cx, |view, cx| {
        view.data = data;
        cx.notify();
    })?;
    Ok(())
}).detach();
```

## Best Practices

1. **Never use `unwrap()`** - Always propagate errors with `?`
2. **Call `cx.notify()`** - After state changes that affect rendering
3. **Use `WeakEntity`** - For back-references to avoid memory leaks
4. **Use `SharedString`** - For efficient text handling
5. **Detach or store tasks** - Don't drop async tasks prematurely
6. **Use GPUI timers in tests** - Avoid `smol::Timer` in test code

## Resources

- [GPUI Documentation](https://gpui.rs)
- [Zed Repository](https://github.com/zed-industries/zed) - Source code and examples
- [Zed GEMINI.md](https://github.com/zed-industries/zed/blob/main/GEMINI.md) - GPUI development guidelines
- [gpui-component](https://github.com/longbridge/gpui-component) - Rich UI component library

## License

Apache-2.0

## Contributing

Contributions are welcome! Please feel free to submit issues or pull requests to improve these skills.
