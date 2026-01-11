---
name: gpui-init
description: Initialize new GPUI application projects with proper structure and boilerplate. Use when creating a new GPUI app, setting up project structure, or scaffolding a cross-platform Rust desktop UI application.
---

# GPUI Project Initialization

This skill enables you to scaffold new GPUI application projects with proper structure, dependencies, and boilerplate code.

## Overview

When creating a new GPUI project, generate the appropriate files based on project complexity:
- **Simple app**: Minimal structure for learning/prototyping (single file)
- **Production app**: Full structure with modules and components

## Simple App Template

### Directory Structure

```
my_app/
├── Cargo.toml
├── src/
│   └── main.rs
└── .gitignore
```

### Cargo.toml

```toml
[package]
name = "my-app"
version = "0.1.0"
edition = "2021"
description = "A GPUI application"
license = "MIT"

[dependencies]
gpui = "0.2.2"
```

### src/main.rs

```rust
use gpui::*;

struct HelloWorld {
    counter: usize,
}

impl Render for HelloWorld {
    fn render(&mut self, _window: &mut Window, _cx: &mut Context<Self>) -> impl IntoElement {
        div()
            .flex()
            .flex_col()
            .gap_4()
            .size_full()
            .items_center()
            .justify_center()
            .child(format!("Counter: {}", self.counter))
            .child(
                div()
                    .px_4()
                    .py_2()
                    .bg(rgb(0x3b82f6))
                    .text_color(rgb(0xffffff))
                    .rounded(px(4.0))
                    .child("Click Me")
                    .on_click(|_event, _window, cx| {
                        println!("Clicked!");
                    })
            )
    }
}

fn main() {
    let app = Application::new();
    
    app.run(move |cx| {
        cx.spawn(async move |cx| {
            cx.open_window(WindowOptions::default(), |window, cx| {
                cx.new(|_cx| HelloWorld { counter: 0 })
            })?;
            
            Ok::<_, anyhow::Error>(())
        })
        .detach();
    });
}
```

### .gitignore

```
/target
.DS_Store
*.swp
*.swo
.idea/
.vscode/
Cargo.lock
```

## Production App Template

### Directory Structure

```
my_app/
├── Cargo.toml
├── src/
│   ├── main.rs
│   ├── app.rs
│   ├── ui/
│   │   ├── mod.rs
│   │   └── home.rs
│   └── shared/
│       ├── mod.rs
│       └── theme.rs
├── .gitignore
└── rust-toolchain.toml
```

### Cargo.toml (Production)

```toml
[package]
name = "my-app"
version = "0.1.0"
edition = "2021"
description = "A cross-platform GPUI application"
license = "MIT"
authors = ["Your Name <your@email.com>"]

[dependencies]
gpui = "0.2.2"
anyhow = "1.0"

[features]
default = []

[profile.release]
opt-level = 3
lto = "thin"
```

### src/main.rs (Production)

```rust
mod app;
mod ui;
mod shared;

fn main() {
    app::run();
}
```

### src/app.rs

```rust
use gpui::*;
use crate::ui::home::HomeView;

pub fn run() {
    let app = Application::new();
    
    app.run(move |cx| {
        cx.spawn(async move |cx| {
            cx.open_window(
                WindowOptions {
                    window_bounds: Some(WindowBounds::Windowed(Bounds {
                        origin: Point { x: px(100.0), y: px(100.0) },
                        size: Size { width: px(1024.0), height: px(768.0) },
                    })),
                    titlebar: Some(TitlebarOptions {
                        title: Some("My App".into()),
                        appears_transparent: false,
                        ..Default::default()
                    }),
                    ..Default::default()
                },
                |window, cx| {
                    cx.new(|_cx| HomeView::new())
                }
            )?;
            
            Ok::<_, anyhow::Error>(())
        })
        .detach();
    });
}
```

### src/ui/mod.rs

```rust
pub mod home;
```

### src/ui/home.rs

```rust
use gpui::*;
use crate::shared::theme;

pub struct HomeView {
    counter: usize,
}

impl HomeView {
    pub fn new() -> Self {
        Self { counter: 0 }
    }
    
    fn increment(&mut self, _window: &mut Window, cx: &mut Context<Self>) {
        self.counter += 1;
        cx.notify();
    }
}

impl Render for HomeView {
    fn render(&mut self, _window: &mut Window, cx: &mut Context<Self>) -> impl IntoElement {
        div()
            .flex()
            .flex_col()
            .gap_6()
            .size_full()
            .p_8()
            .bg(theme::BG_COLOR)
            .child(
                div()
                    .text_2xl()
                    .font_bold()
                    .text_color(theme::TEXT_COLOR)
                    .child("Welcome to GPUI")
            )
            .child(
                div()
                    .flex()
                    .flex_col()
                    .gap_4()
                    .p_6()
                    .bg(theme::CARD_BG)
                    .rounded(px(8.0))
                    .child(
                        div()
                            .text_lg()
                            .text_color(theme::TEXT_COLOR)
                            .child(format!("Counter: {}", self.counter))
                    )
                    .child(
                        div()
                            .px_6()
                            .py_3()
                            .bg(theme::PRIMARY_COLOR)
                            .text_color(rgb(0xffffff))
                            .rounded(px(6.0))
                            .cursor_pointer()
                            .child("Increment")
                            .on_click(cx.listener(Self::increment))
                    )
            )
    }
}
```

### src/shared/mod.rs

```rust
pub mod theme;
```

### src/shared/theme.rs

```rust
use gpui::*;

pub const BG_COLOR: Hsla = hsla(0.0, 0.0, 0.05, 1.0);
pub const CARD_BG: Hsla = hsla(0.0, 0.0, 0.1, 1.0);
pub const TEXT_COLOR: Hsla = hsla(0.0, 0.0, 0.9, 1.0);
pub const PRIMARY_COLOR: Hsla = hsla(0.6, 0.8, 0.5, 1.0);
```

### rust-toolchain.toml

```toml
[toolchain]
channel = "stable"
```

## Using gpui-component Library (Optional)

> [!NOTE]
> **gpui-component is optional**. All GPUI patterns can be implemented with pure GPUI code. This library provides pre-built components similar to shadcn/ui for React.

For production apps with ready-made UI components, you can optionally add gpui-component:

```toml
[dependencies]
gpui = "0.2.2"
gpui-component = "0.4.0"
```

Example with gpui-component:

```rust
use gpui::*;
use gpui_component::{button::*, *};

pub struct Example;

impl Render for Example {
    fn render(&mut self, _: &mut Window, _: &mut Context<Self>) -> impl IntoElement {
        div()
            .v_flex()
            .gap_2()
            .size_full()
            .items_center()
            .justify_center()
            .child("Hello, GPUI!")
            .child(
                Button::new("ok")
                    .primary()
                    .label("Let's Go!")
                    .on_click(|_, _, _| println!("Clicked!"))
            )
    }
}

fn main() {
    let app = Application::new();
    
    app.run(move |cx| {
        gpui_component::init(cx);
        
        cx.spawn(async move |cx| {
            cx.open_window(WindowOptions::default(), |window, cx| {
                let view = cx.new(|_| Example);
                cx.new(|cx| Root::new(view, window, cx))
            })?;
            
            Ok::<_, anyhow::Error>(())
        })
        .detach();
    });
}
```

```

## Component Gallery Template (Optional)

> [!NOTE]
> This template uses **gpui-component** for convenience. For a pure GPUI implementation, see the "Pure GPUI Gallery" section below.

For building a component showcase (like Storybook), this pattern is based on [gpui-component's story system](https://github.com/longbridge/gpui-component/tree/main/crates/story).

### Directory Structure

```
component-gallery/
├── Cargo.toml
├── src/
│   ├── main.rs
│   ├── gallery.rs
│   └── stories/
│       ├── mod.rs
│       ├── button_story.rs
│       └── input_story.rs
```

### Cargo.toml

```toml
[package]
name = "component-gallery"
version = "0.1.0"
edition = "2021"

[dependencies]
gpui = "0.2.2"
gpui-component = "0.4.0"
```

### Story Trait Pattern

Define a trait for all component stories:

```rust
// src/stories/mod.rs
use gpui::*;

pub mod button_story;
pub mod input_story;

/// Story trait that all component demos implement
pub trait Story: Render + 'static {
    /// Title displayed in gallery
    fn title() -> &'static str;
    
    /// Description of the component
    fn description() -> &'static str;
    
    /// Create a new view for this story
    fn new_view(window: &mut Window, cx: &mut App) -> Entity<impl Render>;
}

/// Container for managing story instances
pub struct StoryContainer {
    pub name: SharedString,
    pub description: SharedString,
    view: Box<dyn Fn(&mut Window, &mut App) -> AnyElement>,
}

impl StoryContainer {
    pub fn panel<T: Story>(window: &mut Window, cx: &mut App) -> Entity<Self> {
        let name = T::title().into();
        let description = T::description().into();
        
        cx.new(|_| Self {
            name,
            description,
            view: Box::new(|window, cx| {
                T::new_view(window, cx).into_any_element()
            }),
        })
    }
}

impl Render for StoryContainer {
    fn render(&mut self, window: &mut Window, cx: &mut Context<Self>) -> impl IntoElement {
        (self.view)(window, cx)
    }
}
```

### Example Story Implementation

```rust
// src/stories/button_story.rs
use super::Story;
use gpui::*;
use gpui_component::button::*;

pub struct ButtonStory {
    disabled: bool,
    loading: bool,
}

impl Story for ButtonStory {
    fn title() -> &'static str {
        "Button"
    }
    
    fn description() -> &'static str {
        "Displays a button or a component that looks like a button."
    }
    
    fn new_view(_: &mut Window, cx: &mut App) -> Entity<impl Render> {
        cx.new(|_| Self {
            disabled: false,
            loading: false,
        })
    }
}

impl Render for ButtonStory {
    fn render(&mut self, _: &mut Window, cx: &mut Context<Self>) -> impl IntoElement {
        div()
            .v_flex()
            .gap_6()
            .p_6()
            .child(
                // Control panel
                div()
                    .h_flex()
                    .gap_3()
                    .child(
                        Checkbox::new("disabled")
                            .label("Disabled")
                            .checked(self.disabled)
                            .on_click(cx.listener(|this, _, _, cx| {
                                this.disabled = !this.disabled;
                                cx.notify();
                            }))
                    )
                    .child(
                        Checkbox::new("loading")
                            .label("Loading")
                            .checked(self.loading)
                            .on_click(cx.listener(|this, _, _, cx| {
                                this.loading = !this.loading;
                                cx.notify();
                            }))
                    )
            )
            .child(
                // Demo section
                div()
                    .v_flex()
                    .gap_4()
                    .child(div().text_lg().child("Primary Buttons"))
                    .child(
                        div()
                            .h_flex()
                            .gap_2()
                            .child(
                                Button::new("primary")
                                    .primary()
                                    .label("Primary")
                                    .disabled(self.disabled)
                                    .loading(self.loading)
                            )
                            .child(
                                Button::new("secondary")
                                    .label("Secondary")
                                    .disabled(self.disabled)
                                    .loading(self.loading)
                            )
                            .child(
                                Button::new("danger")
                                    .danger()
                                    .label("Danger")
                                    .disabled(self.disabled)
                                    .loading(self.loading)
                            )
                    )
            )
    }
}
```

### Gallery Implementation

```rust
// src/gallery.rs
use gpui::*;
use gpui_component::{Sidebar, SidebarMenu, SidebarMenuItem};
use crate::stories::*;

pub struct Gallery {
    stories: Vec<(&'static str, Vec<Entity<StoryContainer>>)>,
    active_group_index: Option<usize>,
    active_index: Option<usize>,
}

impl Gallery {
    pub fn new(window: &mut Window, cx: &mut Context<Self>) -> Self {
        let stories = vec![
            (
                "Components",
                vec![
                    StoryContainer::panel::<button_story::ButtonStory>(window, cx),
                    StoryContainer::panel::<input_story::InputStory>(window, cx),
                ],
            ),
        ];
        
        Self {
            stories,
            active_group_index: Some(0),
            active_index: Some(0),
        }
    }
}

impl Render for Gallery {
    fn render(&mut self, _: &mut Window, cx: &mut Context<Self>) -> impl IntoElement {
        let active_story = self.active_group_index
            .and_then(|group_ix| self.stories.get(group_ix))
            .and_then(|group| self.active_index.and_then(|ix| group.1.get(ix)));
        
        div()
            .flex()
            .size_full()
            .child(
                // Sidebar
                Sidebar::left()
                    .w(px(250.0))
                    .children(
                        self.stories.iter().enumerate().map(|(group_ix, (name, items))| {
                            SidebarGroup::new(*name)
                                .child(
                                    SidebarMenu::new()
                                        .children(
                                            items.iter().enumerate().map(|(ix, story)| {
                                                SidebarMenuItem::new(story.read(cx).name.clone())
                                                    .active(
                                                        self.active_group_index == Some(group_ix)
                                                            && self.active_index == Some(ix)
                                                    )
                                                    .on_click(cx.listener(move |this, _, _, cx| {
                                                        this.active_group_index = Some(group_ix);
                                                        this.active_index = Some(ix);
                                                        cx.notify();
                                                    }))
                                            })
                                        )
                                )
                        })
                    )
            )
            .child(
                // Main content
                div()
                    .flex_1()
                    .v_flex()
                    .child(
                        // Header
                        div()
                            .p_4()
                            .border_b_1()
                            .when_some(active_story, |this, story| {
                                let story_read = story.read(cx);
                                this.child(
                                    div()
                                        .v_flex()
                                        .gap_1()
                                        .child(div().text_xl().child(story_read.name.clone()))
                                        .child(div().text_sm().child(story_read.description.clone()))
                                )
                            })
                    )
                    .child(
                        // Story content
                        div()
                            .flex_1()
                            .overflow_y_scroll()
                            .when_some(active_story, |this, story| {
                                this.child(story.clone())
                            })
                    )
            )
    }
}
```

### Main Entry Point

```rust
// src/main.rs
mod gallery;
mod stories;

use gpui::*;
use gallery::Gallery;

fn main() {
    let app = Application::new();
    
    app.run(move |cx| {
        gpui_component::init(cx);
        cx.activate(true);
        
        cx.spawn(async move |cx| {
            cx.open_window(
                WindowOptions {
                    window_bounds: Some(WindowBounds::Windowed(Bounds {
                        origin: Point { x: px(100.0), y: px(100.0) },
                        size: Size { width: px(1200.0), height: px(800.0) },
                    })),
                    titlebar: Some(TitlebarOptions {
                        title: Some("Component Gallery".into()),
                        ..Default::default()
                    }),
                    ..Default::default()
                },
                |window, cx| {
                    cx.new(|cx| Gallery::new(window, cx))
                }
            )?;
            
            Ok::<_, anyhow::Error>(())
        }).detach();
    });
}
```

### Usage

```bash
# Run the gallery
cargo run

# Add new stories by:
# 1. Implement Story trait for your component
# 2. Add to stories/mod.rs
# 3. Register in Gallery::new()
```

### Benefits

- **Live component demos** - See components in all states
- **Interactive controls** - Toggle props via checkboxes
- **Organized navigation** - Sidebar with categories
- **Development tool** - Test components in isolation

### Pure GPUI Gallery (Without gpui-component)

If you prefer not to use gpui-component, implement the gallery with pure GPUI:

```rust
// Simplified StoryContainer without gpui-component
pub struct StoryContainer {
    pub name: SharedString,
    pub description: SharedString,
    content: Box<dyn Fn(&mut Window, &mut App) -> AnyElement>,
}

impl Render for StoryContainer {
    fn render(&mut self, window: &mut Window, cx: &mut Context<Self>) -> impl IntoElement {
        (self.content)(window, cx)
    }
}

// Simple sidebar implementation
fn render_sidebar(
    stories: &[(&str, Vec<SharedString>)],
    active_index: Option<usize>,
    cx: &Context<Gallery>,
) -> impl IntoElement {
    div()
        .w(px(250.0))
        .h_full()
        .bg(rgb(0x1a1a1a))
        .border_r_1()
        .border_color(rgb(0x2a2a2a))
        .v_flex()
        .p_4()
        .gap_2()
        .children(
            stories.iter().enumerate().flat_map(|(group_ix, (group_name, items))| {
                std::iter::once(
                    div()
                        .text_xs()
                        .text_color(rgb(0x888888))
                        .uppercase()
                        .mt_4()
                        .mb_2()
                        .child(*group_name)
                        .into_any_element()
                )
                .chain(items.iter().enumerate().map(move |(ix, name)| {
                    let is_active = active_index == Some(group_ix * 100 + ix);
                    div()
                        .px_3()
                        .py_2()
                        .rounded(px(4.0))
                        .cursor_pointer()
                        .when(is_active, |this| this.bg(rgb(0x3b82f6)))
                        .when(!is_active, |this| {
                            this.hover(|this| this.bg(rgb(0x2a2a2a)))
                        })
                        .child(name.clone())
                        .on_click(cx.listener(move |this, _, _, cx| {
                            this.active_index = Some(group_ix * 100 + ix);
                            cx.notify();
                        }))
                        .into_any_element()
                }))
            })
        )
}
```

**Key difference**: Pure GPUI implementation uses basic `div()` elements instead of pre-styled components from gpui-component.

## Project Checklist

When initializing a new project, ensure:

- [ ] `Cargo.toml` has correct package metadata
- [ ] `gpui` dependency is version `0.2.2` or later
- [ ] `main.rs` initializes `Application::new()`
- [ ] Window is created with `cx.open_window()`
- [ ] `.gitignore` excludes `/target` and IDE files
- [ ] `rust-toolchain.toml` specifies stable channel

## References

- [GPUI Documentation](https://gpui.rs)
- [Zed Repository](https://github.com/zed-industries/zed)
- [gpui-component](https://github.com/longbridge/gpui-component)
- [gpui-component Story System](https://github.com/longbridge/gpui-component/tree/main/crates/story)

