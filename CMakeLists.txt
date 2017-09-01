cmake_minimum_required(VERSION 3.2)
project( PRISM )

#================================
# Find VTK
#================================
find_package(VTK 6.3.0 EXACT COMPONENTS vtkRenderingOpenGL vtkRenderingVolumeOpenGL vtkRenderingImage NO_MODULE )
include(${VTK_USE_FILE})

#================================
# Create custom commands to
# encode each glsl file into a
# C string literal in a header
# file.
#================================
set( LibGlsl vtkPRISMVolumeRaycast_FS )
set( VTK_ENCODESTRING_EXE ${VTK_INSTALL_PREFIX}/bin/vtkEncodeString-${VTK_MAJOR_VERSION}.${VTK_MINOR_VERSION} )
foreach( file ${LibGlsl} )
  set( shader_src ${CMAKE_CURRENT_SOURCE_DIR}/${file}.glsl )
  set( shader_cxx ${CMAKE_CURRENT_BINARY_DIR}/${file}.cxx )
  set( shader_h ${CMAKE_CURRENT_BINARY_DIR}/${file}.h )
  add_custom_command(
    OUTPUT ${shader_cxx} ${shader_h}
    DEPENDS ${shader_src}
    COMMAND ${VTK_ENCODESTRING_EXE}
    ARGS ${shader_cxx} ${shader_src} ${file} --build-header " " vtkSystemIncludes.h )
  set( LibGlslSrc ${LibGlslSrc} ${shader_src} ${shader_h} ${shader_cxx} )
endforeach(file)

include_directories( ${CMAKE_CURRENT_BINARY_DIR} )

#================================
# define sources
#================================
set( ${PROJECT_NAME}_SRCS vtkPRISMVolumeMapper.cxx
	vtkPRISMGlslShader.cxx
	vtkPRISMDrawableTexture.cxx
	vtkColoredCube.cxx
	vtkColorPolyData.cxx
	${LibGlslSrc} )

set( ${PROJECT_NAME}_HDRS vtkPRISMVolumeMapper.h
	vtkPRISMGlslShader.h
	vtkPRISMDrawableTexture.h
	vtkColoredCube.h
	vtkColorPolyData.h )

set( ${PROJECT_NAME}_TARGET_LIBRARIES ${VTK_LIBRARIES} )

#================================
# Add library
#================================
add_library(${PROJECT_NAME} ${${PROJECT_NAME}_SRCS} ${${PROJECT_NAME}_HDRS})

#================================
# Link lib
#================================
include(GenerateExportHeader)
set( ${PROJECT_NAME}_EXPORT_DIRECTIVE “${PROJECT_NAME}_EXPORT” )
generate_export_header(${PROJECT_NAME} EXPORT_FILE_NAME ${PROJECT_NAME}Export.h EXPORT_MACRO_NAME ${${PROJECT_NAME}_EXPORT_DIRECTIVE})
target_link_libraries(${PROJECT_NAME} PUBLIC ${${PROJECT_NAME}_TARGET_LIBRARIES})

#================================
# Include lib
#================================
set(${PROJECT_NAME}_INCLUDE_DIRECTORIES PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>
  $<INSTALL_INTERFACE:include> )
target_include_directories(${PROJECT_NAME} ${${PROJECT_NAME}_INCLUDE_DIRECTORIES})

#================================
# Install
#================================
INSTALL(TARGETS ${PROJECT_NAME} EXPORT ${PROJECT_NAME} 
	RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib 
	INCLUDES DESTINATION include )
INSTALL(FILES ${${PROJECT_NAME}_HDRS} "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Export.h" DESTINATION "${PROJECT_NAME}_INCLUDE_INSTALL" COMPONENT Includes)

#================================
# Examples
#================================
option( BUILD_EXAMPLES "Build PRISM examples" OFF )
if( ${BUILD_EXAMPLES} )
  add_subdirectory(Examples)
endif()
