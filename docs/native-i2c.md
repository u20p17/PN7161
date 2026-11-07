# Native ESP-IDF I2C transport (local test candidate)

`include/i2c.hpp` and `src/i2c.cpp` implement `IPN7160Transport` using only
ESP-IDF GPIO, FreeRTOS, and the synchronous `i2c_master` API. Minimum SDK: 5.4.
SPI and the NCI engine are unchanged. CMake includes I2C on supported SDKs
and retains C++23. The Arduino reader facade and Wire bridge live in the
consuming application module, outside this native driver package.

## Ownership and scheduling

The caller supplies an already-created master bus and must keep it alive until
after transport destruction. The transport registers/removes only its device;
it never creates, deletes, reinitializes or remaps a bus. The IRQ/VEN/optional
DWL pins are exclusive to this transport and must differ from SDA/SCL and other
peripherals. IRQ is polled by level, at one RTOS tick intervals, without an ISR.
VEN changes include a 10 ms settling delay, matching the SPI transport.
VEN is optional; without it, recovery requires an external reset/power cycle.

Stop all NCI users before transport deinit/destruction. Init/deinit and other
bus/device lifecycle changes require external serialization. Synchronous
transfers on different device handles are serialized by IDF's bus semaphore;
this does not make arbitrary bus reconfiguration safe. Do not attach asynchronous
callbacks, access the NFC address through a second driver, or probe/scan it.

Reads retain NCI's separate 3-byte header and up-to-255-byte payload transfers.
Each transfer permits 1..258 bytes. Failed reads clear the output. A transaction
has a finite configurable timeout (default 20 ms). NACKs on supported SDKs get
at most three attempts with 5 ms gaps, rounded up to a tick. Writes also wait
up to 20 ms per attempt for IRQ to fall; allow this extra time in caller budgets.
Timeouts and unknown errors are returned immediately without replaying a frame.
On IDF 5.4/5.5 ACK errors are reported as ESP_ERR_INVALID_STATE; IDF 6 uses
ESP_ERR_INVALID_RESPONSE. The caller must not remove/reconfigure the bus/device
behind the transport, which could otherwise invalidate that distinction.

## Standalone example

With an ESP-IDF 5.4+ installation and a C++23-capable toolchain:

```sh
cd examples/native_i2c
idf.py set-target esp32
idf.py build
idf.py -p YOUR_SERIAL_PORT flash monitor
```

Review `main/main.cpp` wiring before flashing. The example owns its own bus,
initializes NCI (CORE_RESET + CORE_INIT), logs the result, destroys the transport,
and only then deletes its bus. It does not enroll credentials or enable ECP.
Use suitable external pull-ups and 3.3 V signaling.

## Host tests

From this library's root (no Arduino headers or stubs are used):

```sh
clang++ -std=c++17 -Wall -Wextra -Werror \
  -Itest/native_i2c/stubs -Iinclude src/i2c.cpp \
  test/native_i2c/transport_test.cpp -o /tmp/pn7160-native-test
/tmp/pn7160-native-test
```

These tests cover lifecycle/partial initialization, borrowed-bus preservation,
header/payload framing, maximum frames, IRQ levels/delays/wraparound, bounded
NACK retries, failed-read clearing, and no timeout replay. They simulate driver
results; they do not prove electrical framing, a second physical peripheral,
concurrent hardware operation, or compatibility with every PN716x firmware.

The standalone `idf.py` build and physical hardware tests are pending. The native
source is also compiled against the actual IDF 5.5.4 SDK by the local firmware
test build, including unchanged SPI and NCI sources. No replacement PR is ready
for publication until hardware validation is recorded.
