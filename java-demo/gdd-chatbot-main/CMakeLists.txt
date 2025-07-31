cmake_minimum_required(VERSION 3.22)

project(chatbot-arm64)

set(LLAMA_CURL ON)
set(LLAMA_SVE ON)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

add_subdirectory(llama.cpp)

add_library(chatbot-arm64 SHARED chatbot.cpp)
target_link_libraries(chatbot-arm64 PRIVATE llama common ${CMAKE_THREAD_LIBS_INIT})
target_compile_features(chatbot-arm64 PRIVATE cxx_std_11)
