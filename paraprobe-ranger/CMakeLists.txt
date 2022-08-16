cmake_minimum_required(VERSION 3.18.4)

################################################################################################################################
##DEVELOPER SECTION#############################################################################################################
##in this section software developers need to make changes when debugging#######################################################
################################################################################################################################
#please name your project accordingly
set(MYPROJECTNAME "paraprobe_ranger")
message([STATUS] "Projectname is ${MYPROJECTNAME}")
project(${MYPROJECTNAME} LANGUAGES CXX)
set(CMAKE_BUILD_DIR "build")


message([STATUS] "CONDA_PATH: ${CONDA_PREFIX}")


set(EMPLOY_GNUCOMPILER ON)
message([STATUS] "MYCCC_COMPILER: __${CMAKE_C_COMPILER}__")
message([STATUS] "MYCXX_COMPILER: __${CMAKE_CXX_COMPILER}__")

#define which parallelization layers are used
set(EMPLOY_PARALLELISM_CUDA OFF)

#choose optimization level
##-O0 nothing, debug purposes, -O1 moderate optimization, -O2 -O3 for production level up to aggressive architecture specific non-portable optimization
if(EMPLOY_GNUCOMPILER)
	set(MYOPTLEVEL "-O2")
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${MYOPTLEVEL}")
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${MYOPTLEVEL}")
	message([STATUS] "We utilize optimization level ${MYOPTLEVEL}")
endif()


################################################################################################################################
##END OF INTERACTION FOR NON PRO USERS##########################################################################################
##here advanced users might want/need to make modifications if they use non default places for thirdparty libraries#############
################################################################################################################################
#HDF5 local installation for advanced I/O, collecting metadata and bundle analysis results together
#include_directories("${MYHDFPATH}/include")
#link_directories("${MYHDFPATH}/lib")

include_directories("${CONDA_PREFIX}/include/voro++")

set(MYHDFLINKFLAGS "-L${CONDA_PREFIX}/lib/ ${CONDA_PREFIX}/lib/libhdf5_hl.so ${CONDA_PREFIX}/lib/libhdf5.so ${CONDA_PREFIX}/lib/libz.a ${CONDA_PREFIX}/lib/libsz.a ${CONDA_PREFIX}/lib/libaec.a -ldl")


#set(Boost_INCLUDE_DIR "${MYBOOSTPATH}")
#include_directories("${MYCGALPATH}")
#set(CGAL_DIR "${MYCGALPATH}")
#include_directories("${MYEIGENPATH}")
#include_directories("${MYVOROXXPATH}/src")
find_package(CGAL QUIET)
find_package(Eigen)
find_package(Boost)
link_directories(${Boost_LIBRARY_DIR})


#GMP and MPFR is included and found alongside the CGAL library
#connect with ExternalLibraries.cmake via MYCGALHOME and MYEIGENHOME
#find_package(CGAL QUIET)
#if( CGAL_FOUND )
#	include( ${CGAL_USE_FILE} )
#	if (EMPLOY_MYEIGEN)
#		find_package(Eigen3 3.1.0 REQUIRED) #(3.1.0 or greater)
#		include(CGAL_Eigen3_support)
#		if(NOT TARGET CGAL::Eigen3_support)
#			message(STATUS "This project requires the Eigen library, and will not be compiled.")
#			return()
#		endif()
#	endif()
#endif()

#connect with ExternalLibraries.cmake via MYBOOSTHOME
#include_directories(${Boost_INCLUDE_DIR})
#link_directories(${Boost_LIBRARY_DIR})
#find_package(Boost)

#setting up compiler-specifics
if(EMPLOY_GNUCOMPILER)
	set(MYVERBOSE "-fopt-info")
	add_definitions("${MYOPTLEVEL} ${MYVERBOSE}")
	set(MYOMP "-fopenmp -lpthread")
	add_definitions(${MYOMP})
	add_definitions("-std=c++14")
	add_definitions("-Wall -Warray-bounds -Wchar-subscripts -Wcomment -Wenum-compare -Wformat 
		-Wuninitialized -Wmaybe-uninitialized -Wmain -Wnonnull -Wparentheses -Wreorder -Wreturn-type -Wsign-compare -Wsequence-point 
		-Wtrigraphs -Wunused-function -Wunused-but-set-variable -Wunused-variable") #-Wnarrowing
endif()

#parallelization - MPI process-level
#query location of MPI library
message([STATUS] "MPI_INCLUDE_PATH ${MPI_INCLUDE_PATH}")
if(EMPLOY_PARALLELISM_MPI)
	find_package(MPI REQUIRED)
	include_directories(${MPI_INCLUDE_PATH})
endif()

#specific paths of dependencies for this tool
set(MYTOOLSRCPATH "../../paraprobe-toolbox/code/paraprobe-ranger/src/cxx")
set(MYEXTPATH "../../paraprobe-toolbox/code/thirdparty/mandatory/hdf5cxxwrapper/src/cxx")
set(MYVOROSRCPATH "../../voro/src")
set(COMPILEPATH "../compiled_code")


#list firstly the precompiled shared aka utils, secondly the tool-specific components, lastly the tool-specific main
add_executable(${MYPROJECTNAME}
	#${MYUTILSPATH}/../../${MYPROJECTPATH}code/thirdparty/mandatory/hdf5cxxwrapper/src/cxx/hdf5_wrapper_cxx_sequ_core.cpp.o
	#${MYUTILSPATH}/../../${MYPROJECTPATH}code/thirdparty/mandatory/hdf5cxxwrapper/src/cxx/hdf5_wrapper_cxx_structs.cpp.o
	${COMPILEPATH}/hdf5_wrapper_cxx_sequ_core.cpp.o
	${COMPILEPATH}/hdf5_wrapper_cxx_structs.cpp.o
	${COMPILEPATH}/CONFIG_Shared.cpp.o
	${COMPILEPATH}/PARAPROBE_CiteMe.cpp.o
	${COMPILEPATH}/PARAPROBE_Verbose.cpp.o
	${COMPILEPATH}/PARAPROBE_Profiling.cpp.o
	${COMPILEPATH}/PARAPROBE_Math.cpp.o
	${COMPILEPATH}/PARAPROBE_PrimsContinuum.cpp.o
	${COMPILEPATH}/PARAPROBE_PrimsDiscrete.cpp.o
	${COMPILEPATH}/PARAPROBE_Geometry.cpp.o
	${COMPILEPATH}/PARAPROBE_UserStructs.cpp.o
	${COMPILEPATH}/PARAPROBE_UserStructsSorting.cpp.o
	${COMPILEPATH}/PARAPROBE_MPIStructs.cpp.o
	${COMPILEPATH}/PARAPROBE_PeriodicTable.cpp.o
	${COMPILEPATH}/PARAPROBE_Crystallography.cpp.o
	${COMPILEPATH}/PARAPROBE_OriMath.cpp.o
	${COMPILEPATH}/PARAPROBE_Histogram2D.cpp.o
	${COMPILEPATH}/PARAPROBE_TwoPointStats3D.cpp.o
	${COMPILEPATH}/PARAPROBE_ROIs.cpp.o
	${COMPILEPATH}/PARAPROBE_AABBTree.cpp.o
	${COMPILEPATH}/PARAPROBE_KDTree.cpp.o
	${COMPILEPATH}/PARAPROBE_VolumeBinning.cpp.o
	${COMPILEPATH}/PARAPROBE_IonCloudMemory.cpp.o
	${COMPILEPATH}/PARAPROBE_HDF5BaseHdl.cpp.o
	${COMPILEPATH}/PARAPROBE_XDMFBaseHdl.cpp.o
	${COMPILEPATH}/PARAPROBE_TriangleSoupHdl.cpp.o
	${COMPILEPATH}/PARAPROBE_ToolsBaseHdl.cpp.o
	${COMPILEPATH}/PARAPROBE_ConfigBaseHDF5.cpp.o

	${MYTOOLSRCPATH}/CONFIG_Ranger.cpp
	${MYTOOLSRCPATH}/PARAPROBE_RangerStructs.cpp
	${MYTOOLSRCPATH}/PARAPROBE_RangerHDF5.cpp
	${MYTOOLSRCPATH}/PARAPROBE_RangerXDMF.cpp
	##MK::${MYTOOLSRCPATH}/PARAPROBE_RangerIonBuilder.cpp
	${MYTOOLSRCPATH}/PARAPROBE_RangerHdl.cpp
	${MYTOOLSRCPATH}/PARAPROBE_Ranger.cpp
)

#linking process, the target link libraries command is specific for each tool of the toolbox
target_link_libraries(${MYPROJECTNAME} ${MYOMP} ${MPI_LIBRARIES} ${MYHDFLINKFLAGS})

#MPI compilation settings
if(MPI_COMPILE_FLAGS)
	set_target_properties(${MYPROJECTNAME} PROPERTIES COMPILE_FLAGS "${MPI_COMPILE_FLAGS}")
endif()

if(MPI_LINK_FLAGS)
	set_target_properties(${MYPROJECTNAME} PROPERTIES LINK_FLAGS "${MPI_LINK_FLAGS}")
endif()
