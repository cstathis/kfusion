cmake_minimum_required(VERSION 2.8)
project(kfusion)


set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -g -O3                  -Wall -Wno-unknown-pragmas -Wextra -Werror -Wno-error=deprecated-declarations -Wno-error=array-bounds")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -g -O3  --std=c++11 -Wall -Wno-unknown-pragmas -Wextra -Werror -Wno-error=deprecated-declarations -Wno-error=array-bounds ")

find_package(TooN   REQUIRED)
find_package(Eigen3   REQUIRED)

include_directories(${EIGEN3_INCLUDE_DIR})
include_directories(${TOON_INCLUDE_PATHS})


include_directories(${PROJECT_SOURCE_DIR} ${SLAMBENCH_INCLUDE_DIR})
   
 # ----------------- CPP VERSION ----------------- 

add_library(${PROJECT_NAME}-cpp SHARED kernels.cpp kfusion_library.cpp)
target_link_libraries(${PROJECT_NAME}-cpp ${common_libraries} ${SLAMBENCH_LIBRARIES})	
SET_TARGET_PROPERTIES(${PROJECT_NAME}-cpp PROPERTIES COMPILE_FLAGS "-fPIC")
install(TARGETS ${appname}-cpp  DESTINATION  lib/)   

 # ----------------- DYNAMIC  ----------------- 
 
generate_slam_library(${appname}-cpp "")
target_include_directories(${appname}-cpp  PRIVATE  ./cpp_headers )


 #  ----------------- CUDA VERSION ----------------- 

find_package(CUDA)
STRING(REPLACE "_static.a" ".so" CUDA_LIBRARIES "${CUDA_LIBRARIES}")


if (CUDA_FOUND) 

    SET( CUDA_PROPAGATE_HOST_FLAGS OFF ) 

    set(kfusion_cuda_srcs kernels.cu)
	    	
	cuda_add_library(${appname}-cuda    SHARED  ${kfusion_cuda_srcs}  kfusion_library.cpp OPTIONS -O3 -std=c++11 -Xcompiler -fPIC -D_MWAITXINTRIN_H_INCLUDED -D_FORCE_INLINES -I${CMAKE_CURRENT_SOURCE_DIR}/cuda_headers )
	target_link_libraries(${appname}-cuda   ${common_libraries} ${SLAMBENCH_LIBRARIES})	
	target_include_directories(${appname}-cuda  PUBLIC ./cuda_headers )
    install(TARGETS ${appname}-cuda  DESTINATION  lib/)

    
	generate_slam_library(${appname}-cuda "")
	
endif(CUDA_FOUND)




