---
name: go-bootstrap
description: Proper Go application bootstrap pattern. Use when writing application main() functions, especially for long-running servers, daemons, or services that require cleanup (logging, database connections, Sentry, etc).
---

# Go Bootstrap Skill

## Core Principle: Two-Phase Startup

Separate the `main()` function into two distinct phases to ensure defers always run and cleanup happens gracefully.

```
Phase 1 (main): Setup — no defers yet
  └─ Load configuration
  └─ Create logger
  └─ Initialize optional services (Sentry, etc)
  └─ os.Exit() is SAFE here if needed

Phase 2 (runWithCleanup): Application lifecycle — defers active
  └─ defer cleanup actions
  └─ Call run() — handles signals, server, worker loops
  └─ return (never os.Exit) — defers unwind first
  └─ Process exits 0 naturally
```

## The Pattern

```go
func main() {
    // Phase 1: Setup (no defers yet)
    conf := config.GetConfig()
    rawLogger := newLogger(conf.AppENV == "prod")
    log := logger.New(rawLogger)

    if err := sentryx.Init(conf, log); err != nil {
        log.Error("failed to initialize sentry", "err", err)
        // Optional service failed - continue anyway (or os.Exit(1) is safe here)
    }

    // Phase 2: Run with cleanup (defers active from here)
    if err := runWithCleanup(log, rawLogger, conf); err != nil {
        log.Error("application exited with error", "error", err)
        return // safe - defers run before process exit
    }

    log.Info("application gracefully stopped")
}

// runWithCleanup wraps the application lifecycle with guaranteed cleanup
func runWithCleanup(log logger.Logger, rawLog *slog.Logger, conf *config.Config) error {
    defer sentryx.Flush()
    defer closeConnections()
    // Add any other defers here
    
    return run(log, rawLog, conf)
}

// run contains the main application logic (signal handling, server, workers)
func run(log logger.Logger, rawLog *slog.Logger, conf *config.Config) error {
    ctx, stop := signal.NotifyContext(context.Background(), syscall.SIGINT, syscall.SIGTERM)
    defer stop()
    
    // Your application logic here
    // Return errors naturally - defers in runWithCleanup will execute
}
```

## Why This Works

### Problem: os.Exit Skips Defers

```go
// WRONG: This pattern is buggy
func main() {
    defer cleanup()
    if err != nil {
        os.Exit(1)  // ❌ cleanup() never runs
    }
}
```

### Solution: Defer Registration After Phase 1

```go
// RIGHT: Defers only registered in Phase 2
func main() {
    // Phase 1: no defers
    initOptional()
    
    // Phase 2: defers start here
    if err := runWithCleanup(...) {
        return  // ✓ defers run before exit
    }
}

func runWithCleanup(...) error {
    defer cleanup()
    return run(...)
}
```

## When to Use os.Exit in Phase 1

Use `os.Exit()` in Phase 1 **only** for fatal startup failures that should trigger orchestrator restarts:

```go
func main() {
    conf := config.GetConfig()
    log := newLogger(conf.AppENV == "prod")
    
    // These are fatal - fail fast before any defers
    if conf == nil {
        log.Error("config is missing")
        os.Exit(1)  // Safe - no defers registered
    }
    
    if err := someRequiredDependency.Init(); err != nil {
        log.Error("required service failed", "err", err)
        os.Exit(1)  // Safe - no defers registered
    }
    
    // Optional services - log but continue
    if err := sentryx.Init(...); err != nil {
        log.Error("optional service failed", "err", err)
        // No os.Exit - continue to Phase 2
    }
    
    // Phase 2 - defers now safe
    if err := runWithCleanup(...) {
        return  // Defers run
    }
}
```

## Exit Codes in Modern Architectures

**Old model:** Exit codes signal health to orchestrators.

**Modern model (Kubernetes, systemd):**
- Liveness and readiness **probes** detect unhealthy state independently
- `restartPolicy: Always` (default) restarts on any exit
- Exit code 0 for graceful shutdown, even with application-level failures
- Probes catch actual failures and trigger restarts

**Conclusion:** You don't need non-zero exit codes for runtime errors. Save `os.Exit(non-zero)` for startup failures before Phase 2.

## Checklist

When writing `main()`:

- [ ] Phase 1: Load config, create logger, init optional services
- [ ] Phase 2 starts: All defers registered here
- [ ] `runWithCleanup()` wraps `run()` and defers cleanup
- [ ] Phase 1 code: Can use `os.Exit()` for fatal errors
- [ ] Phase 2 code: Always `return`, never `os.Exit()`
- [ ] `run()`: Handles signals, servers, workers; returns errors naturally
- [ ] Lint: Run `golangci-lint` to catch any remaining `exitAfterDefer` issues

## Common Cleanup Actions

```go
func runWithCleanup(log logger.Logger, conf *config.Config) error {
    defer sentryx.Flush()
    defer func() {
        log.Info("flushing Sentry events")
    }()
    
    defer func() {
        if err := db.Close(); err != nil {
            log.Error("database close failed", "err", err)
        }
    }()
    
    defer func() {
        if err := publisher.Close(ctx); err != nil {
            log.Error("message publisher close failed", "err", err)
        }
    }()
    
    return run(log, conf)
}
```

## References

- **Linters:** `golangci-lint` gocritic rule `exitAfterDefer`
- **Pattern name:** "inner main" or "defer-safe main"
- **Go idiom:** Deferring cleanup after phase 1 ensures cleanup runs on normal exit
