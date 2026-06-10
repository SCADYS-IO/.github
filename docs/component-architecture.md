# Component architecture & design principles

These are the shared design principles every SCADYS-IO driver component follows.
They are deliberately uniform: once one component is familiar, the rest read the
same way. Each component's README documents only what is specific to its part —
what the device covers and the maths of its registers — and links here for the
common rationale. The canonical copy lives in the
[`SCADYS-IO/.github`](https://github.com/SCADYS-IO/.github) organization profile
repository.

For the language and consumption contract (C with `extern "C"` headers, taking
ESP-IDF handles directly, and using a C component from C++), see
[Consuming components](consuming-components.md).

## Full device coverage, grown by need

One driver covers a device and its register-identical siblings (for example
TMP102 and TMP112, or the A and B accuracy grades of a part). The *end state*
is the full feature surface — every operating mode, configuration field, and
status flag the device offers — because a driver that permanently hides features
forces the next user to fork it. The *first release*, however, is deliberately
lean: it exposes the minimum API the consuming product needs, or can plausibly
be expected to need, and coverage then grows toward the full surface in additive
stages as real needs pull capability in — see
[Staged growth by composition](#staged-growth-by-composition).

## Configuration as a struct

A device's configuration register is modelled as a plain struct that mirrors it
field for field, with paired `*_get_config` / `*_set_config` calls. Changing a
setting is a read-modify-write: read the struct, change the relevant fields, write
it back. This never clobbers the unrelated bits, and the struct reads in the same
order as the datasheet's register table. The alternatives are rejected: a setter
per bit becomes a sprawl of near-identical functions, and a struct-only API
forces callers to track the whole register state themselves.

## Convenience setters for common cases only

The operations done most often get a one-line helper (for example a shutdown
toggle, an alert-threshold pair, or a calibration call). Rarely-touched fields go
through the config struct instead of each gaining its own function. This keeps the
API small without hiding anything — every field remains reachable through
`*_get_config` / `*_set_config`.

## Reads return physical units

Measurement reads return the physical quantity in SI units as a `float` — degrees
Celsius, volts, amps, watts — not raw counts left for the caller to scale. Where a
device reports one quantity that is wanted in several units, one read is the base
(the device's native quantity) and the others are thin converters chained off it,
so the register decode lives in exactly one place and the variants never disagree.

## Raw register access

Alongside the converted reads, each driver exposes the unmodified register value
through a `*_read_raw` (or per-quantity raw) accessor. This supports logging the
exact bits, applying a custom or fixed-point scaling, or avoiding floating point
entirely. It is the same bus transaction as the converted read, without the
conversion step.

## A host-testable core

All of a driver's arithmetic — unit conversion, register bit-packing, and any
calibration maths — lives in a pure core (named `<component>_convert`) that
depends only on `<stdint.h>` / `<stdbool.h>`, with no ESP-IDF or hardware calls.
That core compiles and runs on a host, so its maths is covered by unit tests that
need no device, and continuous integration runs those tests plus an on-target
build on every change. The hardware layer above it does only I²C/SPI/UART I/O.

## Staged growth by composition

A large, multi-capability device — an IMU with an accelerometer, a gyroscope, a
FIFO, interrupt routing, and embedded functions, say — reaches full coverage in
**stages** rather than in one release. The first stage is deliberately **lean**:
a lightweight component exposing the minimum API the consuming product actually
needs — or can plausibly be expected to need — and nothing more. Stage 1 is
scoped from the product requirement, not from the device's feature list;
capability the product has no use for stays off the API until a later stage has
a real need to pull it in. Each later stage adds capability by **composition**:
new functions over the same handle, and where a capability is substantial, its
own source/header pair (`<component>_fifo.c`, `<component>_events.c`, …) that
shares the component's private register helpers. A later stage only *adds*
symbols — it never changes the earlier API — and CalVer marks the progression.
These are C components, so this is composition, not inheritance: there is no
base type, only a shared handle and shared register access. The lean first stage
applies to every part, large or small: a part whose product need genuinely spans
the whole device reaches full coverage in its first release by that scoping — as
the outcome of asking what the product needs, never as a default. The end state
is the same full coverage either way; staging is how a part gets there without a
breaking rewrite, and the lean first stage is what keeps a product build from
carrying surface it does not use.

---

## What a component README adds

A component's own README does not repeat the principles above. It documents the
part-specific detail:

- **What it covers** — the exact device(s), the feature surface of the current
  stage, and any register-compatible siblings the one driver handles.
- **Why this scope** — the product need the current stage was scoped from and
  how each consuming product uses the part; what is staged out, and the kind of
  need that would pull each piece in.
- **The maths** — the device's LSB/scale values, register layout, and the decode
  and encode formulas, pointing to the host-tested `<component>_convert` core.

It then links back to this document for the shared rationale.
