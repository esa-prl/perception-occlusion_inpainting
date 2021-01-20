# PyTorch installation instructions: https://pytorch.org/cppdocs/installing.html
# inspiration for if clauses: https://github.com/ethz-asl/fw_planning/pull/58/files
find_package(Torch)
if (torch_FOUND)
    message("Compiling with Torch support")
    add_definitions(-DUSE_TORCH=1)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${TORCH_CXX_FLAGS}")
    include_directories(${TORCH_INCLUDE_DIRS})
else()
    message("Compiling without Torch support")
    add_definitions(-DUSE_TORCH=0)
endif()


rock_library(dummy_project
    SOURCES Dummy.cpp
    HEADERS Dummy.hpp)

rock_executable(dummy_project_bin Main.cpp
    DEPS dummy_project)

if (torch_FOUND)
    target_link_libraries(dummy_project_bin "${TORCH_LIBRARIES}")
    set_property(TARGET dummy_project_bin PROPERTY CXX_STANDARD 14)

    # The following code block is suggested to be used on Windows.
    # According to https://github.com/pytorch/pytorch/issues/25457,
    # the DLLs need to be copied to avoid memory errors.
    if (MSVC)
    file(GLOB TORCH_DLLS "${TORCH_INSTALL_PREFIX}/lib/*.dll")
    add_custom_command(TARGET example-app
                        POST_BUILD
                        COMMAND ${CMAKE_COMMAND} -E copy_if_different
                        ${TORCH_DLLS}
                        $<TARGET_FILE_DIR:example-app>)
    endif (MSVC)
endif()
