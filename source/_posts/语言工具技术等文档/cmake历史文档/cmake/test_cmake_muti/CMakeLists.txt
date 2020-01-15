cmake_minimum_required(VERSION 2.8)



project(HelloWorld)

#directories of .h 
INCLUDE_DIRECTORIES(
 ${CMAKE_SOURCE_DIR}/  
)

SET(Sources_code  hellospeak.cpp speak.cpp)

add_executable(hellospeak  ${Sources_code})
