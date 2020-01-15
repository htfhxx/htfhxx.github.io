cmake_minimum_required(VERSION 2.8)



project(HelloWorld)



#头文件目录 
INCLUDE_DIRECTORIES(
 ${CMAKE_SOURCE_DIR}/  #包含该目录下的.h文件
)

# should we use our own math functions?
option (OK
        "Use our cpp" ON) 

if(OK)
	INCLUDE_DIRECTORIES(
	#去找MathFunctions头文件
	 ${PROJECT_SOURCE_DIR}/MathFunctions   
	)
	
	#子文件添加一个库
	add_subdirectory(MathFunctions) 
	
	set (EXTRA_LIBS ${EXTRA_LIBS} MathFunctions)

endif (OK)

#可执行文件，并链接添加的库
add_executable(hellospeak hellospeak.cpp)
target_link_libraries(hellospeak  ${EXTRA_LIBS})

