#
# gas2masm: GNU Assembly to Microsoft Macro Assembler converter
#

include( MSVCRuntime )

set( PREP_SRCS
	gas2masm.c
)

add_executable( gas2masm ${PREP_SRCS} )

configure_msvc_runtime( gas2masm STATIC )

if( MSVC )
	target_compile_definitions( gas2masm PRIVATE
		_CRT_SECURE_NO_WARNINGS
	)
endif()

if( MSVC )
	set_target_properties( gas2masm PROPERTIES
		COMPILE_FLAGS "/W3"
	)
endif()
