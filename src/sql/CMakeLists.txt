add_definitions(
    -DQT_SQL_SQLITE
    -DQT_BUILD_SQL_LIB
    -DQT_SHARED
    -DQT_MAKEDLL
)
set(EXTRA_SQL_LIBS CsCore${BUILD_MAJOR})
set(EXTRA_SQL_CXXFLAGS)
set(EXTRA_SQL_LDFLAGS)

include_directories(
    ${CMAKE_BINARY_DIR}/include
    ${CMAKE_BINARY_DIR}/privateinclude
    ${CMAKE_BINARY_DIR}/include/QtCore
    ${CMAKE_BINARY_DIR}/privateinclude/QtCore
    ${CMAKE_BINARY_DIR}/privateinclude/QtCore/private
    ${CMAKE_BINARY_DIR}/include/QtSql
    ${CMAKE_BINARY_DIR}/privateinclude/QtSql
    ${CMAKE_BINARY_DIR}/privateinclude/QtSql/private
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}/kernel
    ${CMAKE_CURRENT_SOURCE_DIR}/models
    ${CMAKE_CURRENT_SOURCE_DIR}/drivers/db2
    ${CMAKE_CURRENT_SOURCE_DIR}/drivers/ibase
    ${CMAKE_CURRENT_SOURCE_DIR}/drivers/mysql
    ${CMAKE_CURRENT_SOURCE_DIR}/drivers/oci
    ${CMAKE_CURRENT_SOURCE_DIR}/drivers/odbc
    ${CMAKE_CURRENT_SOURCE_DIR}/drivers/psql
    ${CMAKE_CURRENT_SOURCE_DIR}/drivers/sqlite
)

include(kernel/kernel.cmake)
include(models/models.cmake)
include(drivers/db2/db2.cmake)
include(drivers/ibase/ibase.cmake)
include(drivers/mysql/mysql.cmake)
include(drivers/oci/oci.cmake)
include(drivers/odbc/odbc.cmake)
include(drivers/psql/psql.cmake)
include(drivers/sqlite/sqlite.cmake)

macro_generate_public("${SQL_PUBLIC_INCLUDES}" QtSql)
macro_generate_private("${SQL_PRIVATE_INCLUDES}" QtSql)
macro_generate_misc("${SQL_INCLUDES}" QtSql)

if(${CMAKE_SYSTEM_NAME} MATCHES "Windows")
    set(EXTRA_SQL_LIBS
        ${EXTRA_SQL_LIBS}
        ws2_32
    )
elseif(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
    add_definitions(-DENABLE_JSC_MULTIPLE_THREADS=0)
endif()

function_variable_fixup("${EXTRA_SQL_CXXFLAGS}" EXTRA_SQL_CXXFLAGS)
function_variable_fixup("${EXTRA_SQL_LDFLAGS}" EXTRA_SQL_LDFLAGS)

add_library(CsSql${BUILD_MAJOR} SHARED ${SQL_SOURCES} ${SQL_INCLUDES})
target_link_libraries(CsSql${BUILD_MAJOR} ${EXTRA_SQL_LIBS})
set_target_properties(CsSql${BUILD_MAJOR} PROPERTIES
    VERSION "0" # for compat
    SOVERSION "0.0.0" # for compat
    COMPILE_FLAGS ${EXTRA_SQL_CXXFLAGS}
    LINK_FLAGS ${EXTRA_SQL_LDFLAGS}
)

macro_generate_resources("${SQL_SOURCES}")
macro_generate_package(
    CsSql${BUILD_MAJOR}
    QtSql
    "${EXTRA_SQL_CXXFLAGS}"
    "${EXTRA_SQL_LIBS}"
    "CsCore${BUILD_MAJOR}"
)

install(
    TARGETS CsSql${BUILD_MAJOR}
    EXPORT CopperSpiceLibraryTargets ${INSTALL_TARGETS_DEFAULT_ARGS}
    DESTINATION ${CMAKE_INSTALL_LIBDIR}
)

install(
    DIRECTORY ${CMAKE_BINARY_DIR}/include/QtSql
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
    COMPONENT Devel
)