file(GLOB_RECURSE TEST_SRCS *.cpp *.cxx *.cc *.C *.c *.h *.hpp)
set(TEST_LIBS main ${GTEST_BOTH_LIBRARIES} ${GMOCK_LIBRARIES})
set(TEST_BIN ${PROJECT}-test)
add_executable(${TEST_BIN} ${TEST_SRCS})

target_link_libraries(${TEST_BIN} ${TEST_LIBS})

install(TARGETS ${TEST_BIN} DESTINATION bin)
