# ===========================================================
#
# This file is a part of digiKam project
# <a href="http://www.digikam.org">http://www.digikam.org</a>
#
# @date   2010-06-16
# @brief  LibFace library interface for KDE
#
# @author Copyright (C) 2010-2012 by Gilles Caulier
#         <a href="mailto:caulier dot gilles at gmail dot com">caulier dot gilles at gmail dot com</a>
#
# This program is free software; you can redistribute it
# and/or modify it under the terms of the GNU General
# Public License as published by the Free Software Foundation;
# either version 2, or (at your option)
# any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
# GNU General Public License for more details.
#
# ============================================================

PROJECT(libkface)
cmake_minimum_required(VERSION 2.8)

find_package(OpenCV REQUIRED)

#-------------------------------------------------------------------------------
#version
add_library(cvblobs
    blob.cpp
    BlobContour.cpp
    BlobOperators.cpp
    BlobProperties.cpp
    BlobResult.cpp
    ComponentLabeling.cpp
    blob.h
    BlobContour.h
    BlobLibraryConfiguration.h
    BlobOperators.h
    BlobProperties.h
    BlobResult.h
    ComponentLabeling.h)

add_library(libconfig
    grammar.c
    libconfig.c
    libconfigcpp.cc
    scanctx.c
    scanner.c
    strbuf.c
    grammar.h
    libconfig.h
    libconfig.hh
    parsectx.h
    scanctx.h
    scanner.h
    strbuf.h
    wincompat.h)
link_directories(${OpenCV_LIB_DIR})

add_library(libopentld
    ImAcq.cpp
    BB.cpp
    BBPredict.cpp
    FBTrack.cpp
    Lk.cpp
    Median.cpp
    Clustering.cpp
    DetectionResult.cpp
    DetectorCascade.cpp
    EnsembleClassifier.cpp
    ForegroundDetector.cpp
    MedianFlowTracker.cpp
    NNClassifier.cpp
    TLD.cpp
    TLDUtil.cpp
    VarianceFilter.cpp
    ImAcq.h
    BB.h
    BBPredict.h
    FBTrack.h
    Lk.h
    Median.h
    Clustering.h
    DetectionResult.h
    DetectorCascade.h
    EnsembleClassifier.h
    ForegroundDetector.h
    IntegralImage.h
    MedianFlowTracker.h
    NNClassifier.h
    NormalizedPatch.h
    TLD.h
    TLDUtil.h
    VarianceFilter.h)
target_link_libraries(libopentld ${OpenCV_LIBS})
include_directories(main ${OpenCV_INCLUDE_DIRS})
#build type
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release CACHE STRING
        "Choose the type of build, options are: None Debug Release RelWithDebInfo MinSizeRel."
        FORCE)
endif(NOT CMAKE_BUILD_TYPE)


set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fopenmp -fPIC")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fopenmp -fPIC")
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -lgomp")

set(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)
set(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/lib)


configure_file("${PROJECT_SOURCE_DIR}/OpenTLDConfig.cmake.in" "${PROJECT_BINARY_DIR}/OpenTLDConfig.cmake" @ONLY)



MESSAGE(STATUS "----------------------------------------------------------------------------------")
MESSAGE(STATUS "Starting CMake configuration for: libkface")

FIND_PACKAGE(Qt4 4.8.1 REQUIRED)
FIND_PACKAGE(KDE4 REQUIRED)

INCLUDE(KDE4Defaults)
INCLUDE(MacroLibrary)
INCLUDE(MacroOptionalAddSubdirectory)
INCLUDE(MacroOptionalFindPackage)
INCLUDE(FindPackageHandleStandardArgs)

ADD_DEFINITIONS (${QT_DEFINITIONS} ${QT_QTDBUS_DEFINITIONS} ${KDE4_DEFINITIONS} -DQT_USE_FAST_CONCATENATION -DQT_USE_FAST_OPERATOR_PLUS )
INCLUDE_DIRECTORIES (${QDBUS_INCLUDE_DIRS} ${KDE4_INCLUDES})

SET(LIBKFACE_AREA_CODE_GENERAL 51005)
ADD_DEFINITIONS(-DKDE_DEFAULT_DEBUG_AREA=${LIBKFACE_AREA_CODE_GENERAL})

SET(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/cmake/modules)

# Force usage of local copy temporarily in this branch
#FIND_PACKAGE(LibFace)

# BEGIN SPECIAL OPENCV HANDLING

# On some systems, OpenCV can be found using FIND_PACKAGE(OpenCV), for other systems
# we ship our own version of FindOpenCV.cmake. But that does not work on all systems.
# Therefore, first try finding OpenCV using FIND_PACKAGE(OpenCV), and if that fails,
# add our FindOpenCV.cmake to the search path and search again.

MESSAGE(STATUS "First try at finding OpenCV...")
FIND_PACKAGE(OpenCV)

IF (NOT OpenCV_LIBRARIES AND NOT OpenCV_LIBS)
  MESSAGE(STATUS "Could not find OpenCV normally, trying internal FindOpenCV.cmake")
  SET(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/cmake/modules_opencv)
  FIND_PACKAGE(OpenCV REQUIRED)
ELSE (NOT OpenCV_LIBRARIES AND NOT OpenCV_LIBS)
  MESSAGE(STATUS "Great, found OpenCV on the first try.")
ENDIF (NOT OpenCV_LIBRARIES AND NOT OpenCV_LIBS)

MESSAGE(STATUS "OpenCV Root directory is ${OpenCV_DIR}")

# there are two versions of FindOpenCV.cmake in the wild, one defining
# OpenCV_LIBRARIES, the other defining OpenCV_LIBS. Make sure we handle
# both cases.
IF (NOT OpenCV_LIBRARIES)
  SET(OpenCV_LIBRARIES ${OpenCV_LIBS})
ENDIF (NOT OpenCV_LIBRARIES)

# same story with OpenCV_INCLUDE_DIRS and OpenCV_INCLUDE_DIR:
IF (NOT OpenCV_INCLUDE_DIRS)
  SET(OpenCV_INCLUDE_DIRS ${OpenCV_INCLUDE_DIR})
ENDIF (NOT OpenCV_INCLUDE_DIRS)

# END SPECIAL OPENCV HANDLING

INCLUDE_DIRECTORIES(${OpenCV_INCLUDE_DIRS})
IF (LIBFACE_FOUND)
  INCLUDE_DIRECTORIES(${LIBFACE_INCLUDE_DIR})
endif (LIBFACE_FOUND)

# =======================================================
# Information to update before to release this library.

# Library version history:
# API      ABI
# 0.1.0 => 0.1.0     (During digiKam GoSC2010)
# 2.0.0 => 1.0.0     (Included into digiKam 2.0.0 Software Collection)

# Library API version
SET(KFACE_LIB_MAJOR_VERSION "2")
SET(KFACE_LIB_MINOR_VERSION "0")
SET(KFACE_LIB_PATCH_VERSION "0")

# Suffix to add at end of version string. Usual values are:
# "-git"   : alpha code unstable from git. Do not use in production
# "-beta1" : beta1 release.
# "-beta2" : beta2 release.
# "-beta3" : beta3 release.
# "-rc"    : release candidate.
# ""       : final relase. Can be used in production.
SET(KFACE_LIB_SUFFIX_VERSION "")

# Library ABI version used by linker.
# For details : http://www.gnu.org/software/libtool/manual/libtool.html#Updating-version-info
SET(KFACE_LIB_SO_CUR_VERSION "1")
SET(KFACE_LIB_SO_REV_VERSION "0")
SET(KFACE_LIB_SO_AGE_VERSION "0")

# =======================================================
# Set env. variables accordinly.

SET(KFACE_LIB_VERSION_STRING "${KFACE_LIB_MAJOR_VERSION}.${KFACE_LIB_MINOR_VERSION}.${KFACE_LIB_PATCH_VERSION}${KFACE_LIB_SUFFIX_VERSION}")
SET(KFACE_LIB_VERSION_ID "0x0${KFACE_LIB_MAJOR_VERSION}0${KFACE_LIB_MINOR_VERSION}0${KFACE_LIB_PATCH_VERSION}")
SET(KFACE_LIB_SO_VERSION_STRING "${KFACE_LIB_SO_CUR_VERSION}.${KFACE_LIB_SO_REV_VERSION}.${KFACE_LIB_SO_AGE_VERSION}")

# =======================================================

IF (NOT LIBFACE_FOUND)
    INCLUDE_DIRECTORIES( ${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_CURRENT_BINARY_DIR} ${CMAKE_CURRENT_BINARY_DIR}/libface )

    SET(face_LIB_SRCS ${CMAKE_CURRENT_SOURCE_DIR}/LibFaceUtils.cpp
                      ${CMAKE_CURRENT_SOURCE_DIR}/FaceDetect.cpp
                      ${CMAKE_CURRENT_SOURCE_DIR}/Face.cpp
                      ${CMAKE_CURRENT_SOURCE_DIR}/Eigenfaces.cpp
                      ${CMAKE_CURRENT_SOURCE_DIR}/LibFace.cpp
                      ${CMAKE_CURRENT_SOURCE_DIR}/Haarcascades.cpp
       )

    SET(LIBFACE_LIBRARIES "")

    # NOTE: update libface version ID when code is updated from SF.net
    SET(libface_MAJOR_VERSION 0)
    SET(libface_MINOR_VERSION 2)

    SET(DEBUG_MODE 0)
    SET(LOG_LEVEL "libfaceINFO")
    SET(LIB_TYPE STATIC)
    CONFIGURE_FILE("${CMAKE_CURRENT_SOURCE_DIR}/LibFaceConfig.h.cmake"
                   "${CMAKE_CURRENT_BINARY_DIR}/LibFaceConfig.h")

    MESSAGE(STATUS "External libface was not found, use internal version instead...")
ENDIF (NOT LIBFACE_FOUND)



# Install find cmake script to the system for client applications.

ADD_DEFINITIONS(${KDE4_ENABLE_EXCEPTIONS})

ADD_DEFINITIONS(${LIBFACE_DEFINITIONS})
ADD_DEFINITIONS(${KDE4_ENABLE_EXCEPTIONS})
REMOVE_DEFINITIONS(-DQT_NO_STL)

CONFIGURE_FILE(${CMAKE_CURRENT_SOURCE_DIR}/version.h.cmake ${CMAKE_CURRENT_BINARY_DIR}/version.h)

SET(kface_LIB_SRCS database.cpp
                   face.cpp
                   kfaceutils.cpp
                   image.cpp
                   facedetector.cpp
                   recognitiondatabase.cpp
   )

KDE4_ADD_LIBRARY(kface SHARED ${kface_LIB_SRCS} ${face_LIB_SRCS})

TARGET_LINK_LIBRARIES(kface ${KDE4_KDECORE_LIBS}
                            ${KDE4_KDEUI_LIBS}
                            ${QT_QTGUI_LIBRARY}
                            ${LIBFACE_LIBRARIES}
                            ${OpenCV_LIBRARIES}
                            )

SET_TARGET_PROPERTIES(kface PROPERTIES VERSION ${KFACE_LIB_SO_VERSION_STRING}
                            SOVERSION ${KFACE_LIB_SO_CUR_VERSION})

INSTALL(TARGETS kface ${INSTALL_TARGETS_DEFAULT_ARGS})

INSTALL(FILES database.h
              face.h
              image.h
              facedetector.h
              recognitiondatabase.h
              libkface_export.h
              ${CMAKE_CURRENT_BINARY_DIR}/version.h
              DESTINATION ${INCLUDE_INSTALL_DIR}/libkface COMPONENT Devel
       )


SET(detect_SRCS detect.cpp)
KDE4_ADD_EXECUTABLE(detect ${detect_SRCS})
TARGET_LINK_LIBRARIES(detect kface ${KDE4_KDECORE_LIBS} ${QT_QTCORE_LIBRARY} ${QT_QTGUI_LIBRARY})

# -----------------------------------------------------------------------------

SET(libfaceGUI_SRCS main.cpp
                      mainwindow.cpp
                      faceitem.cpp
                      button.cpp
                      marquee.cpp
                      fancyrect.cpp
#opentld
                      Config.cpp
                      Gui.cpp
                      Main.cpp
                      Settings.cpp
                      Config.h
                      Gui.h
                      Main.h
                      Settings.h
                    )

KDE4_ADD_UI_FILES(libfaceGUI_SRCS ${CMAKE_CURRENT_SOURCE_DIR}/mainwindow.ui)
KDE4_ADD_EXECUTABLE(libfaceGUI ${libfaceGUI_SRCS})

TARGET_LINK_LIBRARIES(libfaceGUI kface ${KDE4_KDECORE_LIBS}
                                       ${KDE4_KDEUI_LIBS}
                                       ${KDE4_KIO_LIBS}
                                       ${QT_QTCORE_LIBRARY}
                                       ${QT_QTGUI_LIBRARY}
                                       ${OpenCV_LIBRARIES} 
#opentld
                                       libopentld cvblobs libconfig ${OpenCV_LIBS}
                     )

