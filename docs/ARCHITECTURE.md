# Architecture

A SwiftUI iOS app for WHOOP 5.0 data. BLE and UI live in Swift; parsing, storage and metrics run in a Rust core compiled to a static library and called through a JSON-over-C bridge.

```mermaid
flowchart TD
    Band[(WHOOP 5.0 band)] <-->|BLE| BLE

    subgraph Swift["GooseSwift/ (SwiftUI)"]
        Shell["GooseSwiftApp · RootView · AppShellView<br/>AppRouter"]
        Model["GooseAppModel (+ extensions)<br/>state · lifecycle · overnight runs · notifications"]
        BLE["GooseBLEClient (+ extensions)<br/>scan · connect · historical sync"]
        Bridge[GooseRustBridge]
        Views["Health* · Fitness* · Activity* · Coach* · More"]
        Live[GooseWorkoutLiveActivityExtension]
    end

    subgraph Rust["Rust/core → libgoose_core.a"]
        Bridge2[bridge.rs / commands.rs]
        Proto[protocol.rs · historical_sync.rs]
        Store[store.rs]
        Metrics["metrics · sleep · recovery_rollup<br/>energy_rollup · step_counter · timeline"]
        Misc["export · health_sync · debug_ws<br/>privacy_lint · validation"]
        Bridge2 --> Proto --> Store
        Bridge2 --> Metrics --> Store
        Bridge2 --> Misc
    end

    Shell --> Model --> Views
    Model --> BLE
    BLE -->|packets| Model
    Model --> Bridge -->|JSON over C ABI| Bridge2
    Store -->|summaries| Bridge --> Model
    Model --> Live
    Views --> Coach[Coach chat] -.optional.-> LLM[(Hosted LLM)]
    Build["Scripts/build_ios_rust.sh<br/>Xcode build phase"] -.builds.-> Rust
    HK[(HealthKit)] <--> Model
```
