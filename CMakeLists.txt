cmake_minimum_required(VERSION 3.1)

project(mathtoolbox CXX)

set(CMAKE_CXX_STANDARD 11)

######################################################################
# Options
######################################################################

option(MATHTOOLBOX_NLOPT_BASED_MODULES "Build NLopt-based modules" ON)
option(MATHTOOLBOX_BUILD_EXAMPLES "Build example applications" OFF)

######################################################################
# Check prerequisites
######################################################################

find_package(Eigen3 REQUIRED)
if((NOT TARGET Eigen3::Eigen) AND (DEFINED EIGEN3_INCLUDE_DIR))
	add_library(AliasEigen3 INTERFACE)
	target_include_directories(AliasEigen3 INTERFACE ${EIGEN3_INCLUDE_DIR})
	add_library(Eigen3::Eigen ALIAS AliasEigen3)
endif()

######################################################################
# External libraries
######################################################################

if(MATHTOOLBOX_NLOPT_BASED_MODULES)
	set(BUILD_SHARED_LIBS OFF CACHE INTERNAL "" FORCE)
	set(NLOPT_GUILE       OFF CACHE INTERNAL "" FORCE)
	set(NLOPT_LINK_PYTHON OFF CACHE INTERNAL "" FORCE)
	set(NLOPT_MATLAB      OFF CACHE INTERNAL "" FORCE)
	set(NLOPT_OCTAVE      OFF CACHE INTERNAL "" FORCE)
	set(NLOPT_PYTHON      OFF CACHE INTERNAL "" FORCE)
	set(NLOPT_SWIG        OFF CACHE INTERNAL "" FORCE)
	set(NLOPT_TESTS       OFF CACHE INTERNAL "" FORCE)
	set(NLOPT_CXX         OFF CACHE INTERNAL "" FORCE)
	set(NLOPT_FORTRAN     OFF CACHE INTERNAL "" FORCE)
	set(WITH_THREADLOCAL  OFF CACHE INTERNAL "" FORCE)
	set(INSTALL_BIN_DIR     "bin"             CACHE INTERNAL "" FORCE)
	set(INSTALL_CMAKE_DIR   "lib/cmake/nlopt" CACHE INTERNAL "" FORCE)
	set(INSTALL_DATA_DIR    "share/nlopt"     CACHE INTERNAL "" FORCE)
	set(INSTALL_INCLUDE_DIR "include"         CACHE INTERNAL "" FORCE)
	set(INSTALL_LIB_DIR     "lib"             CACHE INTERNAL "" FORCE)
	set(INSTALL_MAN_DIR     "share/man"       CACHE INTERNAL "" FORCE)
	add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/external/nlopt)
	add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/external/nlopt-util)
endif()

######################################################################
# Build
######################################################################

file(GLOB
	headers
	${CMAKE_CURRENT_SOURCE_DIR}/include/mathtoolbox/bfgs.hpp
	${CMAKE_CURRENT_SOURCE_DIR}/include/mathtoolbox/classical-mds.hpp
	${CMAKE_CURRENT_SOURCE_DIR}/include/mathtoolbox/l-bfgs.hpp
	${CMAKE_CURRENT_SOURCE_DIR}/include/mathtoolbox/numerical-optimization.hpp
	${CMAKE_CURRENT_SOURCE_DIR}/include/mathtoolbox/rbf-interpolation.hpp
)
file(GLOB
	sources
	${CMAKE_CURRENT_SOURCE_DIR}/src/bfgs.cpp
	${CMAKE_CURRENT_SOURCE_DIR}/src/l-bfgs.cpp
	${CMAKE_CURRENT_SOURCE_DIR}/src/rbf-interpolation.cpp
)
file(GLOB
	headers_nlopt
	${CMAKE_CURRENT_SOURCE_DIR}/include/mathtoolbox/gaussian-process-regression.hpp
)
file(GLOB
	sources_nlopt
	${CMAKE_CURRENT_SOURCE_DIR}/src/gaussian-process-regression.cpp
)

if(MATHTOOLBOX_NLOPT_BASED_MODULES)
	add_library(mathtoolbox STATIC ${headers} ${sources} ${headers_nlopt} ${sources_nlopt})
	target_link_libraries(mathtoolbox PUBLIC Eigen3::Eigen nlopt nlopt-util)
else()
	add_library(mathtoolbox STATIC ${headers} ${sources})
	target_link_libraries(mathtoolbox PUBLIC Eigen3::Eigen)
endif()
target_include_directories(mathtoolbox PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include)

######################################################################
# Installation
######################################################################

if(MATHTOOLBOX_NLOPT_BASED_MODULES)
	install(FILES ${headers} ${headers_nlopt} DESTINATION include/mathtoolbox)
else()
	install(FILES ${headers} DESTINATION include/mathtoolbox)
endif()
install(TARGETS mathtoolbox ARCHIVE DESTINATION lib)

######################################################################
# Example demos
######################################################################

if(MATHTOOLBOX_BUILD_EXAMPLES)
	set(OTF_WITH_EIGEN ON CACHE INTERNAL "" FORCE)
	add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/external/optimization-test-functions)

	add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/examples/bfgs)
	add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/examples/classical-mds)
	add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/examples/l-bfgs)
	add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/examples/rbf-interpolation)

	if(MATHTOOLBOX_NLOPT_BASED_MODULES)
		add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/examples/gaussian-process-regression)
	endif()
endif()

######################################################################
# Tests
######################################################################

enable_testing()
if(MATHTOOLBOX_BUILD_EXAMPLES)
	add_test(NAME bfgs-test COMMAND $<TARGET_FILE:bfgs-test>)
	add_test(NAME classical-mds-test COMMAND $<TARGET_FILE:classical-mds-test>)
	add_test(NAME l-bfgs-test COMMAND $<TARGET_FILE:l-bfgs-test>)
	add_test(NAME rbf-interpolation-test COMMAND $<TARGET_FILE:rbf-interpolation-test>)

	if(MATHTOOLBOX_NLOPT_BASED_MODULES)
		add_test(NAME gaussian-process-regression-test COMMAND $<TARGET_FILE:gaussian-process-regression-test> .)
	endif()
endif()
