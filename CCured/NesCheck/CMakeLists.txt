add_llvm_library(NesCheck MODULE
  NesCheck.cpp
  AnalysisState.cpp
  AnalysisState.hpp
  PTAWrapper.cpp
  PTAWrapper.hh
  LLVMEssentials.hh
  
  DEPENDS
  intrinsics_gen
  PLUGIN_TOOL
  opt
  )

set_target_properties(NesCheck PROPERTIES
  COMPILE_FLAGS "-fno-rtti"
)

include_directories("/home/tianrou/Desktop/SVF/include")

target_link_libraries(NesCheck PUBLIC "/home/tianrou/Desktop/SVF/Release-build/lib/libSvf.a")

