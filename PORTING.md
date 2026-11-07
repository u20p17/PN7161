# PN7160/PN7161 native ESP-IDF driver

Based on rednblkx/pn7160 at
`db5cb56b2ccfbaa4c30879043a3597bd6c7d53c8`. The upstream PN7160 class names
are retained. The local package name `PN7161` distinguishes it from the separate
Strooom PN7160 library.

This package contains the native NCI engine, `IPN7160Transport`, SPI transport,
and native I2C transport. It uses ESP-IDF GPIO, peripheral, timer and FreeRTOS
APIs and requires C++23 for the NCI engine's `std::expected` API.
There are no Arduino headers, Wire abstractions, reader facade or application
policy in its implementation. SPI and the NCI engine are unchanged.

See [native I2C ownership and validation](docs/native-i2c.md) and the independent
[ESP-IDF example](examples/native_i2c/main/main.cpp). Native I2C requires IDF 5.4+;
CMake retains the existing SPI sources for older supported SDKs.

The local PlatformIO manifest allows use from ESP-IDF and Arduino-ESP32 builds:
the latter already supplies the required ESP-IDF SDK. This compatibility metadata
does not add Arduino dependencies to the implementation.

The application reader facade, Wire bus bridge, application configuration,
ECP staging and PMU policy are maintained by the consuming application module,
outside this native package. Device authentication and access decisions also
remain outside the driver.
