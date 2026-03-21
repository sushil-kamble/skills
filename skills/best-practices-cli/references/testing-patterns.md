# Testing Patterns for CLI Tools

## Temp directory helper (Node.js)

```ts
import { mkdtempSync, rmSync } from 'node:fs';
import { tmpdir } from 'node:os';
import { join } from 'node:path';

function createTempDir() {
  const dir = mkdtempSync(join(tmpdir(), 'cli-test-'));
  return {
    path: dir,
    cleanup: () => rmSync(dir, { recursive: true, force: true }),
  };
}

// Usage in test
let tmp: ReturnType<typeof createTempDir>;
beforeEach(() => { tmp = createTempDir(); });
afterEach(() => { tmp.cleanup(); });
```

## Testing a command handler directly (not via subprocess)

Prefer calling handlers directly over spawning a subprocess — it's faster, isolates the logic under test, and avoids flakiness from process startup time.

```ts
// ✅ Fast, no process overhead, fully injectable
import { deployHandler } from '../src/commands/deploy.js';

it('prints success on valid input', async () => {
  const logger = { info: jest.fn(), error: jest.fn() };
  await deployHandler({ env: 'staging' }, { logger });
  expect(logger.info).toHaveBeenCalledWith(expect.stringContaining('deployed'));
});
```

When you do need to invoke the binary (end-to-end confidence), prefer `execFile` over `exec` to avoid shell injection:

```ts
import { execFile } from 'node:child_process';
import { promisify } from 'node:util';
const execFileAsync = promisify(execFile);

const { stdout } = await execFileAsync('node', ['bin/cli.js', 'deploy', '--env', 'staging']);
```

## Capturing stdout/stderr (Node.js)

```ts
function captureOutput() {
  const lines: string[] = [];
  const orig = process.stdout.write.bind(process.stdout);
  process.stdout.write = (chunk: string) => { lines.push(chunk); return true; };
  return {
    lines,
    restore: () => { process.stdout.write = orig; },
  };
}
```

## Temp dir + filesystem assertion (Go)

```go
func TestWriteConfig(t *testing.T) {
    dir := t.TempDir() // auto-cleaned after test
    path := filepath.Join(dir, "config.json")
    err := WriteConfig(path, Config{Token: "abc"})
    require.NoError(t, err)
    data, _ := os.ReadFile(path)
    assert.Contains(t, string(data), "abc")
}
```

## Capturing CLI output (Python / Click)

```python
from click.testing import CliRunner
from mycli.commands.deploy import deploy

def test_deploy_success():
    runner = CliRunner()
    result = runner.invoke(deploy, ['--env', 'staging'])
    assert result.exit_code == 0
    assert 'deployed' in result.output
```

## Error path testing — what to cover

- Missing required argument → exits non-zero with usage hint
- Invalid value (e.g., unknown env name) → descriptive error, no stack trace
- Network/IO failure → message that tells the user what to do next
- File not found → says which file and suggests a fix
