Cmake_Minimum_Required(VERSION 2.6.0 FATAL_ERROR)

Project(Pythia6)
Enable_Language(Fortran)

set(Pythia_FSRCS
pythia-6.4.28.f
tpythia6_called_from_cc.F
)


# The order here is important because gfortran is contained in pgfortran
If (CMAKE_Fortran_COMPILER MATCHES "gfortran")
#  set(UnderScoreString "-fno-second_underscore")
EndIf (CMAKE_Fortran_COMPILER MATCHES "gfortran")
If (CMAKE_Fortran_COMPILER MATCHES "pgfortran")
  set(UnderScoreString "-Mnosecond_underscore")
EndIf (CMAKE_Fortran_COMPILER MATCHES "pgfortran")


#Set_Source_Files_Properties(${Pythia_FSRCS} PROPERTIES COMPILE_FLAGS
#                           ${UnderScoreString})

# create c source
Execute_Process(COMMAND echo void MAIN() {} OUTPUT_FILE main.c)


set(Pythia_CSRCS
pythia6_common_address.c
main.c
)

Add_Library(pythia6 SHARED ${Pythia_FSRCS} ${Pythia_CSRCS})

# Check if gfortran library has to specified explicit
# This is needed so far on Mac OSX and gcc6
If(CMAKE_SYSTEM_NAME MATCHES Darwin)
  SET(CMAKE_SHARED_LIBRARY_CREATE_Fortran_FLAGS "${CMAKE_SHARED_LIBRARY_CREATE_Fortran_FLAGS} -flat_namespace -single_module -undefined dynamic_lookup")

  # Use the Fortran compiler CMake found rather than whatever "gfortran" is on
  # PATH (toolchains such as conda-forge's name it with a target prefix), and
  # compare the whole version: gfortran 10 and later have two-digit majors.
  If(CMAKE_Fortran_COMPILER_ID STREQUAL "GNU"
     AND NOT CMAKE_Fortran_COMPILER_VERSION VERSION_LESS 6)
    Execute_Process(COMMAND ${CMAKE_Fortran_COMPILER} -print-file-name=libgfortran.dylib
                    OUTPUT_VARIABLE _gfortran_path
                    OUTPUT_STRIP_TRAILING_WHITESPACE
                   )
    Get_Filename_COMPONENT(_gfortran_libdir ${_gfortran_path} DIRECTORY)
    Message("${_gfortran_libdir}")
    Target_Link_Libraries(pythia6 gfortran)
    Link_Directories(${_gfortran_libdir})
  EndIf()
EndIf()


Install(TARGETS pythia6 DESTINATION LIBRARY DESTINATION lib)

