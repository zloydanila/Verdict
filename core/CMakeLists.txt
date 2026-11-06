add_library(scanner-core STATIC
    src/crawler.cpp
    src/scan_orchestrator.cpp
    src/reporters/json_reporter.cpp
    src/reporters/sarif_reporter.cpp
    src/reporters/text_reporter.cpp
)

add_library(scanner::core ALIAS scanner-core)

target_include_directories(scanner-core
    PUBLIC $(CMAKE_CURRENT_SOURCE_DIR)/include
)

target_link_libraries(scanner-core
    PUBLIC scanner::modules
)

#Сделать core->modules PRIVATE, когда заголовки core перестанут их упоминать