add_library(scanner-infra STATIC
    src/finding.cpp
    src/html_parser.cpp
    src/http_client.cpp
    src/no_op_external_lookup.cpp
    src/rate_limiter.cpp
    src/tls_inspector.cpp
    src/url.cpp
)

add_library(scanner::infra ALIAS scanner-infra)

target_include_directories(scanner-infra
    PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include
)