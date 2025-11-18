---
# You can also start simply with 'default'
theme: ../marcelkoch.net
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
image: ./images/title.png
# some information about your slides (markdown enabled)
title: Ein Core, sie zu rosten
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
transition: slide
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

# Ein Core, sie zu rosten

<v-clicks>
Langlebige & flexible Cross-Platform-Applikationen
</v-clicks>

---
layout: image
image: ./images/MK.png
---

---
layout: image-right
image: ./images/why-cross-plaform.png
---

## Warum Cross-Platform?

<v-clicks>

- Webtechnologien dominieren
- Native Apps weiterhin wichtig (z.B. Hardwarezugriffe, Performance)

### Native Apps bieten...

| Vorteile        | Nachteile                        |
|-----------------|----------------------------------|
| Performance     | spezifische Code Base            |
| Look & Feel     | unterschiedliche Konzepte / APIs |
| Hardwarezugriff | umständliche Installation        |

</v-clicks>

---

# Cross-platform frameworks

<v-clicks>

|              | **Flutter** | **React Native**        | Xamarin / MAUI | Kotlin Multiplattform | **Qt**         |
|--------------|-------------|-------------------------|----------------|-----------------------|----------------|
| **Language** | Dart        | JavaScript / TypeScript | C#             | Kotlin                | C++            |
| **Company**  | Google      | Facebook                | Microsoft      | Jetbrains             | The Qt company |

</v-clicks>

---
layout: image-left-mask
image: ./images/good-for-short-term-solutions.png
clipPath: inset(0 0 0 0 round 0 100% 100% 0)
---

## Gut für schnelle Ergebnisse


<v-clicks>

- Schnelle Ergebnisse durch (oft dynamische) Sprachen
- Gut für kurzlebige Apps (1–2 Jahre)
- Für langlebige Projekte (10+ Jahre) kritischer zu betrachten:
  - Wird das Framework langfristig gepflegt?
  - Reicht Support für Android/iOS/Web?
  - Genügt Performance einer WebView?
  - Wie gut ist die Testbarkeit?
  - Finde ich langfristig Entwickler?
  
</v-clicks>

---
layout: image-right
image: ./images/rust-core-2.png
---

## Langlebigkeit durch separaten Core

<v-clicks>

- UI-Technologie wandelt sich schnell
- Idee: Trennung von Core & UI (z. B. hexagonale Architektur)
- Core enthält Geschäftslogik und stabile Bestandteile
- UI und Plattform-APIs als externe Ports
- Beispiele für Ports: Kamera, Dateisystem, UI

</v-clicks>

---
layout: image-left
image: ./images/hexagonal.png
---

## Ports and Adapters

<v-clicks>

- Render: Port
- UI: Adapter
</v-clicks>

---
layout: image-right
image: ./images/which-language.png
---

## Wahl der Core-Sprache

<v-clicks>

- Zielplattformen: Android, iOS, Windows, macOS, Raspberry Pi
- Wichtige Kriterien:
  - Stabilität (wenig grundlegende Änderungen)
  - Robustheit (gut wartbarer Code)
  - Langlebigkeit (Unterstützung durch große Firmen)
  - Flexibilität (Einsetzbarkeit auf vielen Plattformen inkl. Web)
</v-clicks>

---
layout: image-right
image: ./images/rust-for-core.png
---

## Rust für den Core

<v-clicks>

- Wartbarer, performanter Core mit Rust
- Plattformunterstützung: Desktop, Mobile, Web (via WASM), SBCs
- Explizite Syntax und Compiler unterstützen langfristige Wartbarkeit
- Lernkurve höher, kleinere Community als JS oder C
- Wachsende Community, viele Migrationen von C nach Rust
- Unterstützung durch Firmen wie AWS, Google, Meta, Microsoft
</v-clicks>

---

## Architektur des Cores

<v-clicks>

- Mehr Logik im Core = mehr Wiederverwendbarkeit
- UI wird schlanker und reagiert auf ViewModel

<img src="/images/Architektur_1.png" />

</v-clicks>
--

## Architekturprinzip MVVM

<v-clicks>

- ViewModel kapselt Anzeige-Daten
- View liest ViewModel, UI bleibt zustandslos
- Beispiel in Rust:

  ```rust
  ViewModel {
    name: String,
    email: String
  }
  ```

</v-clicks>

---

## UI-Aktionen als fachliche Aktionen (Domain Events)

<v-clicks>

- UI-Events wie `onClick` werden in fachliche Aktionen übersetzt
- Aktionen sind die Eingangsschnittstelle in den Core
- Beispiel in Rust (Speichern von E-Mail und Name):

  ```rust
  pub enum Actions {
    ChangeName(String),
    ChangeEMail(String),
    ApplyChanges,
  }
  ```

</v-clicks>
---

## Zustand im Core

<v-clicks>

- Core verwaltet den Zustand, nicht die UI
- Beispiel:

  ```rust
  struct State {
    name: String,
    email: String
  }
  ```
</v-clicks>

---
layout: two-cols-header
---

## Implementierung des Core

<v-clicks>

- Zentrale Struct `App` verarbeitet Aktionen und liefert ViewModel
- Trennung von State, Actions und ViewModel

</v-clicks>

::left::

<v-clicks>

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

</v-clicks>

::right::

<v-clicks>

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

</v-clicks>

---
layout: two-cols-header
---

## Validierung im Core (1)

<v-clicks>

- ViewModel enthält auch Fehlermeldungen
- E-Mail-Validierung im Core
- Beispiel: Prüfung auf `@`
</v-clicks>

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

## Validierung im Core (2)

<v-clicks>

- In `do_action()` und `render_view_model()`
</v-clicks>

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
                Some(_err) => Some("Keine valide E-Mail-Adresse".into()),
            },
        },
    }
```

---
layout: image-grid
image: ./images/crux.png
---

## Crux

<v-clicks>

- Actions sind Events
- State ist Model
- App
- Effect (Was raus geht)
- Command (Erst Effect, dann Event)
- Operation (Request-Payload)
</v-clicks>

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

<v-clicks>

- Umschließt die App
- Hält das Model versteckt
</v-clicks>

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

<v-clicks>

- Umschließt den Core
- Kümmert sich um (De)serialisierung
</v-clicks>

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

<v-clicks>

- Die App direkt testen
- Den Core innerhalb eines Integrationtests testen
</v-clicks>

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

## Anbindung an die Shell

Kleber für die Tapete

<v-clicks>

- uniffi (& serde_generate)
- wasm-pack
- Flutter Rust Bridge
- FFI
</v-clicks>

---
layout: two-cols
---

## uniFFI

<v-clicks>

- Mit Macros (oder interface definition file) Funktionsschnittstellen in Fremdsprachen generieren
</v-clicks>

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

<v-clicks>

- Generierte komplexe Typen in Fremdsprachen
</v-clicks>

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

<v-clicks>

- Generiert WASM-Modul und JS-Glue-Code
</v-clicks>

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

<v-clicks>

- Rust-Code in Flutter-Projekt integrieren
</v-clicks>

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

# Es kam einmal ein Kunde

<v-clicks>

- C
  - War gesetzt
  - Ist gesetzt
- Worauf bauen wir die UI?
  - Flutter
  - React Native
</v-clicks>

---

## Erfahrungen aus der Praxis

<v-clicks>

- C
- uniffi - c# ist nicht so einfach
- Mehr als crux liefert
- CI/CD
</v-clicks>

---
layout: image-right
image: ./images/c-build.png
---

# Um C zu unterstützen...

<v-clicks>

- bauen wir libs vor
- unterstützen wir 11 targets
- binden wir die libs in die `build.rs` mit eigenem Tool `grab-n-spread` ein
- C-Integration an sich kein Problem
- Cross compiling sehr wohl
  - Windows: .lib falsche Berechnung
  - dockcross
  - osxcross
</v-clicks>

---

## Mehr als crux mitbringt

<v-clicks>

- I18N (integriert im Kompilat)
- Labels
- Logging
</v-clicks>

---
layout: image-right
image: ./images/rust-for-core.png
---

## Zusammenfassung

<v-clicks>

- Architektur trennt langlebigen Core von kurzlebiger UI
- Core übernimmt soviel wie möglich: Zustand, Logik, Validierung
- Mögliche Lösung: Rust, crux, Flutter, FRB
- Artikelserie auf heise.de/developer: "heise developer rust marcel" duckduckgoen
</v-clicks> 

---
layout: image
image: ./images/MK-end.png
---
