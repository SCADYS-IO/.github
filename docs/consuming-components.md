# Consuming SCADYS-IO components

SCADYS-IO publishes its reusable firmware as **ESP-IDF components**, written in
**C** with `extern "C"` headers. Drivers take ESP-IDF handles directly (e.g. an
`i2c_master_bus_handle_t`) rather than wrapping them — ESP-IDF 5.x already
provides the bus drivers, FreeRTOS, `esp_event`, `esp_err_t`, and `esp_log`, so
the components add only what IDF does not ship.

This guide covers consuming a component from each supported platform, and using a
C component from a C++ application.

These are the shared standards every SCADYS-IO component points to; the canonical
copy lives in the [`SCADYS-IO/.github`](https://github.com/SCADYS-IO/.github)
organization profile repository.

---

## ESP-IDF (native)

Add the component to your project's (or main component's) `idf_component.yml`:

```yaml
dependencies:
  i2c_scan:
    git: https://github.com/SCADYS-IO/i2c_scan.git
    version: "*"
```

The IDF Component Manager fetches it on the next `idf.py build`. During local
development against a sibling checkout, use `override_path` instead of `git`:

```yaml
dependencies:
  i2c_scan:
    version: "*"
    override_path: "../../components/i2c_scan"
```

Then include and use it:

```c
#include "i2c_scan.h"
```

## PlatformIO (`framework = espidf`)

PlatformIO treats the project's `src/` directory as the ESP-IDF "main" component
and installs dependencies declared in `src/idf_component.yml` via the Component
Manager. A minimal `platformio.ini`:

```ini
[env:esp32-s3-devkitc-1]
platform = espressif32
board = esp32-s3-devkitc-1
framework = espidf
```

`src/idf_component.yml`:

```yaml
dependencies:
  i2c_scan:
    git: https://github.com/SCADYS-IO/i2c_scan.git
    version: "*"
```

Use a platform version that provides **ESP-IDF >= 5.4**.

> **Windows — no spaces in any path.** ESP-IDF's build system refuses paths that
> contain a space (`Error: Detected a whitespace character in project paths`).
> This bites two places on Windows: the **project location** and PlatformIO's
> **package cache**, which defaults to `%USERPROFILE%\.platformio` — so a
> username with a space (`C:\Users\Jane Smith\`) breaks the build even when the
> project path is clean. Relocate the cache to a space-free path before building:
>
> ```powershell
> $env:PLATFORMIO_CORE_DIR = "D:\pio"   # any space-free path
> pio run
> ```
>
> The same whitespace rule applies to a native ESP-IDF install: keep ESP-IDF and
> your projects on space-free paths (e.g. `D:\esp`, not `C:\Users\Jane Smith\`).

## Arduino (arduino-esp32 v3.0+)

The components are ESP-IDF-first. They are usable under Arduino **only** on the
**arduino-esp32 core v3.0.0 or newer**, which is built on ESP-IDF 5.x and exposes
the ESP-IDF drivers (e.g. `driver/i2c_master.h`). They drive the ESP-IDF driver
directly — they do **not** use `Wire`.

Because they are published as ESP-IDF components (not yet Arduino Library Manager
packages), install one by making its `include/` + `src/` available to the sketch
as a local Arduino library. Arduino Library Manager packaging is a planned
follow-up.

If your sketch needs a `Wire`-native API, that is a separate (optional) addition
on the component, not the default surface.

---

## Using a C component from a C++ application

SCADYS-IO components are C, but consuming them from C++ (including the SCADYS
firmware apps, which are C++17 with exceptions and RTTI disabled) needs **nothing
special**:

- **Just include the header.** Every public header wraps its declarations in
  `extern "C" { ... }`, so the C++ compiler links against the C symbols without
  name-mangling mismatches:

  ```cpp
  #include "i2c_scan.h"   // already extern "C"-guarded

  extern "C" void app_main(void);   // if your entry point is C-linkage

  void scan(i2c_master_bus_handle_t bus) {
      i2c_scan_result_t result;
      if (i2c_scan(bus, 50, &result) == ESP_OK) {
          // result.count devices found
      }
  }
  ```

- **No exceptions cross the boundary.** The components report failures with
  `esp_err_t` return codes, never C++ exceptions, so they are safe under
  `-fno-exceptions`. RTTI is irrelevant to a C ABI, so `-fno-rtti` is also fine.

- **Pass C types.** Use the plain structs and handles from the component's
  header (`i2c_scan_result_t`, `i2c_master_bus_handle_t`). Don't pass C++ objects,
  references, or `std::` types across the boundary.

- **Linking is automatic** under both ESP-IDF and PlatformIO — the component
  builds to a static library that the linker resolves into the C++ app.

If you are writing a new SCADYS-IO component, keep its public header C-compatible
and `extern "C"`-guarded so it stays consumable from both C and C++.
