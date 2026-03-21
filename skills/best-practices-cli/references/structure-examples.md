# Structure Examples

## Before — flat, everything mixed together

```
cli/
├── index.js          # 800-line file: arg parsing + business logic + output
├── helpers.js        # random grab-bag of utilities
└── package.json
```

Problems: untestable without spawning a real process, impossible to reuse logic, changes break everything.

## After — layered

```
cli/
├── bin/
│   └── cli.js        # #!/usr/bin/env node — just wires commander and calls main()
├── src/
│   ├── commands/
│   │   ├── deploy.js       # parse flags → call core/deploy.js → print result
│   │   └── status.js
│   ├── core/
│   │   ├── deploy.js       # pure logic: build payload, call API, return result object
│   │   └── status.js
│   └── utils/
│       ├── logger.js       # createLogger(level) → { info, warn, error, debug }
│       └── spinner.js      # createSpinner() → { start, stop, fail }
├── test/
│   ├── core/
│   │   └── deploy.test.js  # unit tests — no subprocess, no I/O
│   └── commands/
│       └── deploy.test.js  # integration — call handler directly with mocked deps
└── package.json
```

**Rule of thumb**: if you need to spawn a child process to test it, the logic is in the wrong layer.

## Go equivalent

```
mycli/
├── cmd/
│   ├── root.go       # cobra root command
│   └── deploy.go     # thin: parse flags, call internal/deploy
├── internal/
│   ├── deploy/
│   │   ├── deploy.go
│   │   └── deploy_test.go
│   └── config/
│       └── config.go
├── pkg/              # reusable exported packages (if any)
└── main.go
```

## Python equivalent

```
mycli/
├── mycli/
│   ├── __main__.py   # entry point
│   ├── commands/
│   │   └── deploy.py
│   ├── core/
│   │   └── deploy.py
│   └── utils/
│       └── output.py
├── tests/
│   └── core/
│       └── test_deploy.py
└── pyproject.toml
```
