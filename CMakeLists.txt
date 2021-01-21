###############
### General ###
###############
cmake_minimum_required(VERSION 3.10...3.18)

project(occlusion_inpainting)

list(APPEND CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake")

set(TARGET_NAME ${PROJECT_NAME})
set(CMAKE_INCLUDE_DIR ${PROJECT_SOURCE_DIR})
set(CMAKE_SOURCE_DIR ${PROJECT_SOURCE_DIR}/${TARGET_NAME})

####################
### Dependencies ###
####################
find_package(Eigen3 REQUIRED)
find_package(OpenCV REQUIRED)
find_package(grid_map_core REQUIRED)

# PyTorch installation instructions: https://pytorch.org/cppdocs/installing.html
# inspiration for if clauses: https://github.com/ethz-asl/fw_planning/pull/58/files
find_package(Torch)
if (torch_FOUND)
    message("Compiling with Torch support")
    add_definitions(-DUSE_TORCH=1)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${TORCH_CXX_FLAGS}")
else()
    message("Compiling without Torch support")
    add_definitions(-DUSE_TORCH=0)
endif()

#############
### Build ###
#############
include_directories(
    ${CMAKE_INCLUDE_DIR}
    ${EIGEN3_INCLUDE_DIRS}
    ${OpenCV_INCLUDE_DIRS}
    ${grid_map_core_INCLUDE_DIRS}
)

if (torch_FOUND)
    include_directories(${TORCH_INCLUDE_DIRS})
endif()

add_library(${TARGET_NAME} SHARED
    ${CMAKE_SOURCE_DIR}/OcclusionInpainting.cpp
)

target_link_libraries(${TARGET_NAME}
    ${PCL_LIBRARIES}
    ${OpenCV_LIBS}
    ${grid_map_core_LIBRARIES}
)

add_executable(${TARGET_NAME}_bin ${CMAKE_SOURCE_DIR}/Main.cpp
               ${TARGET_NAME})


if (torch_FOUND)
    target_link_libraries(${TARGET_NAME} "${TORCH_LIBRARIES}")
    set_property(TARGET ${TARGET_NAME} PROPERTY CXX_STANDARD 14)

    # The following code block is suggested to be used on Windows.
    # According to https://github.com/pytorch/pytorch/issues/25457,
    # the DLLs need to be copied to avoid memory errors.
    if (MSVC)
    file(GLOB TORCH_DLLS "${TORCH_INSTALL_PREFIX}/lib/*.dll")
    add_custom_command(TARGET ${TARGET_NAME}
                      POST_BUILD
                      COMMAND ${CMAKE_COMMAND} -E copy_if_different
                      ${TORCH_DLLS}
                      $<TARGET_FILE_DIR:${TARGET_NAME})
    endif (MSVC)
endif()

###############
### Install ###
###############

install(
    TARGETS ${TARGET_NAME}
    LIBRARY DESTINATION lib
)
