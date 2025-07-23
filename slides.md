---
# You can also start simply with 'default'
theme: ../marcelkoch.net
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
image: ./images/title.png
# some information about your slides (markdown enabled)
title: One core to rust them all
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# apply unocss classes to the current slide
class: text-center
# https://sli.dev/features/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations.html#slide-transitions
transition: none
# enable MDC Syntax: https://sli.dev/features/mdc
mdc: true
# open graph
# seoMeta:
#  ogImage: https://cover.sli.dev
fonts:
  family: 'Asap'
  headings: 'Asap Condensed'
  body: 'Asap'
  monospace: 'Ubuntu Mono'
colors:
  primary: '#5B0993'
  secondary: '#FD722B'
  tertiary: '#1F859D'
  background: '#fdfcfd'
  text: '#1a1a1a'
  link: '#7426AA'
  code: '#013E4C'
layout: image
---

---
layout: cover
---

# One core to rust them all

<v-clicks>
Long-lasting & flexible cross-platform applications
</v-clicks>

---
layout: image
image: ./images/MK.png
---

---
layout: image-right
image: ./images/why-cross-plaform.png
---

## Why Cross-Platform?

- Web technologies dominate
- Native apps are still important (e.g., hardware access, performance)

### Native apps offer...

| Advantages      | Disadvantages                    |
|-----------------|----------------------------------|
| Performance     | specific code base               |
| Look & feel     | different concepts / APIs        |
| Hardware access | cumbersome installation          |


---

# Cross-platform frameworks

|              | **Flutter** | **React Native**        | Xamarin / MAUI | Kotlin Multiplattform | **Qt**         |
|--------------|-------------|-------------------------|----------------|-----------------------|----------------|
| **Language** | Dart        | JavaScript / TypeScript | C#             | Kotlin                | C++            |
| **Company**  | Google      | Facebook                | Microsoft      | Jetbrains             | The Qt company |

---
layout: image-left-mask
image: ./images/good-for-short-term-solutions.png
clipPath: inset(0 0 0 0 round 0 100% 100% 0)
---

## Good for quick results

- Fast results thanks to (often dynamic) languages
- Good for short-lived apps (1–2 years)
- For long-lived projects (10+ years) more critical:
  - Will the framework be maintained long-term?
  - Is support for Android/iOS/Web sufficient?
  - Is the performance of a WebView enough?
  - How good is the testability?
  - Will I find developers in the long run?

---
layout: image-right
image: ./images/rust-core-2.png
---

## Longevity through a separate core

- UI technology changes quickly
- Idea: separation of core & UI (e.g., hexagonal architecture)
- Core contains business logic and stable components
- UI and platform APIs as external ports
- Examples of ports: camera, file system, UI

---
layout: image-left
image: ./images/hexagonal.png
---

## Ports and Adapters

- Render: Port
- UI: Adapter


---
layout: image-right
image: ./images/which-language.png
---

## Choice of core language

- Target platforms: Android, iOS, Windows, macOS, Raspberry Pi
- Important criteria:
  - Stability (few fundamental changes)
  - Robustness (well maintainable code)
  - Longevity (support from large companies)
  - Flexibility (usable on many platforms including web)

---
layout: image-right
image: ./images/rust-for-core.png
---

## Rust for the core

- Maintainable, high-performance core with Rust
- Platform support: desktop, mobile, web (via WASM), SBCs
- Explicit syntax and compiler support long-term maintainability
- Steeper learning curve, smaller community than JS or C
- Growing community, many migrations from C to Rust
- Supported by companies like AWS, Google, Meta, Microsoft

---

## Core architecture

- More logic in the core = more reusability
- UI becomes slimmer and reacts to the ViewModel

<img src="/images/Architektur_1.png" />

--

## MVVM architecture principle

- ViewModel encapsulates display data
- View reads ViewModel, UI remains stateless
- Example in Rust:
  ```rust
  ViewModel {
    name: String,
    email: String
  }
  ```

---

## UI actions as domain events

- UI events like `onClick` are translated into domain actions
- Actions are the entry point into the core
- Example in Rust (saving email and name):
  ```rust
  pub enum Actions {
    ChangeName(String),
    ChangeEMail(String),
    ApplyChanges,
  }
  ```

---

## State in the core

- The core manages the state, not the UI
- Example:
  ```rust
  struct State {
    name: String,
    email: String
  }
  ```

---
layout: two-cols-header
---

## Implementation of the core

- Central struct `App` processes actions and returns the ViewModel
- Separation of state, actions, and ViewModel

::left::

```rust {all|8-19|9-17|10-12|18} twoslash
impl Core {
    pub fn new() -> Core {
        Core {
           state: State::default()
        }
    }

    pub fn do_action(&mut self, action: Actions) -> ViewModel {
        match action {
            Actions::ChangeName(name) => {
                self.state.name = name;
            }
            Actions::ChangeEMail(email) => {
                self.state.email = email;
            }
            Actions::ApplyChanges => {}
        }
        self.render_view_model()
    }
```

::right::

```rust {all|2-5|3|all} twoslash
    pub fn render_view_model(&self) -> ViewModel {
        ViewModel{
            name: self.state.name.clone(),
            email: self.state.email.clone()
        }
    }
}

impl Default for Core {
    fn default() -> Self {
        Self::new()
    }
}
```

---
layout: two-cols-header
---

## Validation in the core (1)

- ViewModel also contains error messages
- Email validation in the core
- Example: check for `@`

::left::
```rust {all|1-3|5-8|12} twoslash
enum EMailErrors {
    NoAt,
}

struct StateField<F, E> {
    field: F,
    error: Option<E>,
}

struct State {
    name: String,
    email: StateField<String, EMailErrors>,
}
```
::right::
```rust {all|1-4|8} twoslash
pub struct ViewModelField<F> {
  value: F,
  error: Option<String>,
}

pub struct ViewModel {
  pub name: String,
  pub email: ViewModelField<String>,
}
```

---
layout: two-cols-header
---

## Validation in the core (2)

- In `do_action()` and `render_view_model()`

::left::

```rust {all|1-5|7-10} twoslash
Actions::ChangeEMail(email) => {
    let email_error = if email.contains("@") {
        None
    } else {
        Some(NoAt)
    };

    self.state.email = StateField {
        field: email,
        error: email_error,
    };
}
```
::right::
```rust {all|3-9|4|5-8} twoslash
ViewModel {
        name: self.state.name.clone().into(),
        email: ViewModelField {
            value: self.state.email.field.clone().into(),
            error: match &self.state.email.error {
                None => None,
                Some(_err) => Some("Not a valid email address".into()),
            },
        },
    }
```

---
layout: image-grid
image: ./images/crux.png
---

## Crux

- Actions are events
- State is model
- App
- Effect (what goes out)
- Command (first effect, then event)
- Operation (request payload)

---
layout: image
image: ./images/architecture.svg
---

---
layout: two-cols
---

```rust {all|1-4|6-9|11-14|16-19|21-22} twoslash
#[derive(Deserialize, Serialize)]
pub enum Event {
    ChangeEmail(String),
}

#[derive(Default)]
pub struct Model {
    email: String,
}

#[derive(Deserialize, Serialize)]
pub struct ViewModel {
    pub email: String,
}

#[effect]
pub enum Effect {
    Render(RenderOperation),
}

#[derive(Default)]
pub struct EmailApp;
```

::right::

```rust  {all|2-6|8-18|20-24} twoslash
impl App for EmailApp {
  type Event = Event;
  type Model = Model;
  type ViewModel = ViewModel;
  type Capabilities = ();
  type Effect = Effect;

  fn update(
    &self,
    event: Self::Event,
    model: &mut Self::Model,
    _caps: &Self::Capabilities,
  ) -> Command<Self::Effect, Self::Event> {
    match event {
      Event::ChangeEmail(email) => model.email = email.clone(),
    }
    render()
  }

  fn view(&self, model: &Self::Model) -> Self::ViewModel {
    ViewModel {
      email: model.email.clone(),
    }
  }
}
```
---
layout: image
image: ./images/architecture.svg
---

---
layout: two-cols
---

## Core

- Wraps the app
- Keeps the model hidden

```rust {all|1|3-4|6-14} twoslash
let core: Arc<Core<EmailApp>> = Arc::new(Core::new());

let effects: Vec<Effect> =
    core.process_event(ChangeEmail("m@rcelko.ch".into()));

for effect in effects {
  match effect {
    Effect::Render(_) => {
      let view_model = core.view();
      
      assert_eq!(view_model.email, "m@rcelko.ch")
    }
  }
}
```

::right::

## Bridge

- Wraps the core
- Handles (de)serialization

```rust {all|1-2|4|6-7|9-22} twoslash
let serialized = 
    bincode::serialize(&ChangeEmail("m@rcelko.ch".into())).unwrap();

let effects: Vec<u8> = bridge.process_event(&serialized).unwrap();

let effects: Vec<Request<EffectFfi>> =
bincode::deserialize(effects.as_slice()).unwrap();

for request in effects {
  let effect = request.effect;
  
  match effect {
    EffectFfi::Render(_) => {
    let view_model = bridge.view().unwrap();
    let view_model: ViewModel =
    bincode::deserialize(&view_model).unwrap();
    
    assert_eq!(view_model.email, "m@rcelko.ch")
    }
  }
}
```

---
layout: image
image: ./images/architecture.svg
---

---
layout: two-cols
---

````md magic-move {lines: true}
```rust
#[derive(Deserialize, Serialize)]
pub enum Event {
    ChangeEmail(String),
}

#[derive(Default)]
pub struct Model {
    email: String,
}

#[derive(Deserialize, Serialize)]
pub struct ViewModel {
    pub email: String,
}

#[effect]
pub enum Effect {
    Render(RenderOperation),
}

#[derive(Default)]
pub struct EmailApp;
```

```rust {all|4,10,16,22|22} twoslash
#[derive(Deserialize, Serialize)]
pub enum Event {
    ChangeEmail(String),
    EmailSent(bool),
}

#[derive(Default)]
pub struct Model {
    email: String,
    sent: bool,
}

#[derive(Deserialize, Serialize)]
pub struct ViewModel {
    pub email: String,
    pub sent_message: Option<String>,
}

#[effect]
pub enum Effect {
    Render(RenderOperation),
    StartWritingEmail(StartWritingEmailOp),
}

#[derive(Default)]
pub struct EmailApp;
```

```rust {6|5-6|9-16|9-12|14-16} twoslash
// ...

#[effect]
pub enum Effect {
    Render(RenderOperation),
    StartWritingEmail(StartWritingEmailOp),
}

#[derive(Deserialize, Serialize, Clone, PartialEq, Debug)]
pub struct StartWritingEmailOp {
    pub subject: String,
}

impl Operation for StartWritingEmailOp {
    type Output = bool;
}
```

````

::right::

````md magic-move {lines: true}
```rust  {all|2-7|8-13} twoslash
impl App for EmailApp {
  fn update(...) -> Command<Self::Effect, Self::Event> {
    match event {
      Event::ChangeEmail(email) => model.email = email.clone(),
    }
    render()
  }
  
  fn view(&self, model: &Self::Model) -> Self::ViewModel {
    ViewModel {
      email: model.email.clone(),
    }
  }
}
```

```rust  {all|7-12|14-17|11,14-17|23-27} twoslash
impl App for EmailApp {
  fn update(...) -> Command<Self::Effect, Self::Event> {
        match event {
            Event::ChangeEmail(email) => {
                model.email = email.clone();

                let command: Command<Self::Effect, Self::Event> =
                    Command::request_from_shell(StartWritingEmailOp {
                        subject: "Moin!".into(),
                    })
                    .then_send(|output| Event::EmailSent(output));
                Command::all([command, render()])
            }
            Event::EmailSent(sent) => {
                model.sent = sent;
                render()
            }
        }
    }
    fn view(&self, model: &Self::Model) -> Self::ViewModel {
        ViewModel {
            email: model.email.clone(),
            sent_message: if model.sent {
                Some("Erfolgreich geschickt".into())
            } else {
                None
            },
        }
    }
}
```
````

---
layout: two-cols
---

## Tests

- Test the app directly
- Test the core within an integration test

::right::

```rust {all|5-8,11|12|14,16,17} twoslash
#[test]
fn simple_test() {
  let app = EmailApp;

  let mut model = Model {
    email: "hello@example.com".to_string(),
    sent: false,
  };

  let event = Event::EmailSent(true);
  let mut cmd = app.update(event, &mut model, &());
  assert_effect!(cmd, Effect::Render(_));

  let view = app.view(&model);

  assert_eq!(view.sent_message.unwrap(),
             "Erfolgreich geschickt".to_string());
}
```

---
layout: image-right
image: ./images/glue-tech.png
---

## Connecting to the shell

Glue for the wallpaper

- uniffi (& serde_generate)
- wasm-pack
- Flutter Rust Bridge
- FFI

---
layout: two-cols
---

## uniFFI
- Generate function interfaces in foreign languages using macros (or interface definition file)

::right::

```rust  {all|4} twoslash
static CORE: LazyLock<Bridge<EmailApp>>
    = LazyLock::new(|| Bridge::new(Core::new()));

#[uniffi::export]
#[must_use]
pub fn process_event(data: &[u8]) -> Vec<u8> {
  match CORE.process_event(data) {
    Ok(effects) => effects,
    Err(e) => panic!("{e}"),
  }
}
```
---
layout: two-cols
---

## serde_generate
- Generated complex types in foreign languages

::right::

```rust  {all} twoslash
use serde_generate::{
  Tracer, code_generator::CodeGeneratorConfig,
  typescript::Installer, bincode::BincodeSerializer,
};

fn main() -> anyhow::Result<()> {
  let mut tracer = Tracer::new(Default::default());
  tracer.trace_type::<cross_platform_crux::Event>()?;
  let registry = tracer.registry()?;

  let out_dir = "bindings/ts";
  std::fs::create_dir_all(out_dir)?;

  Installer::new(out_dir)
          .with_serializer(BincodeSerializer::default())
          .install(&registry)?;

  Ok(())
}
```

---
layout: two-cols
---

## wasm-pack
- Generates WASM module and JS glue code

::right::

```rust  {all|4} twoslash
static CORE: LazyLock<Bridge<EmailApp>>
    = LazyLock::new(|| Bridge::new(Core::new()));

#[cfg_attr(target_family = "wasm", wasm_bindgen::prelude::wasm_bindgen)]
#[must_use]
pub fn process_event(data: &[u8]) -> Vec<u8> {
  match CORE.process_event(data) {
    Ok(effects) => effects,
    Err(e) => panic!("{e}"),
  }
}
```

```typescript [cross_platform_crux.d.ts] {all|1}
export function process_event(data: Uint8Array): Uint8Array;
```

---
layout: two-cols
---

## Flutter Rust Bridge
- Integrate Rust code into Flutter project

```rust [frb_cross_platform_core.rs]
static CORE_CELL: OnceLock<EmailAppCore> = OnceLock::new();

#[frb(sync)]
pub fn init_app_and_get_view_model() -> ViewModel {
  let core = cross_platform_rust::create_core();
  let view_model = core.view();
  let _ = CORE_CELL.set(core);
  view_model
}

#[frb(mirror(Event))]
#[frb(dart_metadata=("freezed"))] // generates dart classes
pub enum _Event {
  ChangeEmail(String),
  EmailSent(bool),
}
```

::right::

```dart
import 'package:cross_platform_rust_app/src/rust/api/frb_cross_platform_core.dart';
import 'package:cross_platform_rust_app/src/rust/frb_generated.dart';

Future<ViewModel> initRustCore() async {
  await RustLib.init();
  
  final viewModel = initAppAndGetViewModel();

  return viewModel;
}
```
---
layout: image-left
image: ./images/customer.png
---

# Once upon a time there was a customer

- C
  - Was set
  - Is set
- What do we build the UI with?
  - Flutter
  - React Native

---

## Practical experience

- C
- uniffi - C# is not so easy
- More than crux provides
- CI/CD

---
layout: image-right
image: ./images/c-build.png
---

# To support C...

- we pre-build libraries
- we support 11 targets
- we integrate the libs into `build.rs` with our own tool `grab-n-spread`
- C integration itself is not a problem
- Cross compiling very much so
  - Windows: .lib wrong calculation
  - dockcross
  - osxcross

---

## More than crux provides

- I18N (integrated in the binary)
- Labels
- Logging

---
layout: image-right
image: ./images/rust-for-core.png
---

## Summary

- Architecture separates long-lived core from short-lived UI
- Core takes over as much as possible: state, logic, validation
- Possible solution: Rust, crux, Flutter, FRB

---
layout: image
image: ./images/MK-end.png
---
