if(WITH_RUST)
    set(ALL_DATA
        ${DATA_BIN_DIR}/tsi.dat
        ${DATA_BIN_DIR}/word.dat
    )
else()
    set(ALL_DATA
        ${DATA_BIN_DIR}/dictionary.dat
        ${DATA_BIN_DIR}/index_tree.dat
    )
endif()

add_custom_target(data ALL DEPENDS ${ALL_DATA})

# We need to copy static data to binary tree when using out of tree build.
set(ALL_STATIC_DATA
    ${DATA_BIN_DIR}/pinyin.tab
    ${DATA_BIN_DIR}/swkb.dat
    ${DATA_BIN_DIR}/symbols.dat
)

add_custom_target(all_static_data
    COMMAND ${CMAKE_COMMAND} -E copy_if_different ${DATA_SRC_DIR}/pinyin.tab ${DATA_BIN_DIR}/pinyin.tab
    COMMAND ${CMAKE_COMMAND} -E copy_if_different ${DATA_SRC_DIR}/swkb.dat ${DATA_BIN_DIR}/swkb.dat
    COMMAND ${CMAKE_COMMAND} -E copy_if_different ${DATA_SRC_DIR}/symbols.dat ${DATA_BIN_DIR}/symbols.dat
)

# tools
set(ALL_TOOLS init_database)

if(NOT WITH_RUST)
    list(APPEND ALL_TOOLS dump_database)
    add_executable(init_database ${TOOLS_SRC_DIR}/init_database.c $<TARGET_OBJECTS:common>)
    add_executable(dump_database
        ${TOOLS_SRC_DIR}/dump_database.c
        $<TARGET_OBJECTS:common>
    )
    set_target_properties(${ALL_TOOLS} PROPERTIES
            RUNTIME_OUTPUT_DIRECTORY ${TOOLS_BIN_DIR}
            RUNTIME_OUTPUT_DIRECTORY_DEBUG ${TOOLS_BIN_DIR}
            RUNTIME_OUTPUT_DIRECTORY_MINSIZEREL ${TOOLS_BIN_DIR}
            RUNTIME_OUTPUT_DIRECTORY_RELEASE ${TOOLS_BIN_DIR}
            RUNTIME_OUTPUT_DIRECTORY_RELWITHDEBINFO ${TOOLS_BIN_DIR}
    )
endif()

# tools command
file(MAKE_DIRECTORY ${DATA_BIN_DIR})
if(WITH_RUST)
    set(DATA_COPYRIGHT "Copyright \\(c\\) 2022 libchewing Core Team")
    set(DATA_LICENSE "LGPL-2.1-or-later")
    set(DATA_VERSION ${LIBCHEWING_VERSION})
    add_custom_command(
        OUTPUT
            ${ALL_DATA}
        COMMAND chewing-cli init-database
            -c ${DATA_COPYRIGHT}
            -l ${DATA_LICENSE}
            -r ${DATA_VERSION}
            -t trie
            -n 內建詞庫
            ${DATA_SRC_DIR}/tsi.src tsi.dat
        COMMAND chewing-cli init-database
            -c ${DATA_COPYRIGHT}
            -l ${DATA_LICENSE}
            -r ${DATA_VERSION}
            -t trie
            -n 內建字庫
            ${DATA_SRC_DIR}/word.src word.dat
        DEPENDS
            chewing-cli
            ${DATA_SRC_DIR}/word.src
            ${DATA_SRC_DIR}/tsi.src
        WORKING_DIRECTORY ${DATA_BIN_DIR}
    )
else()
    add_custom_command(
        OUTPUT
            ${ALL_DATA}
        COMMAND init_database ${DATA_SRC_DIR}/phone.cin ${DATA_SRC_DIR}/tsi.src
        DEPENDS
            ${ALL_TOOLS}
            ${DATA_SRC_DIR}/phone.cin
            ${DATA_SRC_DIR}/tsi.src
        WORKING_DIRECTORY ${DATA_BIN_DIR}
    )
endif()

install(FILES ${ALL_DATA} DESTINATION ${CMAKE_INSTALL_DATADIR}/libchewing)
install(FILES ${ALL_STATIC_DATA} DESTINATION ${CMAKE_INSTALL_DATADIR}/libchewing)

