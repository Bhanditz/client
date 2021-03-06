# TODO: OSX and LIB_ONLY seem to require this to go to binary dir only
if(NOT TOKEN_AUTH_ONLY)
endif()

find_package(Qt5 5.6 COMPONENTS Core Network Xml Concurrent REQUIRED)
if (Qt5Core_VERSION VERSION_LESS 5.9.0)
message(STATUS "For HTTP/2 support, compile with Qt 5.9 or higher.")
endif()
get_target_property (QT_QMAKE_EXECUTABLE Qt5::qmake IMPORTED_LOCATION)
message(STATUS "Using Qt ${Qt5Core_VERSION} (${QT_QMAKE_EXECUTABLE})")


if(NOT TOKEN_AUTH_ONLY)
    find_package(Qt5Keychain REQUIRED)
endif()

if(NOT MSVC)
  if(NOT (CMAKE_SYSTEM_PROCESSOR MATCHES "^(alpha|parisc|hppa)") AND NOT CMAKE_CROSSCOMPILING)
    if((CMAKE_CXX_COMPILER_ID MATCHES "GNU") AND (CMAKE_CXX_COMPILER_VERSION VERSION_LESS 4.9))
      set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fstack-protector --param=ssp-buffer-size=4")
      set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fstack-protector --param=ssp-buffer-size=4")
    else()
      set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fstack-protector-strong")
      set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fstack-protector-strong")
    endif()
  endif()

  string(TOLOWER "${CMAKE_BUILD_TYPE}" CMAKE_BUILD_TYPE_LOWER)
  if(CMAKE_BUILD_TYPE_LOWER MATCHES "(release|relwithdebinfo|minsizerel)")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -D_FORTIFY_SOURCE=2")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -D_FORTIFY_SOURCE=2")
  endif()
endif()

if(WIN32)
  # Enable DEP & ASLR
  set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -Wl,--nxcompat -Wl,--dynamicbase")
  set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -Wl,--nxcompat -Wl,--dynamicbase")
elseif(UNIX AND NOT APPLE)
  set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -Wl,-z,relro -Wl,-z,now")
  set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -Wl,-z,relro -Wl,-z,now")
endif()

add_subdirectory(csync)
add_subdirectory(libsync)
if (NOT BUILD_LIBRARIES_ONLY)
    if (BUILD_GUI)
        add_subdirectory(gui)
    endif()
    add_subdirectory(cmd)

    if (WITH_CRASHREPORTER)
        add_subdirectory(3rdparty/libcrashreporter-qt)
        add_subdirectory(crashreporter)
    endif()
endif(NOT BUILD_LIBRARIES_ONLY)

find_program(KRAZY2_EXECUTABLE krazy2)
if(KRAZY2_EXECUTABLE)
    # s/y k/y ALL k/ for building this target always
    add_custom_target( krazy krazy2 --check-sets c++,qt4,foss
                       ${PROJECT_SOURCE_DIR}/src/libsync/*.ui
                       ${PROJECT_SOURCE_DIR}/src/libsync/*.h
                       ${PROJECT_SOURCE_DIR}/src/libsync/*.cpp
                       ${PROJECT_SOURCE_DIR}/src/gui/*.ui
                       ${PROJECT_SOURCE_DIR}/src/gui/*.h
                       ${PROJECT_SOURCE_DIR}/src/gui/*.cpp
                       ${PROJECT_SOURCE_DIR}/src/cmd/*.h
                       ${PROJECT_SOURCE_DIR}/src/cmd/*.cpp
)
endif()

