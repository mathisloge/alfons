find_package(ICU REQUIRED COMPONENTS uc)
find_package(freetype CONFIG REQUIRED)
find_package(harfbuzz CONFIG REQUIRED)
find_package(glm CONFIG REQUIRED)


add_library(linebreak STATIC
  linebreak/linebreak.c
  linebreak/linebreakdata.c
  linebreak/linebreakdef.c
)

target_include_directories (linebreak PUBLIC
  $<INSTALL_INTERFACE:include/linebreak>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/linebreak>
)

set(ALFONS_SRC
  alfons/alfons.cpp
  alfons/fontManager.cpp
  alfons/fontFace.cpp
  alfons/langHelper.cpp
  alfons/font.cpp
  alfons/textBatch.cpp
  alfons/atlas.cpp
  alfons/textShaper.cpp
  alfons/quadMatrix.cpp
  alfons/path/lineSampler.cpp
  alfons/path/splinePath.cpp
  alfons/scrptrun.cpp
)

if (APPLE AND HB_HAVE_CORETEXT)
  list(APPEND ALFONS_SRC alfons/appleFontConverter/fontConverter.mm)
  list(APPEND ALFONS_SRC alfons/appleFontFace.mm)

  set(ALFONS_APPLE_LIBRARIES
    "-framework CoreGraphics"
    "-framework Foundation"
  )
  if(NOT IOS)
    set(ALFONS_APPLE_LIBRARIES ${ALFONS_APPLE_LIBRARIES} "-framework AppKit")
  endif()
  message(STATUS "ALFONS_APPLE_LIBRARIES: " ${ALFONS_APPLE_LIBRARIES})

  add_definitions(-DHAVE_CORETEXT)
endif()

add_library(alfons STATIC ${ALFONS_SRC})

target_include_directories (alfons PUBLIC
  $<INSTALL_INTERFACE:include>
  $<INSTALL_INTERFACE:include/logger>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/logger>
)

target_link_libraries(alfons PUBLIC
  linebreak
  ICU::uc
  glm::glm
  freetype
  harfbuzz::harfbuzz
  ${ALFONS_APPLE_LIBRARIES}
)

set_target_properties(alfons PROPERTIES CXX_STANDARD 11)

### check for undefined symbols
if(BUILD_SHARED_LIBS AND CMAKE_COMPILER_IS_GNUCXX)
  target_link_libraries(alfons PRIVATE "-z defs")
endif()

if(CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
  target_compile_options(alfons
    PRIVATE
    -D_USE_MATH_DEFINES
    -utf-8
    -wd4267
    -wd4244)
endif()
