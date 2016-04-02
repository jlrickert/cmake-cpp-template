file(GLOB_RECURSE SRCS *.cpp *.cxx *.cc *.C *.c *.h *.hpp)
add_library(main ${SRCS})

add_executable(${PROJECT} ${SRCS})

target_link_libraries (${PROJECT} ${libcore} ${CMAKE_THREAD_LIBS_INIT})

install (TARGETS ${PRJECT} DESTINATION bin)
