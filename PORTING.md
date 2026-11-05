# Local OpenKNX PN7161 build wrapper

Branch: `v1dev_HomeKey`. Upstream: https://github.com/rednblkx/pn7160
at `db5cb56b2ccfbaa4c30879043a3597bd6c7d53c8`, matching HomeKey-ESP32.

`library.json` registers the unchanged upstream SPI/NCI source as the distinct
PlatformIO library `PN7161`. Upstream C++ names remain PN7160. This separates
it from the existing Strooom PN7160 I2C library. The Home Key application owns
the reader through the extracted `Pn7160Reader` adapter in OFM-HomeKey.

Requires the ESP32 Arduino core's IDF SPI/GPIO/timer APIs and C++23
(`std::expected`). The prototype uses SPI2 on XIAO ESP32-S3. It compiles and
links with Xtensa GCC 14.2; physical PN7161 wiring and RF behavior still need
verification. No changes to the legacy I2C reader or RP2040 build.


The Arduino PN7161 package now owns `Pn7161Reader` and `Pn7161I2cTransport`,
including NCI discovery, presence/recovery and APDU exchange. Reader configuration
is explicit (shared Wire bus, GPIOs, address, clock, VANT or SPI pins), with no
OpenKNX/HomeKey dependency. Optional ECP is copied and disabled by default.
`setEcpFrame(nullptr, 0)` disables it; nonempty frames must be 18 bytes with CRC-A.
The NCI frame uses the first 16 bytes because the controller appends CRC-A.
