file(GLOB jsonrpc_source *.cpp json/*.c*)
file(GLOB jsonrpc_header *.h)

set(JSONRPC_DEPENDENCIES)

if(HTTP_CONNECTOR)
	file(GLOB connector_mongoose_source connectors/mongoose*.c*)
	file(GLOB connector_mongoose_header connectors/mongoose*.h)
	file(GLOB connector_http_source connectors/http*.c*)
	file(GLOB connector_http_header connectors/http*.h)
	set(JSONRPC_DEPENDENCIES ${JSONRPC_DEPENDENCIES} ${CURL_LIBRARIES} pthread dl)
endif()

if(ZMQ_CONNECTOR)
	file(GLOB connector_cppzmq_header connectors/zmq.hpp)
	file(GLOB connector_zmq_source connectors/zmq*.c*)
	file(GLOB connector_zmq_header connectors/zmq*.h)
	set(JSONRPC_DEPENDENCIES ${JSONRPC_DEPENDENCIES} ${ZMQ_LIBRARIES})
endif()

file(GLOB jsoncpp_header json/*.h)

set(connectors_source ${connector_http_source} ${connector_mongoose_source} ${connector_zmq_source})
set(connectors_header ${connector_http_header} ${connector_mongoose_header} ${connector_cppzmq_header} ${connector_zmq_header})


add_library(jsonrpc SHARED ${jsonrpc_source} ${connectors_source})
add_library(jsonrpcStatic STATIC ${jsonrpc_source} ${connectors_source})

set_target_properties(jsonrpcStatic PROPERTIES OUTPUT_NAME jsonrpc)

set(VERSION_STRING ${MAJOR_VERSION}.${MINOR_VERSION}.${PATCH_VERSION}) 
set_target_properties(jsonrpc jsonrpcStatic PROPERTIES VERSION "${VERSION_STRING}" SOVERSION "${VERSION_MAJOR}")

target_link_libraries(jsonrpc ${JSONRPC_DEPENDENCIES})
target_link_libraries(jsonrpcStatic ${JSONRPC_DEPENDENCIES})



install(FILES ${jsonrpc_header} DESTINATION include/jsonrpc)
install(FILES ${jsoncpp_header} DESTINATION include/jsonrpc/json)
if(HTTP_CONNECTOR or ZMQ_CONNECTOR)
	install(FILES ${connectors_header} DESTINATION include/jsonrpc/connectors)
endif()

install(TARGETS jsonrpc jsonrpcStatic LIBRARY DESTINATION lib
                      ARCHIVE DESTINATION lib
                      RUNTIME DESTINATION bin)

