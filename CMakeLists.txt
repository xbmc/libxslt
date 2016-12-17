cmake_minimum_required(VERSION 3.2)

project(libxslt LANGUAGES C)

option(BUILD_SHARED_LIBS "Build as shared lib instead of static" ON)
option(WITH_TRIO "Enable TRIO string manipulator" ON)
option(WITH_ICONV "Enable iconv support" ON)
option(WITH_DEBUG "Enable XML debugging module" ON)
option(WITH_MEM_DEBUG "Enable memory debugger" OFF)
option(WITH_RUN_DEBUG "Enable memory debugger" OFF)


include(CheckIncludeFile)
include(CheckFunctionExists)

CHECK_INCLUDE_FILE(nan.h HAVE_NAN_H)
CHECK_INCLUDE_FILE(ctype.h HAVE_CTYPE_H)
CHECK_INCLUDE_FILE(dirent.h HAVE_DIRENT_H)
CHECK_INCLUDE_FILE(dl.h HAVE_DL_H)
CHECK_INCLUDE_FILE(errno.h HAVE_ERRNO_H)
CHECK_INCLUDE_FILE(fcntl.h HAVE_FCNTL_H)
CHECK_INCLUDE_FILE(ieeefp.h HAVE_IEEEP_H)
CHECK_INCLUDE_FILE(inttypes.h HAVE_INTTYPES_H)
CHECK_INCLUDE_FILE(limits.h HAVE_LIMITS_H)
CHECK_INCLUDE_FILE(malloc.h HAVE_MALLOC_H)
CHECK_INCLUDE_FILE(math.h HAVE_MATH_H)
CHECK_INCLUDE_FILE(memory.h HAVE_MEMORY_H)
CHECK_INCLUDE_FILE(time.h HAVE_TIME_H)
CHECK_INCLUDE_FILE(unistd.h HAVE_UNISTD_H)
CHECK_INCLUDE_FILE(locale.h HAVE_LOCALE_H)
CHECK_INCLUDE_FILE(xlocale.h HAVE_XLOCALE_H)
CHECK_INCLUDE_FILE(stdarg.h HAVE_STDARG_H)
CHECK_INCLUDE_FILE(stdint.h HAVE_STDINT_H)
CHECK_INCLUDE_FILE(stdlib.h HAVE_STDLIB_H)
CHECK_INCLUDE_FILE(strings.h HAVE_STRINGS_H)
CHECK_INCLUDE_FILE(string.h HAVE_STRING_H)
CHECK_INCLUDE_FILE(sys/select.h HAVE_SYS_SELECT_H)
CHECK_INCLUDE_FILE(sys/stat.h HAVE_SYS_STAT_H)
CHECK_INCLUDE_FILE(sys/timeb.h HAVE_SYS_TIMEB_H)
CHECK_INCLUDE_FILE(sys/time.h HAVE_SYS_TIME_H)
CHECK_INCLUDE_FILE(sys/types.h HAVE_SYS_TYPES_H)
CHECK_INCLUDE_FILE(ansidecl.h HAVE_ANSIDECL_H)
CHECK_INCLUDE_FILE(float.h HAVE_FLOAT_H)
CHECK_INCLUDE_FILE(fp_class.h HAVE_FP_CLASS_H)


if(MSVC)
  set(CMAKE_DEBUG_POSTFIX "d")
  set(XSLT_LOCALE_WINAPI ON)
endif()

set(libxslt_VERSION 1.1.29)
set(VERSION ${libxslt_VERSION})
set(LIBXSLT_VERSION_NUMBER 10129)
set(LIBXSLT_VERSION_EXTRA "")

configure_file(
  config.h.cmake
  ${CMAKE_CURRENT_BINARY_DIR}/config.h
)
configure_file(
  libxslt/xsltconfig.h.in
  ${CMAKE_CURRENT_BINARY_DIR}/libxslt/xsltconfig.h)

configure_file(
  libexslt/exsltconfig.h.in
  ${CMAKE_CURRENT_BINARY_DIR}/libexslt/exsltconfig.h)

set(XSLT_HEADERS
  ${CMAKE_CURRENT_BINARY_DIR}/libxslt/xsltconfig.h
  libxslt/xsltwin32config.h
  libxslt/xsltutils.h
  libxslt/xsltlocale.h
  libxslt/xsltInternals.h
  libxslt/xsltexports.h
  libxslt/xslt.h
  libxslt/win32config.h
  libxslt/variables.h
  libxslt/triodef.h
  libxslt/trio.h
  libxslt/transform.h
  libxslt/templates.h
  libxslt/security.h
  libxslt/preproc.h
  libxslt/pattern.h
  libxslt/numbersInternals.h
  libxslt/namespaces.h
  libxslt/libxslt.h
  libxslt/keys.h
  libxslt/imports.h
  libxslt/functions.h
  libxslt/extra.h
  libxslt/extensions.h
  libxslt/documents.h
  libxslt/attributes.h
)

set(EXSLT_HEADERS
  ${CMAKE_CURRENT_BINARY_DIR}/libexslt/exsltconfig.h
  libexslt/exslt.h
  libexslt/exsltexports.h
  libexslt/libexslt.h
)
add_library(libxslt 
#generated files
  ${CMAKE_CURRENT_BINARY_DIR}/config.h
#libxslt
  ${XSLT_HEADERS}
  libxslt/attributes.c
  libxslt/xsltutils.c
  libxslt/xsltlocale.c
  libxslt/xslt.c
  libxslt/variables.c
  libxslt/transform.c
  libxslt/templates.c
  libxslt/security.c
  libxslt/preproc.c
  libxslt/pattern.c
  libxslt/numbers.c
  libxslt/namespaces.c
  libxslt/keys.c
  libxslt/imports.c
  libxslt/functions.c
  libxslt/extra.c
  libxslt/extensions.c
  libxslt/documents.c
  libxslt/attrvt.c
)
add_library(libexslt STATIC
  ${EXSLT_HEADERS}
  libexslt/common.c
  libexslt/crypto.c
  libexslt/date.c
  libexslt/dynamic.c
  libexslt/exslt.c
  libexslt/functions.c
  libexslt/math.c
  libexslt/saxon.c
  libexslt/sets.c
  libexslt/strings.c
)
target_include_directories(
  libxslt PRIVATE
  $<BUILD_INTERFACE:.;libxslt;${CMAKE_CURRENT_BINARY_DIR}>
  INTERFACE
  $<INSTALL_INTERFACE:include/libxslt>
)

target_include_directories(
  libexslt PRIVATE
  $<BUILD_INTERFACE:.;libxslt;libexslt;${CMAKE_CURRENT_BINARY_DIR}>
)
target_compile_definitions(libxslt 
  PRIVATE 
  _CRT_SECURE_NO_WARNINGS
  _CRT_STDC_NO_DEPRECATE)

add_dependencies(libxslt libexslt)

if(WITH_ICONV)
  if(!UNIFIED_BUILD)
    find_package(iconv REQUIRED NO_MODULE)
  endif()
  target_link_libraries(libxslt PRIVATE iconv::iconv)
  target_link_libraries(libexslt PRIVATE iconv::iconv)
endif()

if (!UNIFIED_BUILD)
  find_package(libxml2 REQUIRED NO_MODULE)
endif()
target_link_libraries(libxslt PRIVATE libxml2::libxml2)
target_link_libraries(libexslt PRIVATE libxml2::libxml2)

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
  ${CMAKE_CURRENT_BINARY_DIR}/libxslt-config-version.cmake
  VERSION ${libxslt_VERSION}
  COMPATIBILITY AnyNewerVersion
)

install(TARGETS libxslt EXPORT libxslt
  RUNTIME DESTINATION bin
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib)


install(FILES
  ${XSLT_HEADERS}
  ${EXSLT_HEADERS}
  DESTINATION include/libxslt)

install(EXPORT libxslt
  FILE
    libxslt.cmake
  NAMESPACE
    libxslt::
  DESTINATION
    lib/cmake/libxslt
)
install(
  FILES
    cmake/libxslt-config.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/libxslt-config-version.cmake
  DESTINATION
    lib/cmake/libxslt
)

if(MSVC)
  install(FILES
    $<TARGET_PDB_FILE:libxslt>
    DESTINATION lib
  )
endif()