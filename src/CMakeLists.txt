function(KV_protobuf_generate_cpp SRCS ROOT PROTO_PATH PROTO_NAME)
add_custom_command(
    OUTPUT "${CMAKE_CURRENT_BINARY_DIR}/${PROTO_PATH}/${PROTO_NAME}.pb.h"
        "${CMAKE_CURRENT_BINARY_DIR}/${PROTO_PATH}/${PROTO_NAME}.pb.cc"
    COMMAND ${CMAKE_COMMAND} -E make_directory ${CMAKE_CURRENT_BINARY_DIR}/${PROTO_PATH}
    COMMAND  ${PROTOBUF_PROTOC_EXECUTABLE}
    ARGS --cpp_out=${CMAKE_CURRENT_BINARY_DIR}/${PROTO_PATH}
        -I ${ROOT}/${PROTO_PATH}
        ${ROOT}/${PROTO_PATH}/${PROTO_NAME}.proto
    DEPENDS ${PROTOBUF_PROTOC_EXECUTABLE} ${ROOT}/${PROTO_PATH}/${PROTO_NAME}.proto
    COMMENT "Generating: ${ROOT}/${PROTO_PATH}/${PROTO_NAME}.proto"
    VERBATIM)
set(${SRCS} ${${SRCS}} "${CMAKE_CURRENT_BINARY_DIR}/${PROTO_PATH}/${PROTO_NAME}.pb.cc" PARENT_SCOPE)
endfunction()

function(KV_protobuf_generate_grpc_cpp SRCS ROOT PROTO_PATH PROTO_NAME)
add_custom_command(
    OUTPUT "${CMAKE_CURRENT_BINARY_DIR}/${PROTO_PATH}/${PROTO_NAME}.grpc.pb.h"
        "${CMAKE_CURRENT_BINARY_DIR}/${PROTO_PATH}/${PROTO_NAME}.grpc.pb.cc"
    COMMAND ${CMAKE_COMMAND} -E make_directory ${CMAKE_CURRENT_BINARY_DIR}/${PROTO_PATH}
    COMMAND  ${PROTOBUF_PROTOC_EXECUTABLE}
    ARGS --grpc_out=${CMAKE_CURRENT_BINARY_DIR}/${PROTO_PATH}
        --plugin=protoc-gen-grpc=$<TARGET_FILE:gRPC::grpc_cpp_plugin>
        -I ${ROOT}/${PROTO_PATH}
        ${ROOT}/${PROTO_PATH}/${PROTO_NAME}.proto
    DEPENDS ${PROTOBUF_PROTOC_EXECUTABLE} ${ROOT}/${PROTO_PATH}/${PROTO_NAME}.proto
    COMMENT "Generating: ${ROOT}/${PROTO_PATH}/${PROTO_NAME}.proto"
    VERBATIM)
set(${SRCS} ${${SRCS}} "${CMAKE_CURRENT_BINARY_DIR}/${PROTO_PATH}/${PROTO_NAME}.grpc.pb.cc" PARENT_SCOPE)
endfunction()

KV_protobuf_generate_cpp(_src_protos ${PROJECT_SOURCE_DIR}/ proto kv)
KV_protobuf_generate_cpp(_src_protos ${PROJECT_SOURCE_DIR}/ proto auth)
KV_protobuf_generate_cpp(_src_protos ${PROJECT_SOURCE_DIR}/ proto rpc)
KV_protobuf_generate_grpc_cpp(_src_protos ${PROJECT_SOURCE_DIR}/ proto rpc)

add_library(etcd-cpp-api STATIC
${_src_protos}
${PROJECT_SOURCE_DIR}/v3/src/AsyncTxnResponse.cpp
${PROJECT_SOURCE_DIR}/v3/src/AsyncRangeResponse.cpp
${PROJECT_SOURCE_DIR}/v3/src/Transaction.cpp
${PROJECT_SOURCE_DIR}/v3/src/action_constants.cpp
${PROJECT_SOURCE_DIR}/v3/src/AsyncSetAction.cpp
${PROJECT_SOURCE_DIR}/v3/src/AsyncCompareAndSwapAction.cpp
${PROJECT_SOURCE_DIR}/v3/src/AsyncUpdateAction.cpp
${PROJECT_SOURCE_DIR}/v3/src/AsyncGetAction.cpp
${PROJECT_SOURCE_DIR}/v3/src/AsyncDeleteAction.cpp
${PROJECT_SOURCE_DIR}/v3/src/AsyncCompareAndDeleteAction.cpp
${PROJECT_SOURCE_DIR}/v3/src/Action.cpp
${PROJECT_SOURCE_DIR}/v3/src/AsyncWatchAction.cpp
${PROJECT_SOURCE_DIR}/v3/src/V3Response.cpp
${PROJECT_SOURCE_DIR}/v3/src/AsyncDeleteRangeResponse.cpp
${PROJECT_SOURCE_DIR}/v3/src/AsyncWatchResponse.cpp
${PROJECT_SOURCE_DIR}/v3/src/AsyncLeaseGrantResponse.cpp
${PROJECT_SOURCE_DIR}/v3/src/AsyncLeaseGrantAction.cpp
${PROJECT_SOURCE_DIR}/v3/src/KeyValue.cpp
${CMAKE_CURRENT_LIST_DIR}/Client.cpp
${CMAKE_CURRENT_LIST_DIR}/Response.cpp
${CMAKE_CURRENT_LIST_DIR}/Value.cpp
${CMAKE_CURRENT_LIST_DIR}/SyncClient.cpp
${CMAKE_CURRENT_LIST_DIR}/Watcher.cpp)

include_directories(${CMAKE_CURRENT_BINARY_DIR})
target_link_libraries(etcd-cpp-api 
        cpprestsdk::cpprest
        Boost::system
        ssl 
        crypto 
        protobuf::libprotobuf 
        gRPC::grpc++)

install (TARGETS etcd-cpp-api EXPORT etcd-cpp-api DESTINATION lib)
install (FILES ${PROJECT_SOURCE_DIR}/etcd/Client.hpp
               ${PROJECT_SOURCE_DIR}/etcd/SyncClient.hpp
               ${PROJECT_SOURCE_DIR}/etcd/Response.hpp
               ${PROJECT_SOURCE_DIR}/etcd/Value.hpp
               ${PROJECT_SOURCE_DIR}/etcd/Watcher.hpp
               DESTINATION include/etcd)


install(DIRECTORY ${PROJECT_SOURCE_DIR}/v3/include/
        DESTINATION include/v3/include/
)

install(FILES 
              ${CMAKE_CURRENT_BINARY_DIR}/proto/kv.pb.h
              ${CMAKE_CURRENT_BINARY_DIR}/proto/auth.pb.h
              ${CMAKE_CURRENT_BINARY_DIR}/proto/rpc.pb.h
              ${CMAKE_CURRENT_BINARY_DIR}/proto/rpc.grpc.pb.h
        DESTINATION include/proto)
