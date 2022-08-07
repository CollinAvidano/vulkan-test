cmake_minimum_required(VERSION 3.16)

set(CMAKE_CXX_STANDARD 14)

project(renderer)

find_package(Vulkan REQUIRED)
find_package(glfw3 3.3 REQUIRED)

file(GLOB_RECURSE SRC "src/*.cpp")

add_executable(${PROJECT_NAME} ${SRC})
target_include_directories(${PROJECT_NAME} PUBLIC include)
target_link_libraries(${PROJECT_NAME} PUBLIC glfw Vulkan::Vulkan)
set_property(TARGET ${PROJECT_NAME} PROPERTY CXX_STANDARD 17)

# Shaders
function(compile_shaders TARGET_NAME)
    if(NOT ARGN)
        return()
    endif()
    foreach(SHADER ${ARGN})
        get_filename_component(FILE_NAME ${SHADER} NAME)
        set(output_file ${CMAKE_CURRENT_BINARY_DIR}/shaders/${FILE_NAME}.spv)
        # sets compiled shaders both in the current scope and the global scope
        set(compiled_shaders ${compiled_shaders} ${output_file})
        set(compiled_shaders ${compiled_shaders} PARENT_SCOPE)
        add_custom_command(
            OUTPUT ${output_file}
            COMMAND mkdir -p shaders
            COMMAND ${GLSLANG_VALIDATOR_VALIDATOR_EXECUTABLE} -w -V ${SHADER} -o ${output_file}
            #DEPENDS ${CMAKE_SOURCE_DIR}/shaders/${SHADER}
            DEPENDS ${SHADER}
            COMMENT "Compiling shader ${output_file}"
        )
    endforeach()
    add_custom_target(shaders-${TARGET_NAME} ALL DEPENDS ${compiled_shaders})
    add_dependencies(${TARGET_NAME} shaders-${TARGET_NAME})
endfunction()


file(GLOB_RECURSE SHADERS "shaders/*.vert" "shaders/*.frag")
compile_shaders(${PROJECT_NAME} ${SHADERS})
