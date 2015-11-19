cmake_minimum_required(VERSION 2.8)
project(qtservice)

option(${PROJECT_NAME}_BUILD_SHARED "Build ${PROJECT_NAME} as shared libraries" true)

find_package(Qt5Core REQUIRED)

add_definitions(-DUNICODE)

if(NOT WIN32)
  find_package(Qt5Network REQUIRED)
endif()

if(MSVC AND BUILD_SHARED_LIBS)
  add_definitions("/DQT_QTSERVICE_EXPORT=__declspec(dllexport)")
endif()

set(${PROJECT_NAME}-sources qtservice.cpp)

if (WIN32)
  set(${PROJECT_NAME}-sources ${${PROJECT_NAME}-sources} qtservice_win.cpp)
elseif(UNIX)
  set(${PROJECT_NAME}-sources ${${PROJECT_NAME}-sources} qtservice_unix.cpp qtunixsocket.cpp qtunixserversocket.cpp)
endif()

# for #include of generated moc files
include_directories(${CMAKE_CURRENT_BINARY_DIR})
    
# for #include of moc headers from binary tree
include_directories(${CMAKE_CURRENT_SOURCE_DIR})

add_library(${PROJECT_NAME} ${${PROJECT_NAME}-sources} ${${PROJECT_NAME}-mocs})
target_link_libraries(${PROJECT_NAME} Qt5::Core)
if(NOT WIN32)
  target_link_libraries(${PROJECT_NAME} Qt5::Network)
endif()


include_directories(${Qt5Network_INCLUDE_DIRS})

install(TARGETS ${PROJECT_NAME}
    EXPORT flow-LibraryDependencies
		RUNTIME DESTINATION bin
		LIBRARY DESTINATION lib
		ARCHIVE DESTINATION lib
		)

configure_file( ${CMAKE_CURRENT_SOURCE_DIR}/ProjectConfig.cmake.in
	${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}-config.cmake @ONLY )
install(FILES ${PROJECT_BINARY_DIR}/${PROJECT_NAME}-config.cmake
	DESTINATION share/cmake/${PROJECT_NAME} COMPONENT Development )

install(EXPORT flow-LibraryDependencies DESTINATION share/cmake/${PROJECT_NAME} COMPONENT Development)

set(QTSERVICE_INCLUDE_DIRS
  ${CMAKE_CURRENT_SOURCE_DIR}
  CACHE INTERNAL "${PROJECT_NAME} include directories"
)

set(QTSERVICE_LIBRARIES
  ${PROJECT_NAME}
  CACHE INTERNAL "${PROJECT_NAME} libraries"
)
