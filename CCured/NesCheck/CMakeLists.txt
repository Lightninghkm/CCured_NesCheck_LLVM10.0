add_library(NesCheck MODULE
  NesCheck.cpp
  AnalysisState.cpp
  AnalysisState.hpp

  )

set_target_properties(NesCheck PROPERTIES
  COMPILE_FLAGS "-fno-rtti"
)
