if($CMAKE_SYSTEM_NAME} MATCHES "Linux")
  find_package(Threads QUIET)
  if(CMAKE_THREAD_LIBS_INIT)
  else()
    message("phtreads not found. Not building spatialaudio.")
    return()
  endif()
endif($CMAKE_SYSTEM_NAME} MATCHES "Linux")

include("../BuildExamples.cmake")

if($CMAKE_SYSTEM_NAME} MATCHES "Windows")

endif()

find_library(FFTW_LIBRARY
  NAMES fftw3f)

if (NOT FFTW_LIBRARY)
  message("fftw3 not found. Not building spatialaudio")
else()
  set(ZITACONVOLVER_DIRECTORY "${CMAKE_CURRENT_LIST_DIR}/zita-convolver-4.0.3/source") 
  message(STATUS "Using fftw3: ${FFTW_LIBRARY}")

  set(THIS_EXTENSION_SRC
  #  src/al_OutputMaster.cpp
  #  src/al_SoundfileBuffered.cpp
  #  src/al_AmbiFilePlayer.cpp
  #  src/al_AmbiTunedDecoder.cpp
  #  src/al_AmbisonicsConfig.cpp
    ${CMAKE_CURRENT_LIST_DIR}/src/al_Convolver.cpp
    ${CMAKE_CURRENT_LIST_DIR}/src/al_Decorrelation.cpp
  #  ${CMAKE_CURRENT_LIST_DIR}/src/butter.cpp
    ${ZITACONVOLVER_DIRECTORY}/zita-convolver.cc
    )

  set(THIS_EXTENSION_HEADERS
  #  alloaudio/al_OutputMaster.hpp
  #  alloaudio/al_SoundfileBuffered.hpp
  #  alloaudio/al_AmbiFilePlayer.hpp
  #  alloaudio/al_AmbiTunedDecoder.hpp
  #  alloaudio/al_AmbisonicsConfig.hpp
    ${CMAKE_CURRENT_LIST_DIR}/al_Convolver.hpp
    ${CMAKE_CURRENT_LIST_DIR}/al_Decorrelation.hpp
    ${ZITACONVOLVER_DIRECTORY}/zita-convolver.h
  )
  set(THIS_EXTENSION_LIBRARY_NAME al_spatialaudio)

  add_library(${THIS_EXTENSION_LIBRARY_NAME} ${AL_LIBRARY_TYPE} ${THIS_EXTENSION_SRC} ${THIS_EXTENSION_HEADERS})

  # c++14 It's very important to set this for the extension as not
  # setting it will cause targets upstream to not use c++14
  set_target_properties(al_spatialaudio PROPERTIES
    CXX_STANDARD 14
    CXX_STANDARD_REQUIRED ON
    )

  set(THIS_EXTENSION_LIBRARIES  ${FFTW_LIBRARY} ${GAMMA_LIBRARY})

  target_link_libraries(${THIS_EXTENSION_LIBRARY_NAME} al ${THIS_EXTENSION_LIBRARIES})
  target_include_directories(${THIS_EXTENSION_LIBRARY_NAME} PRIVATE ${ZITACONVOLVER_DIRECTORY})

  # Connections to outside
  set(CURRENT_EXTENSION_LIBRARIES ${THIS_EXTENSION_LIBRARY_NAME} ${THIS_EXTENSION_LIBRARIES})
  set(CURRENT_EXTENSION_INCLUDE_DIRS ${CMAKE_CURRENT_LIST_DIR})

  # unit tests

  if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/external/catch")
      add_executable(convolverTests ${CMAKE_CURRENT_LIST_DIR}/unitTests/utConvolver.cpp)
      target_link_libraries(convolverTests al ${SPATIALAUDIO_LINK_LIBRARIES} )
      target_include_directories(convolverTests PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/external/catch")
      add_test(NAME convolverTests
          COMMAND $<TARGET_FILE:convolverTests> ${TEST_ARGS})
      set_target_properties(convolverTests PROPERTIES
          CXX_STANDARD 14
          CXX_STANDARD_REQUIRED ON
          )

      #  add_memcheck_test(convolverTests)

      add_executable(decorrelationTests ${CMAKE_CURRENT_LIST_DIR}/unitTests/utDecorrelation.cpp)
      target_link_libraries(decorrelationTests al ${SPATIALAUDIO_LINK_LIBRARIES})
      target_include_directories(decorrelationTests PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/external/catch")
      add_test(NAME decorrelationTests
          COMMAND $<TARGET_FILE:decorrelationTests> ${TEST_ARGS})
      set_target_properties(decorrelationTests PROPERTIES
          CXX_STANDARD 14
          CXX_STANDARD_REQUIRED ON
          )
      #  add_memcheck_test(decorrelationTests)

  endif(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/external/catch")

  if(BUILD_EXAMPLES)
      BuildExamples("${CMAKE_CURRENT_LIST_DIR}/examples" ${THIS_EXTENSION_LIBRARY_NAME} "")
  endif(BUILD_EXAMPLES)

endif(NOT FFTW_LIBRARY)
