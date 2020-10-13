cmake_minimum_required(VERSION 3.14.0)

project(EMSoftCI VERSION 2.0.0.0 LANGUAGES NONE)

include(ExternalProject)

set(EMSOFT_WORKSPACE ${PROJECT_SOURCE_DIR}/Workspace CACHE PATH "Root directory where repos will be cloned")

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

if("${EMSOFT_WORKSPACE}" STREQUAL "")
  message(FATAL_ERROR "EMSOFT_WORKSPACE is empty. Cloning EMsoft can not continue. \
    Please set the -EMSOFT_WORKSPACE=/Path/to/Directory/of/EMsoft to the directory where you want to clone all the DREAM3D \
    repositories. Anything in that directory may be over written.")
endif()

foreach(repo ${EMSOFT_REPOS})
  CloneRepo(NAME ${repo}
    GIT_REPO "${${repo}_BASE_URL}/${${repo}_GH_ORG}/${${repo}_REPO_NAME}.git"
    GIT_TAG "${${repo}_GIT_BRANCH}"
    ROOT_DIR "${EMSOFT_WORKSPACE}"
  )
endforeach()
