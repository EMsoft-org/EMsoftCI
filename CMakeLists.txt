cmake_minimum_required(VERSION 3.14.0)

project(EMsoftCI VERSION 2.0.0.0 LANGUAGES NONE)

include(ExternalProject)

set(EMsoft_WORKSPACE ${EMsoftCI_SOURCE_DIR}/Workspace CACHE PATH "Root directory where repos will be cloned")

include(${PROJECT_SOURCE_DIR}/Repos.cmake)

#------------------------------------------------------------------------------
# Function to clone the repository using ExternalProject_Add
#------------------------------------------------------------------------------
function(CloneRepo)
  set(options)
  set(oneValueArgs NAME ROOT_DIR GIT_REPO GIT_TAG)
  set(multiValueArgs)
  cmake_parse_arguments(ARGS "${options}" "${oneValueArgs}" "${multiValueArgs}" ${ARGN})

  set(CLONE_DIR ${ARGS_ROOT_DIR}/${ARGS_NAME})

  message(STATUS "Cloning \"${ARGS_GIT_REPO}\" at tag \"${ARGS_GIT_TAG}\" in \"${CLONE_DIR}\"" )
  ExternalProject_Add(${ARGS_NAME}
    GIT_REPOSITORY ${ARGS_GIT_REPO}
    GIT_TAG ${ARGS_GIT_TAG}
    GIT_PROGRESS ON

    SOURCE_DIR ${CLONE_DIR}

    UPDATE_COMMAND ""
    PATCH_COMMAND ""
    CONFIGURE_COMMAND ""
    BUILD_COMMAND ""
    INSTALL_COMMAND ""
    TEST_COMMAND ""

    TMP_DIR ${PROJECT_BINARY_DIR}/${ARGS_NAME}/temp
    STAMP_DIR ${PROJECT_BINARY_DIR}/${ARGS_NAME}/stamp
    DOWNLOAD_DIR ${PROJECT_BINARY_DIR}/${ARGS_NAME}/download
    BINARY_DIR ${PROJECT_BINARY_DIR}/${ARGS_NAME}/binary
    INSTALL_DIR ${PROJECT_BINARY_DIR}/${ARGS_NAME}/install
    
    LOG_DOWNLOAD ON
    LOG_UPDATE ON
    LOG_CONFIGURE ON
    LOG_BUILD ON
    LOG_TEST ON
    LOG_INSTALL ON
  )
endfunction()

if("${EMsoft_WORKSPACE}" STREQUAL "")
  message(FATAL_ERROR "EMsoft_WORKSPACE is empty. Cloning DREAM.3D can not continue. \
    Please set the -DEMsoft_WORKSPACE=/Path/to/Directory/of/EMsoft to the directory where you want to clone all the EMsoft \
    repositories. Anything in that directory may be over written.")
endif()

foreach(plugin ${EMsoft_Base_Repos})
  CloneRepo(NAME ${plugin}
    GIT_REPO "${${plugin}_BASE_URL}/${${plugin}_GH_ORG}/${${plugin}_REPO_NAME}.git"
    GIT_TAG "${${plugin}_GIT_BRANCH}"
    ROOT_DIR "${EMsoft_WORKSPACE}"
  )
endforeach()

