project(tracing)

option(IMHEX_TRACE_EXCEPTIONS "Hook thrown exceptions to display a stack trace when possible" ON)

add_library(tracing OBJECT
    source/stacktrace.cpp
    source/exceptions.cpp
)
target_include_directories(tracing PUBLIC include)

if (NOT IMHEX_DISABLE_STACKTRACE)
    if (CMAKE_CXX_COMPILER_ID MATCHES "GNU")
        target_link_libraries(tracing PRIVATE stdc++exp)
    endif()

    if (WIN32)
        message(STATUS "StackWalk enabled!")
        target_link_libraries(tracing PRIVATE DbgHelp.lib)
    else ()
        find_package(Backtrace)
        if (${Backtrace_FOUND})
            message(STATUS "Backtrace enabled! Header: ${Backtrace_HEADER}")

            if (Backtrace_HEADER STREQUAL "backtrace.h")
                target_link_libraries(tracing PRIVATE ${Backtrace_LIBRARY})
                target_include_directories(tracing PRIVATE ${Backtrace_INCLUDE_DIR})
                target_compile_definitions(tracing PRIVATE BACKTRACE_HEADER=<${Backtrace_HEADER}>)
                target_compile_definitions(tracing PRIVATE HEX_HAS_BACKTRACE)
            elseif (Backtrace_HEADER STREQUAL "execinfo.h")
                target_link_libraries(tracing PRIVATE  ${Backtrace_LIBRARY})
                target_include_directories(tracing PRIVATE ${Backtrace_INCLUDE_DIR})
                target_compile_definitions(tracing PRIVATE BACKTRACE_HEADER=<${Backtrace_HEADER}>)
                target_compile_definitions(tracing PRIVATE HEX_HAS_EXECINFO)
            endif()
        endif()
    endif()

    target_link_libraries(tracing PRIVATE LLVMDemangle)
endif()

if (IMHEX_TRACE_EXCEPTIONS)
    if (CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
        target_link_options(tracing PUBLIC "-Wl,--wrap=__cxa_throw")
    endif()
endif()
