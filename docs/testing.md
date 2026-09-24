---
description: Run the suite with or without the rasterizer binaries installed, and see which groups need a real adapter on the machine.
---

# Testing

The full PHPUnit suite runs with:

```bash
vendor/bin/phpunit
```

Most tests are pure unit tests and do not require rasterizer binaries:

```bash
vendor/bin/phpunit --exclude-group system
```

System tests exercise the real binaries when they are available in `PATH`.
They are marked with the `system` PHPUnit group and skip themselves when the
required binary is missing:

```bash
vendor/bin/phpunit --group system
```

Current system coverage:

- `resvg` adapter rendering and sizing.
- `rsvg-convert` adapter rendering, sizing, and `keepAspectRatio`.
- binary resolution through the host `PATH`.

Unit coverage also checks command construction through a `TraceableRunner`
decorator and a fake PNG-writing runner, so adapter options can be asserted
without spawning `resvg` or `rsvg-convert`.
