# PN7161 Arduino ESP32 package

Based on rednblkx/pn7160 at
`db5cb56b2ccfbaa4c30879043a3597bd6c7d53c8`. The existing upstream SPI/NCI
sources and their PN7160 C++ names are retained. The local PlatformIO name
`PN7161` distinguishes this package from the separate Strooom PN7160 library.

## Ownership and interfaces

This package owns `Pn7161Reader` and `Pn7161I2cTransport`. The reader manages NCI
initialization, discovery, UID reads, ISO-DEP/APDU exchange, presence monitoring,
health checks and connection state. It can be used for ordinary UID tags without
HomeKey. It has no dependency on OpenKNX, HomeSpan or a credential store.

In OAM-AccessControl, OFM-AccessControl owns the shared NFC worker and reader
adapter. OFM-HomeKey optionally supplies authentication and ECP data; it does not
own the hardware reader task. Access decisions and presentation suppression
belong to the caller, not this driver.

## Transport configuration

`Pn7161Reader::Config` explicitly supplies hardware settings:

* Non-null `wire` selects I2C: shared `TwoWire`, SDA/SCL, IRQ, VEN, optional DWL,
  address (default `0x28`) and clock (default 400 kHz).
* Null `wire` selects SPI2: `spiPins` order is CS, SCLK, MISO, MOSI, plus IRQ/VEN.
* `vantMv` accepts 3300 or 5000 for explicit PMU configuration; zero retains the
  existing automatic PMU configuration. Choose this to match the actual supply.

I2C initializes/borrows the Arduino bus and removes only its own native device
handle on shutdown. It never calls `Wire.end()`. Other devices must share the
same bus pins and use different addresses. Bounded NACK retries handle controller
wake-up; non-retryable errors propagate to connection/recovery handling.

The package requires Arduino ESP32's IDF SPI/GPIO/timer APIs and a C++23 compiler
for the upstream `std::expected` API. IDF 5.4+ uses the native I2C bus-handle API;
an older Arduino I2C fallback remains in the source. Simulated API-branch tests
are not proof that every older Arduino/IDF release builds or works on hardware.

## Optional ECP

ECP is disabled by default. `setEcpFrame(frame, 18)` copies a complete frame
including CRC-A supplied by the caller; the setter checks its length, not its
CRC. The NCI configuration uses the first 16 bytes because the controller appends
CRC-A. `setEcpFrame(nullptr, 0)` disables ECP. Set it before `init()`, or call
`updateECP()` after changing it on an initialized reader to apply it to hardware.
The package does not generate HomeKey identities or authorize credentials.

## Integration and validation

The current REG1 v00.11 integration uses I2C: DWL GPIO19, SDA GPIO21, SCL GPIO22,
IRQ GPIO14 and VEN GPIO13, with VDD/VANT at 3.3 V and common GND. The historical
XIAO SPI prototype is not the current reference wiring.

Application regression tests exercise the actual I2C transport with simulated
IDF 5.3/5.4/5.5/6.0 APIs and the reader's asynchronous presence failure handling.
Routine firmware validation uses `debug_REG1_ESP_PN7161_IP`. Hardware checks are
still required for each board/transport, including cold boot, shared-bus recovery,
UID presentation/removal and iPhone/Watch authentication and Express Mode.

Reader facade code was adapted from rednblkx/HomeKey-ESP32; its MIT attribution
is retained in `LICENSE.HomeKey-ESP32`.
