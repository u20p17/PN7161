idf_component_register(SRCS "main.cpp" INCLUDE_DIRS "."
    REQUIRES ${PN7160_COMPONENT_NAME} esp_driver_i2c esp_driver_gpio)
target_compile_features(${COMPONENT_LIB} PRIVATE cxx_std_23)
