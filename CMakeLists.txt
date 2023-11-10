cmake_minimum_required(VERSION 3.14)

project(tower_defense VERSION 1.0.0 LANGUAGES CXX)

if(PROJECT_SOURCE_DIR STREQUAL PROJECT_BINARY_DIR)
  message(FATAL_ERROR "In-source builds not allowed. Please make a new directory (called a build directory) and run CMake from there.")
endif()

##################################
########## EXECUTABLE ############
##################################

file(GLOB_RECURSE HEADERS CONFIGURE_DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/include/*.h")
file(GLOB_RECURSE SOURCES CONFIGURE_DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp")
add_executable(${PROJECT_NAME} ${HEADERS} ${SOURCES})
target_include_directories(${PROJECT_NAME} PUBLIC ${PROJECT_SOURCE_DIR}/include)
set_target_properties(${PROJECT_NAME} PROPERTIES C_STANDARD 99)

if (EMSCRIPTEN)
	target_link_options(${PROJECT_NAME} PUBLIC
    -sMODULARIZE
    "SHELL: -s 'EXPORT_NAME=${PROJECT_NAME}'"
  )
endif()

##################################
########## DEPENDENCIES ##########
##################################

if((NOT DEFINED BUILD_SHARED_LIBS) OR (NOT ${BUILD_SHARED_LIBS}))
  set(STR_IS_STATIC_LIBRARY "ON")
  set(STR_IS_SHARED_LIBRARY "OFF")
else()
  set(STR_IS_STATIC_LIBRARY "OFF")
  set(STR_IS_SHARED_LIBRARY "ON") 
endif()

include(cmake/get_cpm.cmake)

CPMAddPackage(
  VERSION 3.2.8
  GITHUB_REPOSITORY SanderMertens/flecs
  OPTIONS "FLECS_STATIC ${STR_IS_STATIC_LIBRARY}"
          "FLECS_SHARED ${STR_IS_SHARED_LIBRARY}"
)

if((NOT DEFINED BUILD_SHARED_LIBS) OR (NOT ${BUILD_SHARED_LIBS}))
  target_link_libraries(${PROJECT_NAME} PRIVATE flecs::flecs_static)
else()
  target_link_libraries(${PROJECT_NAME} PRIVATE flecs::flecs)
endif()

CPMAddPackage("gh:flecs-hub-cmake/flecs-components-transform#83ebab3f8ad86a3f76cd10a8affcc335789af5b9")
target_link_libraries(${PROJECT_NAME} PRIVATE flecs_hub::components::transform)

CPMAddPackage("gh:flecs-hub-cmake/flecs-components-graphics#4a8722f770ab1ea300935da9a7043f9d826870e6")
target_link_libraries(${PROJECT_NAME} PRIVATE flecs_hub::components::graphics)

CPMAddPackage("gh:flecs-hub-cmake/flecs-components-geometry#ae45f5a2433804a9d1e6e62dad55b72a4618bcf4")
target_link_libraries(${PROJECT_NAME} PRIVATE flecs_hub::components::geometry)

CPMAddPackage("gh:flecs-hub-cmake/flecs-components-physics#9ebfd5ec36afae401f63e82ffc37a46de8e888c2")
target_link_libraries(${PROJECT_NAME} PRIVATE flecs_hub::components::physics)

CPMAddPackage("gh:flecs-hub-cmake/flecs-components-gui#913e6849988f4b43ecbd85451289c4fc46b82094")
target_link_libraries(${PROJECT_NAME} PRIVATE flecs_hub::components::gui)

CPMAddPackage("gh:flecs-hub-cmake/flecs-components-input#2d60f0045741c588ba45cb3380a0b7a5a1a4732c")
target_link_libraries(${PROJECT_NAME} PRIVATE flecs_hub::components::input)

CPMAddPackage("gh:flecs-hub-cmake/flecs-systems-transform#14a7add4672a5c83adbea7819c8649872ba10d1e")
target_link_libraries(${PROJECT_NAME} PRIVATE flecs_hub::systems::transform)

CPMAddPackage("gh:flecs-hub-cmake/flecs-systems-sokol#dc1bbe69c0978c248733365a090922e0b173cd4e") 
target_link_libraries(${PROJECT_NAME} PRIVATE flecs_hub::systems::sokol)

CPMAddPackage("gh:flecs-hub-cmake/flecs-systems-physics#8bea5131c251462c390bd08851661b7b4d4af24e")
target_link_libraries(${PROJECT_NAME} PRIVATE flecs_hub::systems::physics)

CPMAddPackage("gh:flecs-hub-cmake/flecs-game#af1f5dd4aca4ae93d25096e590c14b320a3ae38e")
target_link_libraries(${PROJECT_NAME} PRIVATE flecs_hub::game)
