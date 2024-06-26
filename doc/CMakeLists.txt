
# find doxygen
find_package(Doxygen REQUIRED)

# set the input and output documentation paths
set(DOXYGEN_INPUT_DIR ${PROJECT_SOURCE_DIR})
set(DOXYGEN_OUTPUT_DIR ${CMAKE_BINARY_DIR}/doc)

# see if we can generate the CHM documentation
if(SFML_OS_WINDOWS)
    # if HHC is found, we can generate the CHM (compressed HTML) output
    find_program(DOXYGEN_HHC_PROGRAM
                 NAMES hhc.exe
                 PATHS "C:/Program Files/HTML Help Workshop" "C:/Program Files (x86)/HTML Help Workshop"
                 DOC "HTML Help Compiler program")
    if(DOXYGEN_HHC_PROGRAM)
        if(DOXYGEN_VERSION VERSION_LESS "1.10.0")
            set(DOXYGEN_GENERATE_HTMLHELP YES)
        else()
            message("Due to conflicts with the HTML output settings in Doxygen ${DOXYGEN_VERSION}, the HTML Help generation will be disabled")
            set(DOXYGEN_GENERATE_HTMLHELP NO)
        endif()
    else()
        set(DOXYGEN_GENERATE_HTMLHELP NO)
    endif()
else()
    set(DOXYGEN_HHC_PROGRAM)
    set(DOXYGEN_GENERATE_HTMLHELP NO)
endif()

# configure the source Doxyfile by copying it and replacing all @variables@
set(DOXYGEN_CONFIGURED_INPUT ${DOXYGEN_OUTPUT_DIR}/doxyfile)
configure_file(${DOXYGEN_INPUT_DIR}/doc/doxyfile.in ${DOXYGEN_CONFIGURED_INPUT} @ONLY)
configure_file(${DOXYGEN_INPUT_DIR}/doc/header.html.in ${DOXYGEN_OUTPUT_DIR}/header.html @ONLY)

# copy the files needed by the documentation
configure_file(${DOXYGEN_INPUT_DIR}/doc/doxygen.css ${DOXYGEN_OUTPUT_DIR}/html/doxygen.css COPYONLY)
configure_file(${DOXYGEN_INPUT_DIR}/doc/searchOverrides.css ${DOXYGEN_OUTPUT_DIR}/html/searchOverrides.css COPYONLY)

# target setup
add_custom_target(doc ALL
                  COMMAND ${CMAKE_COMMAND} -E echo_append "Building API Documentation..."
                  COMMAND ${DOXYGEN_EXECUTABLE} ${DOXYGEN_CONFIGURED_INPUT}
                  COMMAND ${CMAKE_COMMAND} -E echo "Done."
                  WORKING_DIRECTORY ${DOXYGEN_INPUT_DIR})

# setup install rules
install(DIRECTORY ${DOXYGEN_OUTPUT_DIR}/html
        DESTINATION ${INSTALL_MISC_DIR}/doc
        COMPONENT doc)
if(DOXYGEN_HHC_PROGRAM)
    install(FILES ${DOXYGEN_OUTPUT_DIR}/CSFML.chm
            DESTINATION ${INSTALL_MISC_DIR}/doc
            COMPONENT doc)
endif()
