cmake_minimum_required(VERSION 3.13)
project(libresdr_source)

file(GLOB SRC "src/*.cpp")

add_library(libresdr_source SHARED ${SRC})
target_link_libraries(libresdr_source PRIVATE sdrpp_core)
set_target_properties(libresdr_source PROPERTIES PREFIX "")

target_include_directories(libresdr_source PRIVATE "src/")
target_include_directories(libresdr_source PRIVATE "src/" "/usr/include/sdrpp_core/" "/usr/include/sdrpp_core/imgui")

if (MSVC)
    target_compile_options(libresdr_source PRIVATE /O2 /Ob2 /std:c++17 /EHsc)
elseif (CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    target_compile_options(libresdr_source PRIVATE -O3 -std=c++17 -Wno-unused-command-line-argument -undefined dynamic_lookup)
else ()
    target_compile_options(libresdr_source PRIVATE -O3 -std=c++17)
endif ()

if (MSVC)
    # Lib path
    target_link_directories(libresdr_source PRIVATE "C:/Program Files/PothosSDR/lib/")
    target_include_directories(libresdr_source PRIVATE "C:/Program Files/PothosSDR/include/")

    target_link_libraries(libresdr_source PRIVATE libiio)
    target_link_libraries(libresdr_source PRIVATE libad9361)
elseif (ANDROID)
    target_include_directories(libresdr_source PUBLIC
        /mnt/android_sdr/libiio
        /mnt/android_sdr/libad9361-iio
    )

    target_link_libraries(libresdr_source PUBLIC
        /mnt/android_sdr/output/libxml2/${ANDROID_ABI}/libxml2.so
        /mnt/android_sdr/output/libiio/${ANDROID_ABI}/libiio.so
        /mnt/android_sdr/output/libad9361/${ANDROID_ABI}/libad9361.so
    )
else (MSVC)
    if (${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
        target_include_directories(libresdr_source PRIVATE "/Library/Frameworks/iio.framework/Headers")
        target_link_libraries(libresdr_source PRIVATE "/Library/Frameworks/iio.framework/iio")

        target_include_directories(libresdr_source PRIVATE "/Library/Frameworks/ad9361.framework/Headers")
        target_link_libraries(libresdr_source PRIVATE "/Library/Frameworks/ad9361.framework/ad9361")
    else()
        find_package(PkgConfig)

        pkg_check_modules(LIBIIO REQUIRED libiio)
        pkg_check_modules(LIBAD9361 REQUIRED libad9361)

        target_include_directories(libresdr_source PRIVATE ${LIBIIO_INCLUDE_DIRS})
        target_link_directories(libresdr_source PRIVATE ${LIBIIO_LIBRARY_DIRS})
        target_link_libraries(libresdr_source PRIVATE ${LIBIIO_LIBRARIES})

        target_include_directories(libresdr_source PRIVATE ${LIBAD9361_INCLUDE_DIRS})
        target_link_directories(libresdr_source PRIVATE ${LIBAD9361_LIBRARY_DIRS})
        target_link_libraries(libresdr_source PRIVATE ${LIBAD9361_LIBRARIES})
    endif()

    target_include_directories(libresdr_source PRIVATE ${LIBAD9361_INCLUDE_DIRS})
endif ()

# Install directives
install(TARGETS libresdr_source DESTINATION /usr/lib/sdrpp/plugins)
