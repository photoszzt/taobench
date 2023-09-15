cmake_minimum_required(VERSION 3.7)

project(taobench)

set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_COMPILER "g++-12")
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -pthread")

option(WITH_CRDB OFF)
option(WITH_MYSQL OFF)
option(WITH_SPANNER OFF)
option(WITH_YUGABYTE OFF)

include_directories(src)
file(GLOB SOURCES src/*.h src/*.cc)
add_executable(taobench ${SOURCES})

if(WITH_CRDB)
  include_directories(crdb)
  target_sources(taobench PRIVATE
    crdb/crdb_db.h
    crdb/crdb_db.cc)
if (CMAKE_C_COMPILER_ID STREQUAL "Clang")
  target_include_directories(taobench PUBLIC
  "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/third-party/deps_clang/include;>")
  find_library(PQXX_LIB NAMES pqxx HINTS ${CMAKE_CURRENT_SOURCE_DIR}/third-party/deps_clang/lib)
elseif (CMAKE_C_COMPILER_ID STREQUAL "GNU")
  target_include_directories(taobench PUBLIC 
  "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/third-party/deps_gcc/include;>")
  find_library(PQXX_LIB NAMES pqxx HINTS ${CMAKE_CURRENT_SOURCE_DIR}/third-party/deps_gcc/lib)
endif()
  target_link_libraries(taobench ${PQXX_LIB})
  target_link_libraries(taobench pq)
endif()

if(WITH_MYSQL)
  include_directories(mysqldb)
  target_sources(taobench PRIVATE
    mysqldb/mysql_db.h
    mysqldb/mysql_db.cc)
  target_link_libraries(taobench -lmysqlclient)
endif()

if(WITH_SPANNER)
  include_directories(spanner_db)
  target_sources(taobench PRIVATE
    spanner_db/spanner_db.h
    spanner_db/spanner_db.cc)
  target_link_libraries(taobench google-cloud-cpp::spanner)
endif()

if(WITH_YUGABYTE)
  include_directories(yugabytedb)
  target_sources(taobench PRIVATE
    yugabytedb/yugabytedb.h
    yugabytedb/yugabytedb.cc)
  target_link_libraries(taobench -lpqxx)
  target_link_libraries(taobench -lpq)
endif()
