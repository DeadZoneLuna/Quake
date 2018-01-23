#
# gas2masm: GNU Assembly to Microsoft Macro Assembler converter
#

include( MSVCRuntime )

set( PREP_SRCS
	gas2masm.cpp
)

add_executable( gas2masm ${PREP_SRCS} )

configure_msvc_runtime( gas2masm STATIC )

target_compile_definitions( gas2masm PRIVATE
	$<$<CXX_COMPILER_ID:MSVC>:_CRT_SECURE_NO_WARNINGS>
	$<$<CXX_COMPILER_ID:MSVC>:$<$<CONFIG:Debug>:_DEBUG>>
	$<$<CXX_COMPILER_ID:MSVC>:_CONSOLE>
)

if( MSVC )
	set_target_properties( gas2masm PROPERTIES
		COMPILE_FLAGS "/W3"
	)
endif()
