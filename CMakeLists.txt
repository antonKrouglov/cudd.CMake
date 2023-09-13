cmake_minimum_required(VERSION 3.6...3.27)

####################### Begin Options ###################

option(CUDD_BUILD_TESTS "Build the tests" ON)
option(CUDD_ENABLE_DDDMP "Include libdddmp in libcudd" ON)
option(CUDD_ENABLE_OBJ "Include C++ libobj in libcudd" ON)
option(CUDD_ENABLE_SHARED "Build shared libraries" OFF)
option(CUDD_WITH_SYSTEM_QSORT "Use system qsort instead of portable one" OFF)

####################### End Options ###################


# In-source build prevention.
if(${CMAKE_SOURCE_DIR} STREQUAL ${CMAKE_BINARY_DIR})
  message(STATUS "Project requires an out-of-source build.")
  message(STATUS "Please remove these files from ${CMAKE_BINARY_DIR} first:")
  message(STATUS "CMakeCache.txt")
  message(STATUS "CMakeFiles")
  message(STATUS "Once these files are removed, create a separate directory")
  message(STATUS "and run CMake from there.")
  message(FATAL_ERROR "in-source build detected")
endif()

if(CUDD_ENABLE_OBJ)
  project(CUDD C CXX)
else()
  project(CUDD C)
endif()

set(VERSION "3.0.0")

set(BUILD_SHARED_LIBS "${CUDD_ENABLE_SHARED}")

set(MY_LIBS "")

####################### Begin compiler configurations ###################

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE "RelWithDebInfo")
endif()



# Compiler options fix as per https://stackoverflow.com/a/50882216/2746150
# ---------------------------Old options---------------------------
# set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -Wextra")
# set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra")
# include(CheckCXXCompilerFlag)
# ---------------------------New options---------------------------
if (MSVC)
    add_compile_options(/W4) # warning level 4
    set(CMAKE_C_STANDARD 11) # msvc does not support c99
else()
    add_compile_options(-Wall -Wextra -fPIC) # additional warnings + Generate position-independent code
    set(CMAKE_C_STANDARD 99) 
endif()


if(CUDD_ENABLE_OBJ)

# C++ version fix as per https://stackoverflow.com/q/49093833/2746150  
  # ---------------------------Old checks---------------------------
  # include(CheckCXXCompilerFlag)
  # CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
  # if(NOT COMPILER_SUPPORTS_CXX11)
  #   message(FATAL_ERROR "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support.")
  # endif()
  # set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
  # ---------------------------New checks---------------------------
  set(CMAKE_CXX_STANDARD 11)
  
  include(CheckCXXSourceCompiles)
  set(HAVE_MODERN_CXX TRUE)
  set(HAVE_WORKING_THREAD TRUE)
endif()

if(CUDD_WITH_SYSTEM_QSORT)
  set(USE_SYSTEM_QSORT TRUE)
endif()

# ---------------------------Old phreads checks-------------------
# if(CMAKE_USE_PTHREADS_INIT)
#   add_definitions(-DHAVE_PTHREADS=1)
#   set(HAVE_PTHREADS TRUE)
#   add_definitions("-pthread")  # TODO: Revise per target.
#   # target_link_libraries(${name} ${CMAKE_THREAD_LIBS_INIT})
# endif()
# Modern phreads checks AS PER https://stackoverflow.com/a/60537732/2746150
find_package(Threads)
list(APPEND MY_LIBS "${CMAKE_THREAD_LIBS_INIT}")


include(CheckIncludeFile)

CHECK_INCLUDE_FILE("float.h" HAVE_FLOAT_H)
if(NOT HAVE_FLOAT_H) 
  message(FATAL_ERROR "FLOAT_H header is missing.")
endif()

CHECK_INCLUDE_FILE("inttypes.h" HAVE_INTTYPES_H)
if(NOT HAVE_INTTYPES_H)
  message(FATAL_ERROR "INTTYPES_H header is missing.") 
endif()

CHECK_INCLUDE_FILE("limits.h" HAVE_LIMITS_H)
if(NOT HAVE_LIMITS_H)
  message(FATAL_ERROR "LIMITS_H header is missing.") 
endif()

CHECK_INCLUDE_FILE("stddef.h" HAVE_STDDEF_H)
if(NOT HAVE_STDDEF_H)
  message(FATAL_ERROR "STDDEF_H header is missing.") 
endif()

CHECK_INCLUDE_FILE("stdlib.h" HAVE_STDLIB_H)
if(NOT HAVE_STDLIB_H)
  message(FATAL_ERROR "STDLIB_H header is missing.") 
endif()

CHECK_INCLUDE_FILE("string.h" HAVE_STRING_H)
if(NOT HAVE_STRING_H)
  message(FATAL_ERROR "STRING_H header is missing.") 
endif()

CHECK_INCLUDE_FILE("assert.h" HAVE_ASSERT_H)
if(NOT HAVE_ASSERT_H)
  message(FATAL_ERROR "ASSERT_H header is missing.") 
endif()

CHECK_INCLUDE_FILE("math.h" HAVE_MATH_H)
if(NOT HAVE_MATH_H)
  message(FATAL_ERROR "MATH_H header is missing.") 
endif()

CHECK_INCLUDE_FILE("unistd.h" HAVE_UNISTD_H)
CHECK_INCLUDE_FILE("sys/time.h" HAVE_SYS_TIME_H)
CHECK_INCLUDE_FILE("sys/times.h" HAVE_SYS_TIMES_H)
CHECK_INCLUDE_FILE("sys/resource.h" HAVE_SYS_RESOURCE_H)
CHECK_INCLUDE_FILE("sys/wait.h" HAVE_SYS_WAIT_H)
CHECK_INCLUDE_FILE("sys/stat.h" HAVE_SYS_STAT_H)
CHECK_INCLUDE_FILE("dlfcn.h" HAVE_DLFCN_H)  # libtool
CHECK_INCLUDE_FILE("memory.h" HAVE_MEMORY_H)
CHECK_INCLUDE_FILE("strings.h" HAVE_STRINGS_H)

set(STDC_HEADERS TRUE) 

CHECK_INCLUDE_FILE("stdbool.h" HAVE__BOOL)

include(CheckTypeSize)
CHECK_TYPE_SIZE(size_t SIZE_T)
CHECK_TYPE_SIZE(uint16_t UINT16_T)
CHECK_TYPE_SIZE(uint32_t UINT32_T)
CHECK_TYPE_SIZE(ptrdiff_t PTRDIFF_T)
CHECK_TYPE_SIZE(int SIZEOF_INT)
CHECK_TYPE_SIZE(long SIZEOF_LONG)
CHECK_TYPE_SIZE("void*" SIZEOF_VOID_P)
CHECK_TYPE_SIZE("long double" SIZEOF_LONG_DOUBLE)

include(CheckSymbolExists)
include(CheckFunctionExists)
include(CheckCSourceCompiles)

set(CMAKE_REQUIRED_INCLUDES "math.h")

IF (WIN32) # Ws2_32 is required for gethostname on windows
  list(APPEND MY_LIBS "Ws2_32")
endif()
if (NOT MSVC) # there is no m.lib for MSVC https://answers.ros.org/question/392236/link-fatal-error-lnk1104-cannot-open-file-mlib/
  list(APPEND MY_LIBS "m")
endif()

# string(STRIP "${MY_LIBS}" MY_LIBS)
set(CMAKE_REQUIRED_LIBRARIES ${MY_LIBS})

if (MSVC)
    # As of 9 Sep 2023, according to https://github.com/conan-io/conan/issues/12012
    #  More or less all CMake macros in https://cmake.org/cmake/help/latest/manual/cmake-modules.7.html#utility-modules 
    #  like check_c_source_compiles, check_cxx_source_compiles, check_symbols_exists, check_cxx_symbols_exists, check_function_exists, check_include_file etc
    #  are not working properly with CMakeToolChain and Visual Studio generator
    # Trying best effort with CHECK_C_SOURCE_COMPILES

    # CHECK_FUNCTION_EXISTS(pow HAVE_POW)
    CHECK_C_SOURCE_COMPILES(
      "#include <math.h>
      int main() { double x = pow(1, 2); }
      " HAVE_POW)

    # CHECK_FUNCTION_EXISTS(sqrt HAVE_SQRT)
    CHECK_C_SOURCE_COMPILES(
      "#include <math.h>
      int main() { double x = sqrt(1); }
      " HAVE_SQRT)

    # CHECK_FUNCTION_EXISTS(strchr HAVE_STRCHR)
    CHECK_C_SOURCE_COMPILES(
      "#include <string.h>
      int main() { char* x = strchr(\"ab\", 'a'); }
      " HAVE_STRCHR)

    # CHECK_FUNCTION_EXISTS(strstr HAVE_STRSTR)
    CHECK_C_SOURCE_COMPILES(
      "#include <string.h>
      int main() { char* x = strstr(\"aaa\", \"aa\"); }
      " HAVE_STRSTR)
      
    # CHECK_FUNCTION_EXISTS(powl HAVE_POWL)
    CHECK_C_SOURCE_COMPILES(
      "#include <math.h>
      int main() { long double x = powl(3, 4); }
      " HAVE_POWL)

    # MSVC have this function in diffent libarary; assuming it does not exist https://stackoverflow.com/a/8267991/2746150
    # CHECK_FUNCTION_EXISTS(gethostname HAVE_GETHOSTNAME)
    CHECK_C_SOURCE_COMPILES(
      "#include <winsock2.h>
      int main() { char hostname[1024]; gethostname(hostname, 1023); }
      " HAVE_GETHOSTNAME)

    # MSVC does not probably have this function 
    CHECK_FUNCTION_EXISTS(getrlimit HAVE_GETRLIMIT)

    # MSVC does not probably have this function 
    CHECK_FUNCTION_EXISTS(getrusage HAVE_GETRUSAGE)

    # MSVC does not probably have this function 
    CHECK_FUNCTION_EXISTS(sysconf HAVE_SYSCONF)

else()
    CHECK_FUNCTION_EXISTS(pow HAVE_POW)
    CHECK_FUNCTION_EXISTS(sqrt HAVE_SQRT)
    CHECK_FUNCTION_EXISTS(strchr HAVE_STRCHR)
    CHECK_FUNCTION_EXISTS(strstr HAVE_STRSTR)
    CHECK_FUNCTION_EXISTS(powl HAVE_POWL)
    CHECK_FUNCTION_EXISTS(gethostname HAVE_GETHOSTNAME)
    CHECK_FUNCTION_EXISTS(getrlimit HAVE_GETRLIMIT)
    CHECK_FUNCTION_EXISTS(getrusage HAVE_GETRUSAGE)
    CHECK_FUNCTION_EXISTS(sysconf HAVE_SYSCONF)
endif()

if(NOT HAVE_POW)
  message(FATAL_ERROR "pow function is missing.")
endif()
if(NOT HAVE_SQRT)
  message(FATAL_ERROR "sqrt function is missing.")
endif()
if(NOT HAVE_STRCHR)
  message(FATAL_ERROR "strchr function is missing.")
endif()
if(NOT HAVE_STRSTR)
  message(FATAL_ERROR "strstr function is missing.")
endif()
if(NOT HAVE_GETHOSTNAME)
  message(FATAL_ERROR "gethostname function is missing.")
endif()


CHECK_C_SOURCE_COMPILES(
   "#include <math.h>
   int main() { double x = INFINITY; }
   " HAVE_IEEE_754)


######################## End compiler configurations ####################

##################### Begin cmake configuration ###################

# Set project directories.
set(PROJECT_SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}")
set(CUDD_SOURCE_DIR "${PROJECT_SOURCE_DIR}/cudd")
set(CUDD_CPP_DIR "${PROJECT_SOURCE_DIR}/cplusplus")
set(CUDD_UTIL_DIR "${PROJECT_SOURCE_DIR}/util")
set(CUDD_EPD_DIR "${PROJECT_SOURCE_DIR}/epd")
set(CUDD_MTR_DIR "${PROJECT_SOURCE_DIR}/mtr")
set(CUDD_ST_DIR "${PROJECT_SOURCE_DIR}/st")
set(CUDD_CPP_DIR "${PROJECT_SOURCE_DIR}/cplusplus")
set(CUDD_DDDMP_DIR "${PROJECT_SOURCE_DIR}/dddmp")

######################## End cmake configuration ###################

########################## Begin includes ###############################

configure_file(config-cmake.h.in "config.h" @ONLY)

SET(MY_SOURCE_FILES 
  "${CUDD_SOURCE_DIR}/cuddAddAbs.c"
  "${CUDD_SOURCE_DIR}/cuddAddApply.c"
  "${CUDD_SOURCE_DIR}/cuddAddFind.c"
  "${CUDD_SOURCE_DIR}/cuddAddInv.c"
  "${CUDD_SOURCE_DIR}/cuddAddIte.c"
  "${CUDD_SOURCE_DIR}/cuddAddNeg.c"
  "${CUDD_SOURCE_DIR}/cuddAddWalsh.c"
  "${CUDD_SOURCE_DIR}/cuddAndAbs.c"
  "${CUDD_SOURCE_DIR}/cuddAnneal.c"
  "${CUDD_SOURCE_DIR}/cuddApa.c"
  "${CUDD_SOURCE_DIR}/cuddAPI.c"
  "${CUDD_SOURCE_DIR}/cuddApprox.c"
  "${CUDD_SOURCE_DIR}/cuddBddAbs.c"
  "${CUDD_SOURCE_DIR}/cuddBddCorr.c"
  "${CUDD_SOURCE_DIR}/cuddBddIte.c"
  "${CUDD_SOURCE_DIR}/cuddBridge.c"
  "${CUDD_SOURCE_DIR}/cuddCache.c"
  "${CUDD_SOURCE_DIR}/cuddCheck.c"
  "${CUDD_SOURCE_DIR}/cuddClip.c"
  "${CUDD_SOURCE_DIR}/cuddCof.c"
  "${CUDD_SOURCE_DIR}/cuddCompose.c"
  "${CUDD_SOURCE_DIR}/cuddDecomp.c"
  "${CUDD_SOURCE_DIR}/cuddEssent.c"
  "${CUDD_SOURCE_DIR}/cuddExact.c"
  "${CUDD_SOURCE_DIR}/cuddExport.c"
  "${CUDD_SOURCE_DIR}/cuddGenCof.c"
  "${CUDD_SOURCE_DIR}/cuddGenetic.c"
  "${CUDD_SOURCE_DIR}/cuddGroup.c"
  "${CUDD_SOURCE_DIR}/cuddHarwell.c"
  "${CUDD_SOURCE_DIR}/cuddInit.c"
  "${CUDD_SOURCE_DIR}/cuddInteract.c"
  "${CUDD_SOURCE_DIR}/cuddLCache.c"
  "${CUDD_SOURCE_DIR}/cuddLevelQ.c"
  "${CUDD_SOURCE_DIR}/cuddLinear.c"
  "${CUDD_SOURCE_DIR}/cuddLiteral.c"
  "${CUDD_SOURCE_DIR}/cuddMatMult.c"
  "${CUDD_SOURCE_DIR}/cuddPriority.c"
  "${CUDD_SOURCE_DIR}/cuddRead.c"
  "${CUDD_SOURCE_DIR}/cuddRef.c"
  "${CUDD_SOURCE_DIR}/cuddReorder.c"
  "${CUDD_SOURCE_DIR}/cuddSat.c"
  "${CUDD_SOURCE_DIR}/cuddSign.c"
  "${CUDD_SOURCE_DIR}/cuddSolve.c"
  "${CUDD_SOURCE_DIR}/cuddSplit.c"
  "${CUDD_SOURCE_DIR}/cuddSubsetHB.c"
  "${CUDD_SOURCE_DIR}/cuddSubsetSP.c"
  "${CUDD_SOURCE_DIR}/cuddSymmetry.c"
  "${CUDD_SOURCE_DIR}/cuddTable.c"
  "${CUDD_SOURCE_DIR}/cuddUtil.c"
  "${CUDD_SOURCE_DIR}/cuddWindow.c"
  "${CUDD_SOURCE_DIR}/cuddZddCount.c"
  "${CUDD_SOURCE_DIR}/cuddZddFuncs.c"
  "${CUDD_SOURCE_DIR}/cuddZddGroup.c"
  "${CUDD_SOURCE_DIR}/cuddZddIsop.c"
  "${CUDD_SOURCE_DIR}/cuddZddLin.c"
  "${CUDD_SOURCE_DIR}/cuddZddMisc.c"
  "${CUDD_SOURCE_DIR}/cuddZddPort.c"
  "${CUDD_SOURCE_DIR}/cuddZddReord.c"
  "${CUDD_SOURCE_DIR}/cuddZddSetop.c"
  "${CUDD_SOURCE_DIR}/cuddZddSymm.c"
  "${CUDD_SOURCE_DIR}/cuddZddUtil.c"

  "${CUDD_EPD_DIR}/epd.c"

  "${CUDD_MTR_DIR}/mtrBasic.c"
  "${CUDD_MTR_DIR}/mtrGroup.c"

  "${CUDD_ST_DIR}/st.c"

  "${CUDD_UTIL_DIR}/cpu_stats.c"
  "${CUDD_UTIL_DIR}/cpu_time.c"
  "${CUDD_UTIL_DIR}/cstringstream.c"
  "${CUDD_UTIL_DIR}/datalimit.c"
  "${CUDD_UTIL_DIR}/pathsearch.c"
  "${CUDD_UTIL_DIR}/pipefork.c"
  "${CUDD_UTIL_DIR}/prtime.c"
  "${CUDD_UTIL_DIR}/safe_mem.c"
  "${CUDD_UTIL_DIR}/strsav.c"
  "${CUDD_UTIL_DIR}/texpand.c"
  "${CUDD_UTIL_DIR}/ucbqsort.c"
)

if(CUDD_ENABLE_OBJ)
  list(APPEND MY_SOURCE_FILES "${CUDD_CPP_DIR}/cuddObj.cc")
endif()

if(CUDD_ENABLE_DDDMP)
  list(APPEND MY_SOURCE_FILES 
    "${CUDD_DDDMP_DIR}/dddmpBinary.c"
    "${CUDD_DDDMP_DIR}/dddmpConvert.c"
    "${CUDD_DDDMP_DIR}/dddmpDbg.c"
    "${CUDD_DDDMP_DIR}/dddmpLoad.c"
    "${CUDD_DDDMP_DIR}/dddmpLoadCnf.c"
    "${CUDD_DDDMP_DIR}/dddmpNodeAdd.c"
    "${CUDD_DDDMP_DIR}/dddmpNodeBdd.c"
    "${CUDD_DDDMP_DIR}/dddmpNodeCnf.c"
    "${CUDD_DDDMP_DIR}/dddmpStoreAdd.c"
    "${CUDD_DDDMP_DIR}/dddmpStoreBdd.c"
    "${CUDD_DDDMP_DIR}/dddmpStoreCnf.c"
    "${CUDD_DDDMP_DIR}/dddmpStoreMisc.c"
    "${CUDD_DDDMP_DIR}/dddmpUtil.c"
  )
endif()

add_library(cudd ${MY_SOURCE_FILES})

# headers
target_sources(cudd
  INTERFACE "${CUDD_SOURCE_DIR}/cudd.h"
  PRIVATE "${CUDD_SOURCE_DIR}/cuddInt.h"

  PRIVATE "${CUDD_EPD_DIR}/epd.h"
  PRIVATE "${CUDD_EPD_DIR}/epdInt.h"

  PRIVATE "${CUDD_MTR_DIR}/mtr.h"
  PRIVATE "${CUDD_MTR_DIR}/mtrInt.h"

  PRIVATE "${CUDD_ST_DIR}/st.h"

  PRIVATE "${CUDD_UTIL_DIR}/cstringstream.h"
  PRIVATE "${CUDD_UTIL_DIR}/util.h"
)

if(CUDD_ENABLE_OBJ)
  target_sources(cudd INTERFACE "${CUDD_CPP_DIR}/cuddObj.hh")
endif()

if(CUDD_ENABLE_DDDMP)
  target_sources(cudd 
    INTERFACE "${CUDD_DDDMP_DIR}/dddmp.h"
    PRIVATE "${CUDD_DDDMP_DIR}/dddmpInt.h"
  )
endif()

set(CUDD_ALL_INCLUDE_DIRS "")
list(APPEND CUDD_ALL_INCLUDE_DIRS 
  PRIVATE "${CMAKE_CURRENT_BINARY_DIR}"
  PRIVATE "${CUDD_SOURCE_DIR}"
  PRIVATE "${CUDD_UTIL_DIR}"
  PRIVATE "${CUDD_EPD_DIR}"
  PRIVATE "${CUDD_MTR_DIR}"
  PRIVATE "${CUDD_ST_DIR}"
  PRIVATE "${CUDD_UTIL_DIR}"
  PRIVATE "${CUDD_DDDMP_DIR}"
)
if(CUDD_ENABLE_OBJ)
  list(APPEND CUDD_ALL_INCLUDE_DIRS PRIVATE "${CUDD_CPP_DIR}")
endif()
if(CUDD_ENABLE_DDDMP)
  list(APPEND CUDD_ALL_INCLUDE_DIRS PRIVATE "${CUDD_DDDMP_DIR}")
endif()

target_include_directories(cudd ${CUDD_ALL_INCLUDE_DIRS})

# target_include_directories(cudd 
#   PRIVATE "${CMAKE_CURRENT_BINARY_DIR}"
#   PRIVATE "${CUDD_SOURCE_DIR}"
#   PRIVATE "${CUDD_UTIL_DIR}"
#   PRIVATE "${CUDD_EPD_DIR}"
#   PRIVATE "${CUDD_MTR_DIR}"
#   PRIVATE "${CUDD_ST_DIR}"
#   PRIVATE "${CUDD_UTIL_DIR}"
#   PRIVATE "${CUDD_DDDMP_DIR}"
# )
# if(CUDD_ENABLE_OBJ)
#   target_include_directories(cudd PRIVATE "${CUDD_CPP_DIR}")
# endif()
# if(CUDD_ENABLE_DDDMP)
#   target_include_directories(cudd PRIVATE "${CUDD_DDDMP_DIR}")
# endif()

####################### End includes ####################################


####################### Libaries ########################################
target_link_libraries(cudd PRIVATE ${MY_LIBS})

# # Modern CMake equivalent to directly setting -fPIC gcc compiler option
include(CheckPIESupported)
check_pie_supported(OUTPUT_VARIABLE output LANGUAGES C CXX)
set_property(TARGET cudd PROPERTY POSITION_INDEPENDENT_CODE TRUE)
# if(NOT CMAKE_C_LINK_PIE_SUPPORTED)
#   message(WARNING "PIE is not supported at link time: ${output}.\n"
#                   "PIE link options will not be passed to linker.")
# endif()

if(BUILD_SHARED_LIBS)
  target_compile_definitions(cudd PRIVATE "EXPORTING_CUDD")
endif()  

####################### End libaries ####################################


####################### install #########################################

set(installable_libs cudd)
# if(BUILD_SHARED_LIBS)
#   install(TARGETS ${installable_libs} DESTINATION bin)
# else()  
#   install(TARGETS ${installable_libs} DESTINATION lib)
# endif()
install(TARGETS ${installable_libs} 
  RUNTIME DESTINATION bin
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
)

install(FILES "${CUDD_SOURCE_DIR}/cudd.h" DESTINATION include)
if(CUDD_ENABLE_OBJ)
  install(FILES "${CUDD_CPP_DIR}/cuddObj.hh" DESTINATION include)
endif()
if(CUDD_ENABLE_DDDMP)
  install(FILES "${CUDD_DDDMP_DIR}/dddmp.h" DESTINATION include)
endif()

####################### end install ######################################


####################### TEST #############################################
if(CUDD_BUILD_TESTS)

  if(WIN32)
    find_program (SH_PROGRAM sh HINTS "C:/Program Files/Git/bin/")
  else()
    find_program (SH_PROGRAM sh)
  endif()

  if(CUDD_ENABLE_OBJ)
  
    # test_obj.test expects test executables placed into cplusplus subfolder
    add_custom_target(build-time-make-directory ALL
      COMMAND ${CMAKE_COMMAND} -E make_directory "${CMAKE_BINARY_DIR}/cplusplus")

    if(WIN32)
      set(EXEEXT ".exe")
    else()
      set(EXEEXT "")
    endif()
    set(srcdir "..") # assuming build directory is one level below source folder
    configure_file("${CUDD_CPP_DIR}/test_obj.test.in" "${CMAKE_BINARY_DIR}/test_obj.test" @ONLY)
  
    # testobj
    add_executable(testobj "${CUDD_CPP_DIR}/testobj.cc" "${CUDD_CPP_DIR}/cuddObj.hh")
    target_include_directories(testobj ${CUDD_ALL_INCLUDE_DIRS})
    target_link_libraries(testobj PRIVATE ${MY_LIBS})
    target_link_libraries(testobj PUBLIC cudd)

    # test_obj.test expects test executables placed into cplusplus subfolder
    add_custom_command(TARGET testobj POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy 
      $<TARGET_FILE_DIR:testobj>/testobj${EXEEXT}
      ${CMAKE_BINARY_DIR}/cplusplus) 

    # testmulti
    add_executable(testmulti "${CUDD_CPP_DIR}/testmulti.cc" "${CUDD_CPP_DIR}/cuddObj.hh")
    target_include_directories(testmulti ${CUDD_ALL_INCLUDE_DIRS})
    target_link_libraries(testmulti PRIVATE ${MY_LIBS})
    target_link_libraries(testmulti PUBLIC cudd)

    # test_obj.test expects test executables placed into cplusplus subfolder
    add_custom_command(TARGET testmulti POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy 
      $<TARGET_FILE_DIR:testmulti>/testmulti${EXEEXT}
      ${CMAKE_BINARY_DIR}/cplusplus) 

    if (SH_PROGRAM)
      enable_testing()
      # define tests
      add_test(
        NAME test_obj 
        COMMAND ${SH_PROGRAM} "test_obj.test"
        WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
      )
      # hack to display verbose testing after build
      add_custom_command(TARGET testmulti POST_BUILD
        COMMAND ${CMAKE_CTEST_COMMAND} -C ${CMAKE_BUILD_TYPE} --verbose
        COMMENT "\n\n\n---===== PLEASE REVIEW TEST OUTPUT BELOW =====---\n\n\n"
      )      
      # TODO rewrite test_obj.test to eliminate sed errors and return correct exit code
    endif(SH_PROGRAM)
  endif(CUDD_ENABLE_OBJ)
endif(CUDD_BUILD_TESTS)
####################### end TEST #########################################