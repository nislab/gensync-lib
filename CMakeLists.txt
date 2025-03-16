cmake_minimum_required(VERSION 3.10)
project(gensync)
set(CMAKE_CXX_STANDARD 17)

# if compiled with RECORD, GenSync server and client produce extensive log files
# that can be used to reproduce exact sync behavior afterwards
add_definitions(-DRECORD=".gensync")

include(CTest)
include(GNUInstallDirs)

# include Apache Data Sketches as a header-only library
include_directories(gensync-core/incubator-datasketches-cpp/common/include
                    gensync-core/incubator-datasketches-cpp/hll/include
                    gensync-core/incubator-datasketches-cpp/fi/include)

# some flags
set(CMAKE_CXX_FLAGS "-std=c++${CMAKE_CXX_STANDARD} -DDEFAULT_LOGLEVEL=TEST")

# Set project directory structure
set(ROOT_DIR gensync-core)
set(SRC_DIR ${ROOT_DIR}/src)
set(AUX_DIR ${SRC_DIR}/Aux)
set(DATA_DIR ${SRC_DIR}/Data)
set(COMM_DIR ${SRC_DIR}/Communicants)
set(SYNC_DIR ${SRC_DIR}/Syncs)
set(BENCH_DIR ${SRC_DIR}/Benchmarks)
set(TOOLS_DIR ${SRC_DIR}/Tools)
set(TESTS_DIR ${ROOT_DIR}/tests)
set(UNIT_TESTS_DIR ${TESTS_DIR}/unit)

set(INCLUDE gensync-core/include/GenSync)
set(AUX_DIR_INC ${INCLUDE}/Aux)
set(DATA_DIR_INC ${INCLUDE}/Data)
set(COMM_DIR_INC ${INCLUDE}/Communicants)
set(SYNC_DIR_INC ${INCLUDE}/Syncs)
set(SYNC_BENCH_INC ${INCLUDE}/Benchmarks)

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
        ${SYNC_DIR}/IBLT.cpp
        ${SYNC_DIR}/IBLTMultiset.cpp
        ${SYNC_DIR}/IBLTSync.cpp
        ${SYNC_DIR}/IBLTSync_Multiset.cpp
        ${SYNC_DIR}/IBLTSetOfSets.cpp
        ${SYNC_DIR}/Compact2DBitArray.cpp
        ${SYNC_DIR}/Cuckoo.cpp
        ${SYNC_DIR}/CuckooSync.cpp
        ${SYNC_DIR}/FullSync.cpp
	    ${SYNC_DIR}/BloomFilter.cpp
	    ${SYNC_DIR}/BloomFilterSync.cpp

        ${BENCH_DIR}/Runner.cpp
        ${BENCH_DIR}/BenchParams.cpp
        ${BENCH_DIR}/FromFileGen.cpp

        ${TOOLS_DIR}/EncodeJoin.cpp

        ${SRC_DIR}/main.cpp)

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

        ${SYNC_BENCH_INC}/BenchObserv.h
        ${SYNC_BENCH_INC}/BenchParams.h
        ${SYNC_BENCH_INC}/DataObjectGenerator.h
        ${SYNC_BENCH_INC}/RandGen.h
        ${SYNC_BENCH_INC}/FromFileGen.h)

include_directories(gensync-core/include)
# Check if the "includedir" variable is already set
if(${CMAKE_SYSTEM_NAME} STREQUAL "Linux")
        if(NOT DEFINED includedir)
                set(includedir "/usr/local")
        endif()
elseif(${CMAKE_SYSTEM_NAME} STREQUAL "Darwin")
        if(NOT DEFINED includedir)
                set(includedir "/opt/local")
        endif()
endif()

# Users may need to update these paths
include_directories(${includedir}/include)
link_directories(${includedir}/lib)
link_directories(${includedir}/lib64)

include_directories(${CMAKE_INSTALL_INCLUDEDIR})
link_directories(${CMAKE_INSTALL_LIBDIR})

# Add libs and executables
add_library(gensync STATIC ${SOURCE_FILES} ${HEADERS})
target_include_directories(gensync PRIVATE ${CMAKE_SOURCE_DIR}/gensync-core/include)
TARGET_LINK_LIBRARIES(gensync ntl cppunit pthread gmp)

install(CODE "FILE(REMOVE_RECURSE ${CMAKE_INSTALL_INCLUDEDIR}/gensync)") #Remove the existing GenSync library data and reinstall
install(CODE "FILE(REMOVE_RECURSE ${CMAKE_INSTALL_LIBDIR}/libgensync.a)") #Remove the existing GenSync library data and reinstall
install(TARGETS gensync LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR} ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR})
install(DIRECTORY DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/GenSync) #create a new folder for GenSync inside of the system include directory
install(DIRECTORY ${AUX_DIR_INC} ${DATA_DIR_INC} ${COMM_DIR_INC} ${SYNC_DIR_INC} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/GenSync COMPONENT devel) #copy the contents of the include folder into the system include directory

# CppUnit tests
# ... find cppunit include and library, if it exists
find_path(CPPUNIT_INCLUDE_DIR cppunit/TestRunner.h
        PATHS /usr/include /usr/local/include /opt/local/include
)
find_library(CPPUNIT_LIBRARY cppunit
        PATHS /usr/lib /usr/local/lib /opt/local/lib
)
if (CPPUNIT_INCLUDE_DIR AND CPPUNIT_LIBRARY)
    message(STATUS "✅ CppUnit found: Enabling unit tests.")
    file(GLOB TEST_SOURCES ${UNIT_TESTS_DIR}/*.cpp) # detect test files
    include_directories(${TESTS_DIR})
    add_executable(test_runner ${TEST_SOURCES} ${TESTS_DIR}/testRunner.cpp
            gensync-core/tests/unit/AuxiliaryTest.cpp
            gensync-core/tests/unit/AuxiliaryTest.h
            gensync-core/tests/unit/BloomFilterSyncTest.cpp
            gensync-core/tests/unit/BloomFilterSyncTest.h
            gensync-core/tests/unit/BloomFilterTest.cpp
            gensync-core/tests/unit/BloomFilterTest.h
            gensync-core/tests/unit/CommSocketTest.cpp
            gensync-core/tests/unit/CommSocketTest.h
            gensync-core/tests/unit/CommStringTest.cpp
            gensync-core/tests/unit/CommStringTest.h
            gensync-core/tests/unit/CommunicantTest.cpp
            gensync-core/tests/unit/CommunicantTest.h
            gensync-core/tests/unit/Compact2DBitArrayTest.cpp
            gensync-core/tests/unit/Compact2DBitArrayTest.h
            gensync-core/tests/unit/CPISyncTest.cpp
            gensync-core/tests/unit/CPISyncTest.h
            gensync-core/tests/unit/CuckooSyncTest.cpp
            gensync-core/tests/unit/CuckooSyncTest.h
            gensync-core/tests/unit/CuckooTest.cpp
            gensync-core/tests/unit/CuckooTest.h
            gensync-core/tests/unit/DataObjectTest.cpp
            gensync-core/tests/unit/DataObjectTest.h
            gensync-core/tests/unit/DataPriorityObjectTest.cpp
            gensync-core/tests/unit/DataPriorityObjectTest.h
            gensync-core/tests/unit/FullSyncTest.cpp
            gensync-core/tests/unit/FullSyncTest.h
            gensync-core/tests/unit/IBLTSetOfSetsTest.cpp
            gensync-core/tests/unit/CPISyncTest.cpp)
    target_link_libraries(test_runner )
    enable_testing()
    add_test(NAME UnitTests COMMAND test_runner)
else()
    message(WARNING "CppUnit NOT found.  Skipping unit tests.")
endif()


    #Packaging
set(CPACK_GENERATOR "RPM;DEB")
set(CPACK_PACKAGE_VERSION ${VERSION})
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "A library for remote file synchronization research")
set(CPACK_PACKAGE_DESCRIPTION_FILE ${CMAKE_SOURCE_DIR}/README.md)
set(CPACK_RESOURCE_FILE_LICENSE ${CMAKE_SOURCE_DIR}/LICENSE)
set(CPACK_PACKAGE_NAME "GenSync")
set(CPACK_PACKAGE_VERSION_MAJOR "2")
set(CPACK_PACKAGE_VERSION_MINOR "0")
set(CPACK_PACKAGE_VERSION_PATCH "5")
set(CPACK_PACKAGE_CONTACT "Ari Trachtenberg")
set(CPACK_PACKAGE_VENDOR "NISLAB")
set(CPACK_PACKAGING_INSTALL_PREFIX ${CMAKE_INSTALL_PREFIX})
set(CPACK_PACKAGE_FILE_NAME "${CPACK_PACKAGE_NAME}-${CPACK_PACKAGE_VERSION_MAJOR}.${CPACK_PACKAGE_VERSION_MINOR}.${CPACK_PACKAGE_VERSION_PATCH}-${CPACK_PACKAGE_RELEASE}.${CMAKE_SYSTEM_PROCESSOR}")
set(CPACK_RPM_EXCLUDE_FROM_AUTO_FILELIST /opt/local /opt/local/lib /opt/local/include)
include(CPack)
