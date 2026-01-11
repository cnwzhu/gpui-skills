---
name: gpui-elements
description: UI element creation and composition with GPUI. Use when building element trees, creating custom components, implementing layouts, or working with conditional rendering.
---

# GPUI Elements

This skill covers creating and composing UI elements in GPUI applications.

## Overview

Elements are the building blocks of GPUI UIs:
- Built with `div()` as the base element
- Styled with chainable methods
- Composed with `.child()` and `.children()`
- Support conditional rendering with `.when()`
- Use flexbox for layout

## Basic Element Creation

### div() Element

```rust
use gpui::*;

impl Render for MyView {
    fn render(&mut self, _window: &mut Window, _cx: &mut Context<Self>) -> impl IntoElement {
        div()
            .child("Hello, GPUI!")
    }
}
```

### Adding Children

```rust
div()
    .child("First child")
    .child(
        div()
            .child("Nested child")
    )
    .child("Third child")
```

### Multiple Children with Vector

```rust
let items = vec!["One", "Two", "Three"];

div().children(
    items.into_iter().map(|text| {
        div().child(text)
    })
)
```

## Text Elements

### String Types

```rust
use gpui::*;

impl Render for MyView {
    fn render(&mut self, _window: &mut Window, _cx: &mut Context<Self>) -> impl IntoElement {
        div()
            // &str
            .child("Static text")
            // String
            .child(format!("Count: {}", self.count))
            // SharedString (most efficient)
            .child(self.title.clone())
    }
}
```

### SharedString

```rust
use gpui::*;

struct MyView {
    title: SharedString,
}

impl MyView {
    fn new() -> Self {
        Self {
            title: "Default Title".into(),
        }
    }
    
    fn set_title(&mut self, title: impl Into<SharedString>) {
        self.title = title.into();
    }
}

// SharedString implements IntoElement
impl Render for MyView {
    fn render(&mut self, _window: &mut Window, _cx: &mut Context<Self>) -> impl IntoElement {
        div().child(self.title.clone())
    }
}
```

## Conditional Rendering

### when() Method

```rust
div()
    .when(condition, |this| {
        this.bg(rgb(0x3b82f6))
            .child("Condition is true")
    })
```

### when_some() for Options

```rust
div()
    .when_some(maybe_text, |this, text| {
        this.child(text)
    })
```

### Practical Example

```rust
struct MyView {
    is_active: bool,
    error: Option<String>,
}

impl Render for MyView {
    fn render(&mut self, _window: &mut Window, _cx: &mut Context<Self>) -> impl IntoElement {
        div()
            .when(self.is_active, |this| {
                this.bg(rgb(0x22c55e))
            })
            .when_some(self.error.clone(), |this, error| {
                this.child(
                    div()
                        .p_2()
                        .bg(rgb(0xef4444))
                        .text_color(rgb(0xffffff))
                        .child(error)
                )
            })
    }
}
```

## Custom Components

### RenderOnce Components

```rust
use gpui::*;

#[derive(IntoElement)]
struct Button {
    label: SharedString,
    on_click: Option<Box<dyn Fn(&mut App) + 'static>>,
}

impl Button {
    fn new(label: impl Into<SharedString>) -> Self {
        Self {
            label: label.into(),
            on_click: None,
        }
    }
    
    fn on_click(mut self, handler: impl Fn(&mut App) + 'static) -> Self {
        self.on_click = Some(Box::new(handler));
        self
    }
}

impl RenderOnce for Button {
    fn render(self, _window: &mut Window, _cx: &mut App) -> impl IntoElement {
        let mut element = div()
            .px_4()
            .py_2()
            .bg(rgb(0x3b82f6))
            .text_color(rgb(0xffffff))
            .rounded(px(6.0))
            .cursor_pointer()
            .child(self.label);
        
        if let Some(handler) = self.on_click {
            element = element.on_click(move |_event, _window, cx| {
                handler(cx);
            });
        }
        
        element
    }
}

// Usage
impl Render for MyView {
    fn render(&mut self, _window: &mut Window, _cx: &mut Context<Self>) -> impl IntoElement {
        div().child(
            Button::new("Click Me")
                .on_click(|cx| {
                    println!("Button clicked!");
                })
        )
    }
}
```

### Render Components

```rust
struct Card {
    title: SharedString,
    content: SharedString,
}

impl Render for Card {
    fn render(&mut self, _window: &mut Window, _cx: &mut Context<Self>) -> impl IntoElement {
        div()
            .p_4()
            .bg(rgb(0x1a1a1a))
            .rounded(px(8.0))
            .flex()
            .flex_col()
            .gap_2()
            .child(
                div()
                    .text_lg()
                    .font_bold()
                    .child(self.title.clone())
            )
            .child(
                div()
                    .text_sm()
                    .child(self.content.clone())
            )
    }
}
```

## Layout Patterns

### Vertical Stack

```rust
div()
    .flex()
    .flex_col()
    .gap_4()
    .child("Item 1")
    .child("Item 2")
    .child("Item 3")
```

### Horizontal Row

```rust
div()
    .flex()
    .flex_row()
    .gap_4()
    .items_center()
    .child("Left")
    .child("Center")
    .child("Right")
```

### Grid-like Layout

```rust
div()
    .flex()
    .flex_wrap()
    .gap_4()
    .children((0..12).map(|i| {
        div()
            .w(px(100.0))
            .h(px(100.0))
            .bg(rgb(0x3b82f6))
            .child(format!("Item {}", i))
    }))
```

### Centering

```rust
div()
    .flex()
    .items_center()
    .justify_center()
    .size_full()
    .child("Centered content")
```

## Dynamic Lists

### Simple List

```rust
struct ListView {
    items: Vec<String>,
}

impl Render for ListView {
    fn render(&mut self, _window: &mut Window, _cx: &mut Context<Self>) -> impl IntoElement {
        div()
            .flex()
            .flex_col()
            .gap_2()
            .children(
                self.items.iter().map(|item| {
                    div()
                        .p_2()
                        .bg(rgb(0x1a1a1a))
                        .rounded(px(4.0))
                        .child(item.clone())
                })
            )
    }
}
```

### List with Index

```rust
div()
    .children(
        self.items.iter().enumerate().map(|(i, item)| {
            div()
                .child(format!("{}. {}", i + 1, item))
        })
    )
```

## Nested Components

### Component Composition

```rust
struct Header;
impl Render for Header {
    fn render(&mut self, _window: &mut Window, _cx: &mut Context<Self>) -> impl IntoElement {
        div()
            .h(px(64.0))
            .bg(rgb(0x1a1a1a))
            .child("Header")
    }
}

struct Sidebar;
impl Render for Sidebar {
    fn render(&mut self, _window: &mut Window, _cx: &mut Context<Self>) -> impl IntoElement {
        div()
            .w(px(250.0))
            .bg(rgb(0x2a2a2a))
            .child("Sidebar")
    }
}

struct AppLayout {
    header: Entity<Header>,
    sidebar: Entity<Sidebar>,
}

impl Render for AppLayout {
    fn render(&mut self, _window: &mut Window, _cx: &mut Context<Self>) -> impl IntoElement {
        div()
            .size_full()
            .flex()
            .flex_col()
            .child(self.header.clone())
            .child(
                div()
                    .flex()
                    .flex_1()
                    .child(self.sidebar.clone())
                    .child(
                        div()
                            .flex_1()
                            .child("Main content")
                    )
            )
    }
}
```

## Element Patterns

### Clickable Card

```rust
div()
    .p_4()
    .bg(rgb(0x1a1a1a))
    .rounded(px(8.0))
    .cursor_pointer()
    .hover(|style| style.bg(rgb(0x2a2a2a)))
    .on_click(|_event, _window, cx| {
        println!("Card clicked");
    })
    .child("Click me")
```

### Input Field

```rust
struct InputField {
    value: SharedString,
}

impl Render for InputField {
    fn render(&mut self, _window: &mut Window, cx: &mut Context<Self>) -> impl IntoElement {
        div()
            .px_3()
            .py_2()
            .bg(rgb(0x1a1a1a))
            .border_1()
            .border_color(rgb(0x4a4a4a))
            .rounded(px(4.0))
            .child(self.value.clone())
            .on_click(cx.listener(|this, _event, _window, cx| {
                // Handle focus
            }))
    }
}
```

### Toggle Switch

```rust
struct Toggle {
    enabled: bool,
}

impl Render for Toggle {
    fn render(&mut self, _window: &mut Window, cx: &mut Context<Self>) -> impl IntoElement {
        div()
            .w(px(44.0))
            .h(px(24.0))
            .rounded(px(12.0))
            .bg(if self.enabled { 
                rgb(0x22c55e) 
            } else { 
                rgb(0x4a4a4a) 
            })
            .cursor_pointer()
            .on_click(cx.listener(|this, _event, _window, cx| {
                this.enabled = !this.enabled;
                cx.notify();
            }))
    }
}
```

## Best Practices

1. **Use SharedString for text**: Avoids unnecessary string copies
2. **Use .when() for conditions**: More readable than if/else in render
3. **Extract reusable components**: Create RenderOnce components for common patterns
4. **Keep render pure**: Don't mutate state directly in render() method
5. **Use children() for lists**: More efficient than multiple .child() calls
6. **Chain methods**: Build elements with method chaining for clarity

## Common Mistakes

| Mistake | Problem | Solution |
|---------|---------|----------|
| Mutating state in render | Race conditions | Use event handlers to mutate |
| Not using SharedString | Extra allocations | Use `SharedString` for text |
| Deep nesting | Hard to read | Extract into components |
| Missing IntoElement | Compile error | Ensure types implement `IntoElement` |
| Forgetting .clone() | Borrow errors | Clone `SharedString` when using multiple times |

## Summary

- Use `div()` as the base element
- Add children with `.child()` or `.children()`
- Use `.when()` for conditional rendering
- Create components with `RenderOnce` or `Render`
- Use `SharedString` for efficient text handling
- Compose with flexbox layout methods

## References

- [GPUI Elements Documentation](https://gpui.rs)
- [gpui-component Library](https://github.com/longbridge/gpui-component)
