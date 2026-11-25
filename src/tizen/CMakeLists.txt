add_executable(tsl-wasm app.cpp player.cpp)
target_include_directories(tsl-wasm PRIVATE ${PROJECT_SOURCE_DIR}/src)
target_link_libraries(tsl-wasm PRIVATE tsl-core opus)
set_target_properties(tsl-wasm PROPERTIES OUTPUT_NAME tsl SUFFIX ".js")
# Link flags follow Moonlight for Tizen, which runs on the same Samsung toolchain.
target_link_options(tsl-wasm PRIVATE
        --bind
        -pthread
        "SHELL:-s USE_PTHREADS=1"
        "SHELL:-s PTHREAD_POOL_SIZE=16"
        "SHELL:-s TOTAL_MEMORY=134217728"
        "SHELL:-s TOTAL_STACK=8388608"
        "SHELL:-s NO_EXIT_RUNTIME=1"
        "SHELL:-s WASM=1"
        "SHELL:-s ENVIRONMENT_MAY_BE_TIZEN"
        "SHELL:-s EXTRA_EXPORTED_RUNTIME_METHODS=['UTF8ToString']")

# Debugging heap corruption: -g3 at link time makes this Emscripten pick its debug
# dlmalloc, which checks the heap's bookkeeping on every malloc and free and aborts at
# the first broken chunk. Slower and bigger, so never for a release.
option(TSL_DEBUG_MALLOC "Link the TV app with Emscripten's debug malloc" OFF)
if(TSL_DEBUG_MALLOC)
    target_link_options(tsl-wasm PRIVATE -g3)
endif()

set(TSL_WIDGET_DIR widget)
install(FILES
        ${CMAKE_CURRENT_BINARY_DIR}/tsl.js
        ${CMAKE_CURRENT_BINARY_DIR}/tsl.wasm
        ${CMAKE_CURRENT_BINARY_DIR}/tsl.worker.js
        DESTINATION ${TSL_WIDGET_DIR})
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/tsl.js.mem DESTINATION ${TSL_WIDGET_DIR} OPTIONAL)
install(DIRECTORY ${PROJECT_SOURCE_DIR}/web/ DESTINATION ${TSL_WIDGET_DIR}
        PATTERN "dev" EXCLUDE)
