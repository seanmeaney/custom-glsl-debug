
add_subdirectory(glcpp)
include_directories(
	"${CMAKE_CURRENT_BINARY_DIR}"
	"${CMAKE_CURRENT_SOURCE_DIR}"
	"${CMAKE_CURRENT_SOURCE_DIR}/glcpp"
	"${PROJECT_SOURCE_DIR}/glsldb"
)

if (MSVC)
	include_directories("${CMAKE_CURRENT_SOURCE_DIR}/../include/c99")
endif()

find_package(BISON REQUIRED)
find_package(FLEX REQUIRED)

mark_as_advanced(YACC_BINARY FLEX_BINARY)
add_definitions(-DYYDEBUG=1)

add_custom_command(OUTPUT "${CMAKE_CURRENT_BINARY_DIR}/glsl_lexer.cpp"
        SOURCE "${CMAKE_CURRENT_SOURCE_DIR}/glsl_lexer.ll"
        COMMAND ${FLEX_EXECUTABLE}
        ARGS --nounistd -o"${CMAKE_CURRENT_BINARY_DIR}/glsl_lexer.cpp"
             "${CMAKE_CURRENT_SOURCE_DIR}/glsl_lexer.ll"
)

# add_custom_command(TARGET "${CMAKE_CURRENT_BINARY_DIR}/glsl_parser.cpp" "${CMAKE_CURRENT_BINARY_DIR}/glsl_parser.h"
# 	COMMAND rm
# 	ARGS "${CMAKE_CURRENT_BINARY_DIR}/glsl_parser.cpp" "${CMAKE_CURRENT_BINARY_DIR}/glsl_parser.h"
# )

add_custom_command(OUTPUT "${CMAKE_CURRENT_BINARY_DIR}/glsl_parser.cpp" "${CMAKE_CURRENT_BINARY_DIR}/glsl_parser.h"
        SOURCE "${CMAKE_CURRENT_SOURCE_DIR}/glsl_parser.yy"
        COMMAND ${BISON_EXECUTABLE}
        ARGS -t -v -o "${CMAKE_CURRENT_BINARY_DIR}/glsl_parser.cpp"
             -p "_mesa_glsl_"
             --defines="${CMAKE_CURRENT_BINARY_DIR}/glsl_parser.h"
             "${CMAKE_CURRENT_SOURCE_DIR}/glsl_parser.yy"
        DEPENDS "${CMAKE_CURRENT_BINARY_DIR}/glsl_lexer.cpp"
)

set_source_files_properties("${CMAKE_CURRENT_BINARY_DIR}/glsl_lexer.cpp" GENERATED)
set_source_files_properties("${CMAKE_CURRENT_BINARY_DIR}/glsl_parser.h" GENERATED)
set_source_files_properties("${CMAKE_CURRENT_BINARY_DIR}/glsl_parser.cpp" GENERATED)


file(GLOB SRC_CPP *.cpp)
file(GLOB SRC_C *.c)
file(GLOB SRC_CPP_REMOVE main.cpp)
list(REMOVE_ITEM SRC_CPP ${SRC_CPP_REMOVE})
set(SRC ${SRC_C} ${SRC_CPP}
	"${CMAKE_CURRENT_BINARY_DIR}/glsl_lexer.cpp"
	"${CMAKE_CURRENT_BINARY_DIR}/glsl_parser.cpp"
)
add_library(glsl_lib ${SRC})
target_link_libraries(glsl_lib glsl_glcpp_lib mesa_lib)

if(NOT GLSLDB_WIN)
	add_executable(glsl_compiler main.cpp)
	target_link_libraries(glsl_compiler glsl_lib)
endif()

#add_executable(glsl_glcpp glcpp/glcpp.c)
#target_link_libraries(glsl_glcpp glsl_lib)
