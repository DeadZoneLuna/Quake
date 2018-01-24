#
# WinQuake
#

# Source additional modules from the "cmake" directory
list( APPEND CMAKE_MODULE_PATH
	"${CMAKE_CURRENT_LIST_DIR}/../cmake"
)

include( MSVCRuntime )

cmake_minimum_required( VERSION 3.10 )

project( WinQuake LANGUAGES C CXX )

find_package( OpenGL REQUIRED )

add_subdirectory( gas2masm )

set( WINQUAKE_RENDERER "OpenGL" CACHE STRING "Renderer to use" )
set_property( CACHE WINQUAKE_RENDERER PROPERTY STRINGS Software OpenGL )

if( ${WINQUAKE_RENDERER} STREQUAL "Software" )
	set( RENDERER_IS_SOFTWARE ON )
elseif( ${WINQUAKE_RENDERER} STREQUAL "OpenGL" )
	set( RENDERER_IS_OPENGL ON )
else()
	message( FATAL_ERROR "Unknown renderer ${WINQUAKE_RENDERER} selected" )
endif()

set( PREP_SRCS
	anorm_dots.h
	anorms.h
	bspfile.h
	cdaudio.h
	chase.cpp
	cl_demo.cpp
	cl_input.cpp
	cl_main.cpp
	cl_parse.cpp
	cl_tent.cpp
	client.h
	cmd.cpp
	cmd.h
	common.cpp
	common.h
	conproc.cpp
	conproc.h
	console.cpp
	console.h
	crc.cpp
	crc.h
	cvar.cpp
	cvar.h
	dosisms.h
	draw.h
	host.cpp
	host_cmd.cpp
	in_win.cpp
	input.h
	keys.cpp
	keys.h
	math.s
	mathlib.cpp
	mathlib.h
	menu.cpp
	menu.h
	model.h
	modelgen.h
	net.h
	net_dgrm.cpp
	net_dgrm.h
	net_loop.cpp
	net_loop.h
	net_main.cpp
	net_ser.h
	net_vcr.cpp
	net_vcr.h
	net_win.cpp
	net_wins.cpp
	net_wins.h
	net_wipx.cpp
	net_wipx.h
	pr_cmds.cpp
	pr_comp.h
	pr_edict.cpp
	pr_exec.cpp
	progdefs.h
	progs.h
	protocol.h
	quakeasm.h
	quakedef.h
	r_part.cpp
	render.h
	sbar.cpp
	sbar.h
	screen.h
	server.h
	snd_dma.cpp
	snd_mem.cpp
	snd_mix.cpp
	snd_mixa.s
	sound.h
	spritegn.h
	sv_main.cpp
	sv_move.cpp
	sv_phys.cpp
	sv_user.cpp
	sys.h
	vid.h
	view.cpp
	view.h
	wad.cpp
	wad.h
	winquake.h
	world.cpp
	world.h
	worlda.s
	zone.cpp
	zone.h
)

if( MSVC )
	set( PREP_SRCS
		${PREP_SRCS}
		cd_win.cpp
		snd_win.cpp
		sys_win.cpp
		sys_wina.s
	)
endif()

if( UNIX )
	set( PREP_SRCS
		${PREP_SRCS}
		cd_linux.cpp
		snd_linux.cpp
		sys_dosa.s
		sys_linux.cpp
	)
endif()

if( RENDERER_IS_SOFTWARE )
	set( PREP_SRCS
		${PREP_SRCS}
		d_draw.s
		d_draw16.s
		d_edge.cpp
		d_fill.cpp
		d_iface.h
		d_init.cpp
		d_modech.cpp
		d_part.cpp
		d_parta.s
		d_polysa.s
		d_polyse.cpp
		d_scan.cpp
		d_scana.s
		d_sky.cpp
		d_spr8.s
		d_sprite.cpp
		d_surf.cpp
		d_vars.cpp
		d_varsa.s
		d_zpoint.cpp
		draw.cpp
		model.cpp
		r_aclip.cpp
		r_aclipa.s
		r_alias.cpp
		r_aliasa.s
		r_bsp.cpp
		r_draw.cpp
		r_drawa.s
		r_edge.cpp
		r_edgea.s
		r_efrag.cpp
		r_light.cpp
		r_local.h
		r_main.cpp
		r_misc.cpp
		r_shared.h
		r_sky.cpp
		r_sprite.cpp
		r_surf.cpp
		r_vars.cpp
		r_varsa.s
		screen.cpp
		surf8.s
		surf16.s
		vid_win.cpp
	)
elseif( RENDERER_IS_OPENGL )
	set( PREP_SRCS
		${PREP_SRCS}
		gl_draw.cpp
		gl_mesh.cpp
		gl_model.cpp
		gl_model.h
		gl_refrag.cpp
		gl_rlight.cpp
		gl_rmain.cpp
		gl_rmisc.cpp
		gl_rsurf.cpp
		gl_screen.cpp
		gl_test.cpp
		gl_vidnt.cpp
		gl_warp.cpp
		gl_warp_sin.h
		glquake.h
	)
endif()

if( MSVC )
	set( PREP_SRCS
		${PREP_SRCS}
		winquake.rc
	)
endif()

set( PROGDEFS
	progdefs.q1
	progdefs.q2
)

add_executable( winquake ${PREP_SRCS} ${PROGDEFS} )

set_property( DIRECTORY PROPERTY VS_STARTUP_PROJECT winquake )

configure_msvc_runtime( winquake STATIC )

target_include_directories( winquake PRIVATE
	$<$<CXX_COMPILER_ID:MSVC>:${CMAKE_CURRENT_SOURCE_DIR}/dxsdk/sdk/inc>
)

if( RENDERER_IS_SOFTWARE )
	target_include_directories( winquake PRIVATE
		${CMAKE_CURRENT_SOURCE_DIR}/scitech/include
	)
endif()

target_compile_definitions( winquake PRIVATE
	$<$<CXX_COMPILER_ID:MSVC>:_CRT_SECURE_NO_WARNINGS>
	$<$<CXX_COMPILER_ID:MSVC>:$<$<CONFIG:Debug>:_DEBUG>>
	$<$<CXX_COMPILER_ID:MSVC>:CINTERFACE> #DirectX interfaces need this (for now)
)

if( RENDERER_IS_OPENGL )
	target_compile_definitions( winquake PRIVATE
		GLQUAKE
	)
endif()

target_link_libraries( winquake PRIVATE
	$<$<CXX_COMPILER_ID:MSVC>:winmm>
	$<$<CXX_COMPILER_ID:MSVC>:wsock32>
	$<$<CXX_COMPILER_ID:MSVC>:comctl32>
	$<$<CXX_COMPILER_ID:MSVC>:${CMAKE_CURRENT_SOURCE_DIR}/dxsdk/sdk/lib/dxguid.lib>
)

if( RENDERER_IS_SOFTWARE )
	target_link_libraries( winquake PRIVATE
		${CMAKE_CURRENT_SOURCE_DIR}/scitech/lib/win32/vc/mgllt.lib
	)
endif()

if( RENDERER_IS_OPENGL )
	target_link_libraries( winquake PRIVATE
		OpenGL::GLU
		OpenGL::GL
	)
endif()

if( MSVC )
	#Software mode tries to link with this, but it doesn't appear to exist for new versions of MSVC
	if( RENDERER_IS_SOFTWARE )
		set( IGNORE_LIBC "/NODEFAULTLIB:LIBC.lib" )
	endif()

	set_target_properties( winquake PROPERTIES
		# Disable warning 4805: '==': unsafe mix of type 'qboolean' and type 'bool' in operation
		# TODO: once all code has upgraded away from qboolean, remove this
		# Disable warning 4244: '=': conversion from 'type1' to 'type2', possible loss of data
		# Disable warning 4100: 'parameter': unreferenced formal parameter
		# Disable warning 4459: declaration of 'local variable' hides global declaration
		# Disable warning 4611: interaction between '_setjmp' and C++ object destruction is non-portable TODO remove setjmp and use proper error handling
		COMPILE_FLAGS "/W2 /wd4805 /wd4244 /wd4100 /wd4459 /wd4611"
		# Uses WinMain entry point
		# /SAFESEH:NO is needed because of assembly files. Can be removed once those are gone
		LINK_FLAGS "/SUBSYSTEM:WINDOWS /SAFESEH:NO ${IGNORE_LIBC}"
	)
endif()

# Until we get rid of assembly code we'll need to add these custom builds
function( configure_asm_file filename )
	set( OUT_FILENAME ${CMAKE_CURRENT_BINARY_DIR}/${filename} )
	
	if( MSVC )
		# Some commands can't handle forward slashes, so make sure they're all replaced
		STRING( REGEX REPLACE "/" "\\\\" OUT_FILENAME ${OUT_FILENAME} ) 
		
		add_custom_command(
			OUTPUT ${filename}.obj
			MAIN_DEPENDENCY ${filename}.s
			COMMAND cl /EP > ${OUT_FILENAME}.spp ${CMAKE_CURRENT_SOURCE_DIR}\\${filename}.s
			COMMAND $<TARGET_FILE:gas2masm> < ${OUT_FILENAME}.spp > ${OUT_FILENAME}.asm
			COMMAND ml /c /Cp /coff /Fo${OUT_FILENAME}.obj /Zm /Zi ${OUT_FILENAME}.asm
			COMMAND del ${OUT_FILENAME}.spp
		)
		
		target_sources( winquake PRIVATE ${filename}.obj )
	endif()
endfunction()

configure_asm_file( math )
configure_asm_file( snd_mixa )
configure_asm_file( sys_wina )
configure_asm_file( worlda )

if( UNIX )
	configure_asm_file( sys_dosa )
endif()

# Software renderer specific assembly files
if( RENDERER_IS_SOFTWARE )
	configure_asm_file( d_draw )
	configure_asm_file( d_draw16 )
	configure_asm_file( d_parta )
	configure_asm_file( d_polysa )
	configure_asm_file( d_scana )
	configure_asm_file( d_spr8 )
	configure_asm_file( d_varsa )
	configure_asm_file( r_aclipa )
	configure_asm_file( r_aliasa )
	configure_asm_file( r_drawa )
	configure_asm_file( r_edgea )
	configure_asm_file( r_varsa )
	configure_asm_file( surf8 )
	configure_asm_file( surf16 )
endif()

set_target_properties( winquake PROPERTIES
	RUNTIME_OUTPUT_DIRECTORY ${CMAKE_INSTALL_PREFIX}
)

foreach( config IN LISTS CMAKE_CONFIGURATION_TYPES )
	# The property names are all caps
	string( TOUPPER ${config} config )
	
	set_target_properties( winquake PROPERTIES
		RUNTIME_OUTPUT_DIRECTORY_${config} ${CMAKE_INSTALL_PREFIX}
	)
endforeach()