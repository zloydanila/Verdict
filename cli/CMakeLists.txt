add_executable(scanner-cli
    main.cpp
    cli_options.cpp
)

target_link_libraries(scanner-cli
    PRIVATE scanner::core
)

