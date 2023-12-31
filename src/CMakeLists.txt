cmake_minimum_required(VERSION 3.20)

file(GLOB_RECURSE OT_SOURCES ./**.cpp)

if(USE_STATIC_LIBS)
    SET(CMAKE_FIND_LIBRARY_SUFFIXES ".a")
    SET(CMAKE_EXE_LINKER_FLAGS "-static-libgcc -static-libstdc++ -static")
endif()

add_executable(${PROJECT_NAME} ${OT_SOURCES})

find_package(Boost REQUIRED COMPONENTS system iostreams filesystem thread)
find_package(LibXml2 REQUIRED)
find_package(Lua REQUIRED)
find_package(cryptopp CONFIG REQUIRED)
find_package(OpenSSL REQUIRED)

target_link_libraries(${PROJECT_NAME}
        ${Boost_LIBRARIES}
        ${LIBXML2_LIBRARIES}
        ${LUA_LIBRARIES}
        ${OPENSSL_LIBRARIES}
        cryptopp::cryptopp)

if(USE_STATIC_LIBS)
    find_package(ZLIB REQUIRED)
    find_package(Iconv REQUIRED)
    target_link_libraries(${PROJECT_NAME} ${ZLIB_LIBRARIES} ${ICONV_LIBRARIES})

    if(UNIX)
        find_package(LZMA REQUIRED)
        target_link_libraries(${PROJECT_NAME} ${LZMA_LIBRARIES})
    endif()

    # notify cmake about static build
    set_target_properties(${PROJECT_NAME} PROPERTIES LINK_SEARCH_START_STATIC ON)
    set_target_properties(${PROJECT_NAME} PROPERTIES LINK_SEARCH_END_STATIC ON)

    # notify source code about static build
    remove_definitions(-D__STATIC__)
    add_definitions(-D__STATIC__)
endif()

if(USE_SQLITE)
    add_definitions(-D__USE_SQLITE__)
    find_package(unofficial-sqlite3 CONFIG REQUIRED)
    target_link_libraries(${PROJECT_NAME} unofficial::sqlite3::sqlite3)
endif ()

if(USE_MYSQL)
    add_definitions(-D__USE_MYSQL__)
    find_package(unofficial-libmysql CONFIG REQUIRED)
    target_link_libraries(${PROJECT_NAME} unofficial::libmysql::libmysql)
endif ()

if (UNIX)
    find_package(PkgConfig REQUIRED)
    pkg_check_modules(gmpxx REQUIRED IMPORTED_TARGET gmpxx)
    target_link_libraries(${PROJECT_NAME} PkgConfig::gmpxx)
else()
    find_package(GMP REQUIRED)
    target_link_libraries(${PROJECT_NAME} ${GMP_LIBRARY})
endif ()

if(UNIX)
    find_package(Threads)
    target_link_libraries(${PROJECT_NAME} ${CMAKE_THREAD_LIBS_INIT} dl)
endif()

if(WIN32)
    target_link_libraries(${PROJECT_NAME} ws2_32 mswsock)
endif()

target_include_directories(${PROJECT_NAME} PUBLIC ${LIBXML2_INCLUDE_DIRS})