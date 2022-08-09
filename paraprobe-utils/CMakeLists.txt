cmake_minimum_required(VERSION 3.18.4)

#please name your project accordingly
set(MYPROJECTNAME "paraprobe_utils")
message([STATUS] "Projectname is ${MYPROJECTNAME}")
project(${MYPROJECTNAME} LANGUAGES CXX)
set(CMAKE_BUILD_DIR "build")

#pull general information about external libraries and paths related to paraprobe
message([STATUS] "CONDA_PATH: ${CONDA_PREFIX}")


set(EMPLOY_GNUCOMPILER ON)
message([STATUS] "MYCCC_COMPILER: __${CMAKE_C_COMPILER}__")
message([STATUS] "MYCXX_COMPILER: __${CMAKE_CXX_COMPILER}__")


#choose optimization level
##-O0 nothing, debug purposes, -O1 moderate optimization, -O2 -O3 for production level up to aggressive architecture specific non-portable optimization
if(EMPLOY_GNUCOMPILER)
	set(MYOPTLEVEL "-O2")
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${MYOPTLEVEL}")
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${MYOPTLEVEL}")
	message([STATUS] "We utilize optimization level ${MYOPTLEVEL}")													
endif()

set(MYHDFLINKFLAGS "-L${CONDA_PREFIX}/lib/ ${CONDA_PREFIX}/lib/libhdf5_hl.so ${CONDA_PREFIX}/lib/libhdf5.so ${CONDA_PREFIX}/lib/libz.a ${CONDA_PREFIX}/lib/libsz.a ${CONDA_PREFIX}/lib/libaec.a -ldl")

include_directories("${CONDA_PREFIX}/include/voro++")

#GMP and MPFR is included and found alongside the CGAL library
#connect with ExternalLibraries.cmake via MYCGALHOME
find_package(CGAL QUIET)
find_package(Eigen)


link_directories(${Boost_LIBRARY_DIR})
find_package(Boost)

#include_directories("${MYVOROXXPATH}/src")

#automatically assign project name and compiler flags

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
set(MYTOOLSRCPATH "paraprobe-toolbox/code/paraprobe-utils/src/cxx")
set(MYVOROSRCPATH "voro++/voro/src")

add_library(paraprobe-utils OBJECT
	../thirdparty/mandatory/hdf5cxxwrapper/src/cxx/hdf5_wrapper_cxx_structs.cpp
	../thirdparty/mandatory/hdf5cxxwrapper/src/cxx/hdf5_wrapper_cxx_sequ_core.cpp
	${MYTOOLSRCPATH}/PARAPROBE_CoordinateSystem.cpp
	${MYTOOLSRCPATH}/CONFIG_Shared.cpp
	${MYTOOLSRCPATH}/PARAPROBE_CiteMe.cpp
	${MYTOOLSRCPATH}/PARAPROBE_Verbose.cpp
	${MYTOOLSRCPATH}/PARAPROBE_Profiling.cpp
	${MYTOOLSRCPATH}/PARAPROBE_Math.cpp
	${MYTOOLSRCPATH}/PARAPROBE_PrimsContinuum.cpp
	${MYTOOLSRCPATH}/PARAPROBE_PrimsDiscrete.cpp
	${MYTOOLSRCPATH}/PARAPROBE_Geometry.cpp
	${MYTOOLSRCPATH}/PARAPROBE_UserStructs.cpp
	${MYTOOLSRCPATH}/PARAPROBE_UserStructsSorting.cpp
	${MYTOOLSRCPATH}/PARAPROBE_MPIStructs.cpp
	${MYTOOLSRCPATH}/PARAPROBE_PeriodicTable.cpp
	${MYTOOLSRCPATH}/PARAPROBE_Crystallography.cpp
	${MYTOOLSRCPATH}/PARAPROBE_OriMath.cpp
	${MYTOOLSRCPATH}/PARAPROBE_CGALInterface.cpp
	${MYTOOLSRCPATH}/PARAPROBE_Solutes.cpp
	${MYTOOLSRCPATH}/PARAPROBE_ThermodynamicPhase.cpp
	${MYTOOLSRCPATH}/PARAPROBE_Grains.cpp
	${MYTOOLSRCPATH}/PARAPROBE_Precipitates.cpp
	${MYTOOLSRCPATH}/PARAPROBE_Histogram2D.cpp
	${MYTOOLSRCPATH}/PARAPROBE_TwoPointStats3D.cpp
	${MYTOOLSRCPATH}/PARAPROBE_ROIs.cpp
	${MYTOOLSRCPATH}/PARAPROBE_AABBTree.cpp
	${MYTOOLSRCPATH}/PARAPROBE_KDTree.cpp
	${MYTOOLSRCPATH}/PARAPROBE_VolumeBinning.cpp
	${MYTOOLSRCPATH}/PARAPROBE_IonCloudMemory.cpp
	${MYTOOLSRCPATH}/PARAPROBE_HDF5BaseHdl.cpp
	${MYTOOLSRCPATH}/PARAPROBE_XDMFBaseHdl.cpp
	${MYTOOLSRCPATH}/PARAPROBE_TriangleSoupHdl.cpp
	${MYTOOLSRCPATH}/PARAPROBE_ToolsBaseHdl.cpp
	${MYTOOLSRCPATH}/PARAPROBE_ConfigBaseHDF5.cpp


	${MYVOROSRCPATH}/c_loops.cc
	${MYVOROSRCPATH}/cell.cc
	#${MYVOROSRCPATH}/cmd_line.cc
	${MYVOROSRCPATH}/common.cc
	${MYVOROSRCPATH}/container.cc
	${MYVOROSRCPATH}/container_prd.cc
	${MYVOROSRCPATH}/pre_container.cc
	${MYVOROSRCPATH}/unitcell.cc
	${MYVOROSRCPATH}/v_base.cc
	#${MYVOROSRCPATH}/v_base_wl.cc
	${MYVOROSRCPATH}/v_compute.cc
	#${MYVOROSRCPATH}/voro++.cc
	${MYVOROSRCPATH}/wall.cc
)

#nothing to link because utilities are compiled into object files only
