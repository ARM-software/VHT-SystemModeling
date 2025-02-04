set(CMAKE_BUILD_TYPE Release)

cmake_minimum_required(VERSION 3.0)

project(VHT)

option(TESTFEATURES "Enable tests" NO)

function(testapp project flag)
  add_executable(${project} main.c)
  target_include_directories(${project} PUBLIC ARM/Resources/Include)
  if (WIN32)
  target_link_libraries(${project} PRIVATE libVHT)
  target_link_libraries(${project} PRIVATE libWAVE)
  target_link_libraries(${project} PRIVATE libFIR)
  else()
  target_link_libraries(${project} PRIVATE VHT)
  target_link_libraries(${project} PRIVATE WAVE)
  target_link_libraries(${project} PRIVATE FIR)
  endif()
  target_compile_definitions(${project} PRIVATE ${flag})
  if (WIN32)
  target_compile_definitions(${project} PRIVATE WINTEST)
  endif()
endfunction()

function(mklib project src)
  add_library(${project} STATIC ${src})
  if (NOT (TESTFEATURES))
    if (WIN32)
      set_property(TARGET ${project} PROPERTY ARCHIVE_OUTPUT_DIRECTORY  ${PROJECT_SOURCE_DIR}/ARM/Resources/Library/win64)
    else() 
      set_property(TARGET ${project} PROPERTY ARCHIVE_OUTPUT_DIRECTORY  ${PROJECT_SOURCE_DIR}/ARM/Resources/Library/linux64)
    endif() 
  endif()

  if (NOT (WIN32))
    target_link_libraries(${project} PRIVATE m)
  endif()
  target_include_directories(${project} PUBLIC ARM/Resources/Include)
  
  if (NOT (TESTFEATURES))
     if (WIN32)
       target_include_directories(${project} PRIVATE "C:\\Program Files\\OpenModelica1.18.0-64bit\\include\\omc\\c")
     else()
       target_include_directories(${project} PRIVATE "/usr/include/omc/c")
     endif()
  endif()

  if (WIN32)
       target_compile_definitions(${project} PRIVATE  _CRT_SECURE_NO_WARNINGS)
       target_compile_options(${project} PRIVATE  "/GS-")
  endif()

  if (TESTFEATURES)
    target_compile_definitions(${project} PRIVATE  TEST)
  endif()

endfunction()


if (WIN32)
SET(CMAKE_STATIC_LINKER_FLAGS "${CMAKE_STATIC_LINKER_FLAGS}  /NODEFAULTLIB:library" )
endif() 

if (WIN32)
mklib(libVHT model.c)
mklib(libWAVE wave.c)
mklib(libFIR fir.c)
else()
mklib(VHT model.c)
mklib(WAVE wave.c)
mklib(FIR fir.c)
endif()

if (TESTFEATURES)
testapp(WaveTest WAVETEST)
testapp(FirTest FIRTEST)
testapp(ServerTest SERVERTEST)
endif()
