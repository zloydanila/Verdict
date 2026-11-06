add_library(scanner-modules STATIC
    src/module_registry.cpp
    cookie_flags_check/cookie_flags_check.cpp
    cors_check/cors_check.cpp
    exposure_check/exposure_check.cpp
    headers_check/headers_check.cpp
    open_redirect_check/open_redirect_check.cpp
    sqli_check/sqli_check.cpp
    tls_check/tls_check.cpp
    verification/boolean_pair.cpp
    verification/double_marker.cpp
    verification/status_diff.cpp
    xss_check/xss_check.cpp
)

add_library(scanner::modules ALIAS scanner-modules)

target_include_directories(scanner-modules
    PUBLIC 
        $(CMAKE_CURRENT_SOURCE_DIR)/include
        $(CMAKE_CURRENT_SOURCE_DIR)
)

target_link_libraries(scanner-modules
    PUBLIC scanner::infra
)

#Надо перенести заголовки модулей в include/scanner/modules/ чтобы стиль #include совпадал с infra

