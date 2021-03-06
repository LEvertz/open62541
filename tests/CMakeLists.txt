include_directories(${PROJECT_SOURCE_DIR}/include)
include_directories(${PROJECT_SOURCE_DIR}/deps)
include_directories(${PROJECT_SOURCE_DIR}/src)
include_directories(${PROJECT_SOURCE_DIR}/plugins)
include_directories(${PROJECT_BINARY_DIR}/src_generated)

find_package(Check REQUIRED)
find_package(Threads REQUIRED)

set(LIBS ${CHECK_LIBRARIES} ${open62541_LIBRARIES})
if(NOT WIN32)
  list(APPEND LIBS pthread m)
  if (NOT APPLE)
    list(APPEND LIBS rt subunit)
  endif()
else()
    list(APPEND LIBS ws2_32)
endif()
if(UA_ENABLE_MULTITHREADING)
    list(APPEND LIBS urcu-cds urcu urcu-common)
endif()

add_definitions(-Wno-sign-conversion)

# the unit test are built directly on the open62541 object files. so they can
# access symbols that are hidden/not exported to the shared library

add_executable(check_builtin check_builtin.c $<TARGET_OBJECTS:open62541-object>)
target_link_libraries(check_builtin ${LIBS})
add_test(builtin ${CMAKE_CURRENT_BINARY_DIR}/check_builtin)

add_executable(check_memory check_memory.c $<TARGET_OBJECTS:open62541-object>)
target_link_libraries(check_memory ${LIBS})
add_test(memory ${CMAKE_CURRENT_BINARY_DIR}/check_memory)

add_executable(check_chunking check_chunking.c $<TARGET_OBJECTS:open62541-object>)
target_link_libraries(check_chunking ${LIBS})
add_test(chunking ${CMAKE_CURRENT_BINARY_DIR}/check_chunking)

# add_executable(check_stack check_stack.c)
# target_link_libraries(check_stack ${LIBS})
# add_test(stack ${CMAKE_CURRENT_BINARY_DIR}/check_stack)

add_executable(check_services_view check_services_view.c $<TARGET_OBJECTS:open62541-object>)
target_link_libraries(check_services_view ${LIBS})
add_test(services_view ${CMAKE_CURRENT_BINARY_DIR}/check_services_view)

add_executable(check_services_attributes check_services_attributes.c $<TARGET_OBJECTS:open62541-object>)
target_link_libraries(check_services_attributes ${LIBS})
add_test(services_attributes ${CMAKE_CURRENT_BINARY_DIR}/check_services_attributes)

add_executable(check_services_nodemanagement check_services_nodemanagement.c $<TARGET_OBJECTS:open62541-object>)
target_link_libraries(check_services_nodemanagement ${LIBS})
add_test(services_nodemanagement ${CMAKE_CURRENT_BINARY_DIR}/check_services_nodemanagement)

add_executable(check_nodestore check_nodestore.c $<TARGET_OBJECTS:open62541-object>)
target_link_libraries(check_nodestore ${LIBS})
add_test(nodestore ${CMAKE_CURRENT_BINARY_DIR}/check_nodestore)

add_executable(check_session check_session.c $<TARGET_OBJECTS:open62541-object>)
target_link_libraries(check_session ${LIBS})
add_test(session ${CMAKE_CURRENT_BINARY_DIR}/check_session)

add_executable(check_server_userspace check_server_userspace.c $<TARGET_OBJECTS:open62541-object>)
target_link_libraries(check_server_userspace ${LIBS})
add_test(check_server_userspace ${CMAKE_CURRENT_BINARY_DIR}/check_server_userspace)

# add_executable(check_startup check_startup.c)
# target_link_libraries(check_startup ${LIBS})
# add_test(startup ${CMAKE_CURRENT_BINARY_DIR}/check_startup)

# test with canned interactions from files

add_custom_command(OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/client_HELOPN.bin
                          ${CMAKE_CURRENT_BINARY_DIR}/client_CLO.bin
                          ${CMAKE_CURRENT_BINARY_DIR}/client_CreateActivateSession.bin
                          ${CMAKE_CURRENT_BINARY_DIR}/client_Browse.bin
                          ${CMAKE_CURRENT_BINARY_DIR}/client_Read.bin
                          ${CMAKE_CURRENT_BINARY_DIR}/client_Write.bin
                  PRE_BUILD
                  COMMAND python ${PROJECT_SOURCE_DIR}/tools/hex2bin.py
                                 ${CMAKE_CURRENT_SOURCE_DIR}/dumps/client_HELOPN.hex
                                 ${CMAKE_CURRENT_SOURCE_DIR}/dumps/client_CLO.hex
                                 ${CMAKE_CURRENT_SOURCE_DIR}/dumps/client_CreateActivateSession.hex
                                 ${CMAKE_CURRENT_SOURCE_DIR}/dumps/client_Browse.hex
                                 ${CMAKE_CURRENT_SOURCE_DIR}/dumps/client_Read.hex
                                 ${CMAKE_CURRENT_SOURCE_DIR}/dumps/client_Write.hex
                  DEPENDS ${PROJECT_SOURCE_DIR}/tools/hex2bin.py
                          ${CMAKE_CURRENT_SOURCE_DIR}/dumps/client_HELOPN.hex
                          ${CMAKE_CURRENT_SOURCE_DIR}/dumps/client_CLO.hex
                          ${CMAKE_CURRENT_SOURCE_DIR}/dumps/client_CreateActivateSession.hex
                          ${CMAKE_CURRENT_SOURCE_DIR}/dumps/client_Browse.hex
                          ${CMAKE_CURRENT_SOURCE_DIR}/dumps/client_Read.hex
                          ${CMAKE_CURRENT_SOURCE_DIR}/dumps/client_Write.hex)
add_custom_target(client_HELOPN.bin DEPENDS "${CMAKE_CURRENT_BINARY_DIR}/client_HELOPN.bin")
add_custom_target(client_CreateActivateSession.bin DEPENDS "${CMAKE_CURRENT_BINARY_DIR}/client_CreateActivateSession.bin")

add_executable(check_server_binary_messages check_server_binary_messages.c testing_networklayers.c $<TARGET_OBJECTS:open62541-object>)
target_include_directories(check_server_binary_messages PRIVATE ${PROJECT_SOURCE_DIR}/src/server)
target_link_libraries(check_server_binary_messages ${LIBS})
add_dependencies(check_server_binary_messages client_HELOPN.bin)

add_test(check_server_binary_messages_browse ${CMAKE_CURRENT_BINARY_DIR}/check_server_binary_messages
                                             ${CMAKE_CURRENT_BINARY_DIR}/client_HELOPN.bin
                                             ${CMAKE_CURRENT_BINARY_DIR}/client_CreateActivateSession.bin
                                             ${CMAKE_CURRENT_BINARY_DIR}/client_Browse.bin
                                             ${CMAKE_CURRENT_BINARY_DIR}/client_CLO.bin)

add_test(check_server_binary_messages_read ${CMAKE_CURRENT_BINARY_DIR}/check_server_binary_messages
                                           ${CMAKE_CURRENT_BINARY_DIR}/client_HELOPN.bin
                                           ${CMAKE_CURRENT_BINARY_DIR}/client_CreateActivateSession.bin
                                           ${CMAKE_CURRENT_BINARY_DIR}/client_Read.bin
                                           ${CMAKE_CURRENT_BINARY_DIR}/client_CLO.bin)

add_test(check_server_binary_messages_write ${CMAKE_CURRENT_BINARY_DIR}/check_server_binary_messages
                                           ${CMAKE_CURRENT_BINARY_DIR}/client_HELOPN.bin
                                           ${CMAKE_CURRENT_BINARY_DIR}/client_CreateActivateSession.bin
                                           ${CMAKE_CURRENT_BINARY_DIR}/client_Write.bin
                                           ${CMAKE_CURRENT_BINARY_DIR}/client_CLO.bin)
