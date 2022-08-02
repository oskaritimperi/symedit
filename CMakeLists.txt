cmake_minimum_required(VERSION 3.21)

project(symedit)

find_package(Qt5 COMPONENTS Widgets Quick REQUIRED)

set(CMAKE_AUTORCC ON)
set(CMAKE_AUTOMOC ON)

add_executable(symedit WIN32
    icon.rc
    locale.qrc
    main.cpp
    qml.qrc
    symbol.cpp
    symbol.h
    symedit.cpp
    symedit.h
)

target_compile_features(symedit PRIVATE cxx_std_11)

target_link_libraries(symedit PRIVATE Qt5::Widgets Qt5::Quick)

set_property(SOURCE symbol.cpp PROPERTY SKIP_AUTOMOC TRUE)

option(SYMEDIT_BUILD_HELP "Build the Symbol editor HTML help. Requires Python3.")

if(SYMEDIT_BUILD_HELP)
    find_package(Python3 COMPONENTS Interpreter REQUIRED)

    if(WIN32)
        set(VENV_PYTHON "${CMAKE_CURRENT_BINARY_DIR}/venv/Scripts/python.exe")
        set(SPHINX_BUILD "${CMAKE_CURRENT_BINARY_DIR}/venv/Scripts/sphinx-build.exe")
    else()
        set(VENV_PYTHON "${CMAKE_CURRENT_BINARY_DIR}/venv/bin/python")
        set(SPHINX_BUILD "${CMAKE_CURRENT_BINARY_DIR}/venv/bin/sphinx-build")
    endif()

    add_custom_command(OUTPUT "${CMAKE_CURRENT_BINARY_DIR}/venv/stamp"
        COMMAND Python3::Interpreter -m venv "${CMAKE_CURRENT_BINARY_DIR}/venv"
        COMMAND "${VENV_PYTHON}" -m pip install sphinx
        COMMAND "${CMAKE_COMMAND}" -E touch "${CMAKE_CURRENT_BINARY_DIR}/venv/stamp"
    )

    file(GLOB_RECURSE HELP_ENG_FILES LIST_DIRECTORIES FALSE help/eng/*.rst help/eng/*.py help/eng/*.png)

    add_custom_command(OUTPUT "${CMAKE_CURRENT_BINARY_DIR}/help/eng/html/index.html"
        COMMAND "${SPHINX_BUILD}" -q -b html "${CMAKE_CURRENT_SOURCE_DIR}/help/eng" "${CMAKE_CURRENT_BINARY_DIR}/help/eng"
        COMMAND "${CMAKE_COMMAND}" -E touch "${CMAKE_CURRENT_BINARY_DIR}/help/eng/html/index.html"
        DEPENDS "${CMAKE_CURRENT_BINARY_DIR}/venv/stamp" ${HELP_ENG_FILES}
    )

    file(GLOB_RECURSE HELP_FIN_FILES LIST_DIRECTORIES FALSE help/fin/*.rst help/fin/*.py help/fin/*.png)

    add_custom_command(OUTPUT "${CMAKE_CURRENT_BINARY_DIR}/help/fin/html/index.html"
        COMMAND "${SPHINX_BUILD}" -q -b html "${CMAKE_CURRENT_SOURCE_DIR}/help/fin" "${CMAKE_CURRENT_BINARY_DIR}/help/fin"
        COMMAND "${CMAKE_COMMAND}" -E touch "${CMAKE_CURRENT_BINARY_DIR}/help/fin/html/index.html"
        DEPENDS "${CMAKE_CURRENT_BINARY_DIR}/venv/stamp" ${HELP_FIN_FILES}
    )

    add_custom_target(build-help ALL
        DEPENDS
            "${CMAKE_CURRENT_BINARY_DIR}/help/eng/html/index.html"
            "${CMAKE_CURRENT_BINARY_DIR}/help/fin/html/index.html"
    )
endif()

install(TARGETS symedit RUNTIME DESTINATION $<CONFIG>)
install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/help/eng/html/ DESTINATION help/eng)
install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/help/fin/html/ DESTINATION help/fin)
