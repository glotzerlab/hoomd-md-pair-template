# TODO: Set COMPONENT_NAME to the name of your Python package.
set(COMPONENT_NAME template)

# TODO: List all host C++ source code files in _${COMPONENT_NAME}_sources.
set(_${COMPONENT_NAME}_sources
    module.cc
    )

# TODO: List all GPU C++ source code files in _${COMPONENT_NAME}_cu_sources.
set(_${COMPONENT_NAME}_cu_sources
    PotentialPairExampleGPUKernel.cu
    )

# TODO: List all Python modules in python_files.
set(python_files
    __init__.py
    pair.py
    version.py
    )

if (ENABLE_HIP)
set(_cuda_sources ${_${COMPONENT_NAME}_cu_sources})
endif (ENABLE_HIP)

hoomd_add_module(_${COMPONENT_NAME} SHARED ${_${COMPONENT_NAME}_sources} ${_cuda_sources} NO_EXTRAS)
# Alias into the HOOMD namespace so that external and symlinked components both work.
add_library(HOOMD::_${COMPONENT_NAME} ALIAS _${COMPONENT_NAME})

if (APPLE)
set_target_properties(_${COMPONENT_NAME} PROPERTIES INSTALL_RPATH "@loader_path/..;@loader_path")
else()
set_target_properties(_${COMPONENT_NAME} PROPERTIES INSTALL_RPATH "\$ORIGIN/..;\$ORIGIN")
endif()

# Link the library to its dependencies. Add or remove HOOMD extension modules (and/or external C++
# libraries) as needed.
target_link_libraries(_${COMPONENT_NAME}
                      PUBLIC HOOMD::_hoomd
                      PUBLIC HOOMD::_md
                      )

# Install the library.
install(TARGETS _${COMPONENT_NAME}
        LIBRARY DESTINATION ${PYTHON_SITE_INSTALL_DIR}/${COMPONENT_NAME}
        )

# Install the Python package.
install(FILES ${python_files}
        DESTINATION ${PYTHON_SITE_INSTALL_DIR}/${COMPONENT_NAME}
       )

# Copy the Python package to the build directory.
copy_files_to_build("${python_files}" "hoomd-component-${COMPONENT_NAME}" "*.py")

# Python tests.
add_subdirectory(pytest)
