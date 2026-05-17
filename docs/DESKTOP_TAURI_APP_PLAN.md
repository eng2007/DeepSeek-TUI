# Desktop App Plan: Tauri 2 + React + TypeScript

## Goal

Build a separate desktop application using Tauri 2, React, TypeScript, and Vite.
The app should start as a GUI wrapper around the existing `deepseek` runtime
instead of rewriting the current `crates/tui` runtime.

## Recommended Architecture

Use the existing runtime API as the integration boundary:

```text
Desktop UI
  -> Tauri shell
    -> starts sidecar: deepseek serve --http
      -> communicates over HTTP/SSE
```

This keeps the first version small and avoids invasive changes to the large
`crates/tui` runtime. The desktop app can later move selected functionality into
direct Rust calls if the runtime API becomes a bottleneck.

## Phase Completion Rule

After every phase:

1. Add or update tests for the behavior introduced in that phase.
2. Run the relevant test suite and fix failures before moving on.
3. Make a git commit that contains only that phase's completed work.

For the desktop app, "relevant tests" should start with frontend unit/component
tests and Rust/Tauri backend tests, then expand to integration or smoke tests as
runtime process management, SSE, approvals, and packaging are added.

## Proposed Directory Layout

```text
apps/desktop/
  package.json
  vite.config.ts
  tsconfig.json
  src/
    main.tsx
    app/
    components/
    lib/runtime-client.ts
    lib/tauri.ts
    styles/
  src-tauri/
    Cargo.toml
    tauri.conf.json
    src/main.rs
```

For the MVP, keep `apps/desktop/src-tauri` isolated from the root Cargo
workspace unless there is a strong reason to include it. That prevents
`cargo test --workspace` in the main project from unexpectedly building Tauri.

## Phase 1: Scaffold

1. Create `apps/desktop`.
2. Initialize Vite React TypeScript.
3. Add Tauri 2.
4. Add development commands:
   - `npm run desktop:dev`
   - `npm run desktop:build`
5. Build a basic application shell:
   - sidebar
   - main transcript panel
   - composer area
   - status bar

Outcome: the desktop app opens with a basic empty UI.

## Phase 2: Runtime Bridge

1. Add a Tauri backend command that locates the `deepseek` binary.
2. Start `deepseek serve --http --host 127.0.0.1 --port <free-port>`.
3. Store the child process handle in Tauri state.
4. Shut down the child process cleanly when the desktop app exits.
5. Add a frontend runtime client:
   - health check
   - list threads/sessions
   - start turn
   - subscribe to SSE events
6. Show runtime state in the UI:
   - starting
   - ready
   - failed

Outcome: the desktop app can start and monitor a local DeepSeek runtime.

## Phase 3: MVP UI

Build the first useful application surface:

1. Workspace picker.
2. Session/thread list.
3. Transcript view.
4. Composer input.
5. Streaming assistant response through SSE.
6. Basic settings:
   - model
   - provider
   - approval mode
   - sandbox mode
7. Clear error display for auth and runtime failures.

Outcome: users can run a simple agent conversation from the desktop UI.

## Phase 4: Tool And Approval UX

1. Render tool calls as structured rows or compact cards.
2. Show tool state:
   - queued
   - running
   - waiting for approval
   - completed
   - failed
3. Add approval dialogs:
   - approve
   - deny
   - approve with remembered command prefix, if supported by the runtime API
4. Display key tool families clearly:
   - shell
   - file edits
   - git
   - web
   - sub-agents

Outcome: the desktop UI supports practical Agent-mode workflows.

## Phase 5: Sessions And Persistence

1. Show recent sessions.
2. Add resume and fork actions.
3. Support session title display and rename if available through the API.
4. Persist desktop-only preferences:
   - window size
   - last workspace
   - preferred theme
   - runtime port strategy

Outcome: the app becomes comfortable to use across restarts.

## Phase 6: Packaging

1. Configure Tauri bundles for Windows, macOS, and Linux.
2. Decide how to ship `deepseek`:
   - bundled sidecar binary
   - installed binary from `PATH`
   - bundled preferred with `PATH` fallback
3. Add a startup smoke test.
4. Add GitHub Actions packaging jobs.
5. Document install and troubleshooting paths.

Outcome: the desktop app can be distributed as a real application.

## Main Risks

- The runtime API may not expose every TUI interaction needed by the desktop UI.
- Sidecar lifecycle needs careful handling: ports, crashes, shutdown, and logs.
- Approval flow may require extensions in `crates/tui/src/runtime_api.rs`.
- Bundling `deepseek` as a sidecar will complicate release automation.
- WebView behavior differs across platforms, so UI testing needs Windows,
  macOS, and Linux coverage.

## Recommended MVP Scope

Start with:

1. `apps/desktop` scaffold.
2. Tauri process management for `deepseek serve --http`.
3. Runtime health check.
4. Session/thread list.
5. Simple chat composer.
6. SSE transcript rendering.

This creates a working desktop foundation while keeping changes to the existing
Rust runtime minimal.
