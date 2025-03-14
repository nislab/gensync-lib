cmake_minimum_required(VERSION 3.6)
project(gensync)

set(CMAKE_CXX_STANDARD 11)

# if compiled with RECORD, GenSync server and client produce extensive log files
# that can be used to reproduce exact sync behavior afterwards
set(RECORD_DIR ".gensync" CACHE STRING "location of record directory")

option(BUILD_TESTS "build tests" OFF)
set(DEFAULT_LOG_LEVEL "TEST" CACHE STRING "default log level")

if(BUILD_TESTS)
    include(CTest)
endif()

include(GNUInstallDirs)

# include Apache Data Sketches as a header-only library
include_directories(gensync-core/incubator-datasketches-cpp/common/include
                    gensync-core/incubator-datasketches-cpp/hll/include
                    gensync-core/incubator-datasketches-cpp/fi/include)

# some flags
set(CMAKE_CXX_FLAGS "-std=c++11 -DDEFAULT_LOGLEVEL=TEST")

# Set project directory strucuture
set(SRC_DIR gensync-core/src)
set(AUX_DIR ${SRC_DIR}/Aux)
set(DATA_DIR ${SRC_DIR}/Data)
set(COMM_DIR ${SRC_DIR}/Communicants)
set(SYNC_DIR ${SRC_DIR}/Syncs)
set(BENCH_DIR ${SRC_DIR}/Benchmarks)
set(TOOLS_DIR ${SRC_DIR}/Tools)

set(INCLUDE gensync-core/include/GenSync)
set(AUX_DIR_INC ${INCLUDE}/Aux)
set(DATA_DIR_INC ${INCLUDE}/Data)
set(COMM_DIR_INC ${INCLUDE}/Communicants)
set(SYNC_DIR_INC ${INCLUDE}/Syncs)
set(SYNC_BENCH_INC ${INCLUDE}/Benchmarks)

# Test directory structure
set(TEST_DIR gensync-core/tests)
set(UNIT_TEST_DIR ${TEST_DIR}/unit)
set(SYSLONG_TEST_DIR ${TEST_DIR}/sys/long)
set(BENCHMARK_TEST_DIR ${TEST_DIR}/sys/benchmark)

# Set location of the test runner
set(TEST_RUNNER ${TEST_DIR}/testRunner.cpp)

# Set file config
set(SOURCE_FILES

        ${AUX_DIR}/Logger.cpp
        ${AUX_DIR}/UID.cpp
        ${AUX_DIR}/SyncMethod.cpp
        ${AUX_DIR}/Sketches.cpp

        ${DATA_DIR}/DataObject.cpp

        ${COMM_DIR}/CommSocket.cpp
        ${COMM_DIR}/CommString.cpp
        ${COMM_DIR}/Communicant.cpp
        ${COMM_DIR}/CommDummy.cpp

        ${SYNC_DIR}/CPISync.cpp
        ${SYNC_DIR}/GenSync.cpp
        ${SYNC_DIR}/InterCPISync.cpp
        ${SYNC_DIR}/probCPISync.cpp
        ${SYNC_DIR}/HashSync.cpp
        ${SYNC_DIR}/GenIBLT.cpp
        ${SYNC_DIR}/IBLT.cpp
        ${SYNC_DIR}/IBLTMultiset.cpp
        ${SYNC_DIR}/IBLTSync.cpp
        ${SYNC_DIR}/IBLTSync_Multiset.cpp
        ${SYNC_DIR}/IBLTSetOfSets.cpp
        ${SYNC_DIR}/Compact2DBitArray.cpp
        ${SYNC_DIR}/Cuckoo.cpp
        ${SYNC_DIR}/CuckooSync.cpp
        ${SYNC_DIR}/FullSync.cpp
	    ${SYNC_DIR}/BloomFilterSync.cpp
	    ${SYNC_DIR}/BloomFilter.cpp
        ${SYNC_DIR}/MET_IBLTSync.cpp
        ${SYNC_DIR}/MET_IBLT.cpp

        ${BENCH_DIR}/BenchParams.cpp
        ${BENCH_DIR}/FromFileGen.cpp
    )

set(HEADERS

        ${AUX_DIR_INC}/Auxiliary.h
        ${AUX_DIR_INC}/ConstantsAndTypes.h
        ${AUX_DIR_INC}/Exceptions.h
        ${AUX_DIR_INC}/ForkHandle.h
        ${AUX_DIR_INC}/Logger.h
        ${AUX_DIR_INC}/SyncMethod.h
        ${AUX_DIR_INC}/UID.h
        ${AUX_DIR_INC}/Sketches.h

        ${DATA_DIR_INC}/DataFileC.h
        ${DATA_DIR_INC}/DataMemC.h
        ${DATA_DIR_INC}/DataObjC.h
        ${DATA_DIR_INC}/DataObject.h
        ${DATA_DIR_INC}/DataPriorityObject.h

        ${COMM_DIR_INC}/CommSocket.h
        ${COMM_DIR_INC}/CommString.h
        ${COMM_DIR_INC}/Communicant.h
        ${COMM_DIR_INC}/CommDummy.h

        ${SYNC_DIR_INC}/CPISync.h
        ${SYNC_DIR_INC}/CPISync_ExistingConnection.h
        ${SYNC_DIR_INC}/CPISync_HalfRound.h
        ${SYNC_DIR_INC}/CPISync_HalfRound_Hashed.h
        ${SYNC_DIR_INC}/CPISync_OneLessRound.h
        ${SYNC_DIR_INC}/FullSync.h
        ${SYNC_DIR_INC}/GenSync.h
        ${SYNC_DIR_INC}/HashSync.h
        ${SYNC_DIR_INC}/GenIBLT.h
        ${SYNC_DIR_INC}/IBLT.h
        ${SYNC_DIR_INC}/IBLTMultiset.h
        ${SYNC_DIR_INC}/IBLTSync.h
        ${SYNC_DIR_INC}/IBLTSetOfSets.h
        ${SYNC_DIR_INC}/IBLTSync_HalfRound.h
        ${SYNC_DIR_INC}/IBLTSync_Multiset.h
        ${SYNC_DIR_INC}/Compact2DBitArray.h
        ${SYNC_DIR_INC}/Cuckoo.h
        ${SYNC_DIR_INC}/CuckooSync.h
        ${SYNC_DIR_INC}/InterCPISync.h
        ${SYNC_DIR_INC}/PrioCPISync.h
        ${SYNC_DIR_INC}/ProbCPISync.h
	    ${SYNC_DIR_INC}/BloomFilter.h
	    ${SYNC_DIR_INC}/BloomFilterSync.h
	    ${SYNC_DIR_INC}/BloomFilter.h
        ${SYNC_DIR_INC}/MET_IBLTSync.h
        ${SYNC_DIR_INC}/MET_IBLT.h

        ${SYNC_BENCH_INC}/BenchObserv.h
        ${SYNC_BENCH_INC}/BenchParams.h
        ${SYNC_BENCH_INC}/DataObjectGenerator.h
        ${SYNC_BENCH_INC}/RandGen.h
        ${SYNC_BENCH_INC}/FromFileGen.h
    )

# Users may need to update these paths
set(DEPENDENCIES /opt/local)

include_directories(${DEPENDENCIES}/include)
link_directories(${DEPENDENCIES}/lib)
link_directories(${DEPENDENCIES}/lib64)

include_directories(${CMAKE_INSTALL_INCLUDEDIR})
link_directories(${CMAKE_INSTALL_LIBDIR})

# Add libs and executables
add_library(gensync STATIC ${SOURCE_FILES} ${HEADERS})
target_include_directories(gensync PRIVATE ${CMAKE_SOURCE_DIR}/gensync-core/include)
TARGET_LINK_LIBRARIES(gensync ntl cppunit pthread gmp)
target_include_directories(gensync PUBLIC
        incubator-datasketches-cpp/common/include
        incubator-datasketches-cpp/hll/include
        incubator-datasketches-cpp/fi/include
)

install(CODE "FILE(REMOVE_RECURSE ${CMAKE_INSTALL_INCLUDEDIR}/gensync)") #Remove the existing GenSync library data and reinstall
install(CODE "FILE(REMOVE_RECURSE ${CMAKE_INSTALL_LIBDIR}/libgensync.a)") #Remove the existing GenSync library data and reinstall
install(TARGETS gensync LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR} ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR})
install(DIRECTORY DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/GenSync) #create a new folder for GenSync inside of the system include directory
install(DIRECTORY ${AUX_DIR_INC} ${DATA_DIR_INC} ${COMM_DIR_INC} ${SYNC_DIR_INC} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/GenSync COMPONENT devel) #copy the contents of the include folder into the system include directory


#Packaging
set(CPACK_GENERATOR "RPM;DEB")
set(CPACK_PACKAGE_VERSION ${VERSION})
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "A library for remote file synchronization research")
set(CPACK_PACKAGE_DESCRIPTION_FILE ${CMAKE_SOURCE_DIR}/README.md)
set(CPACK_RESOURCE_FILE_LICENSE ${CMAKE_SOURCE_DIR}/LICENSE)
set(CPACK_PACKAGE_NAME "GenSync")
set(CPACK_PACKAGE_VERSION_MAJOR "2")
set(CPACK_PACKAGE_VERSION_MINOR "0")
set(CPACK_PACKAGE_VERSION_PATCH "3")
set(CPACK_PACKAGE_CONTACT "Ari Trachtenberg")
set(CPACK_PACKAGE_VENDOR "NISLAB")
set(CPACK_PACKAGING_INSTALL_PREFIX ${CMAKE_INSTALL_PREFIX})
set(CPACK_PACKAGE_FILE_NAME "${CPACK_PACKAGE_NAME}-${CPACK_PACKAGE_VERSION_MAJOR}.${CPACK_PACKAGE_VERSION_MINOR}.${CPACK_PACKAGE_VERSION_PATCH}-${CPACK_PACKAGE_RELEASE}.${CMAKE_SYSTEM_PROCESSOR}")
set(CPACK_RPM_EXCLUDE_FROM_AUTO_FILELIST /opt/local /opt/local/lib /opt/local/include)
include(CPack)
