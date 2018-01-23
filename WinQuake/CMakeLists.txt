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
	cd_win.c
	chase.c
	cl_demo.c
	cl_input.c
	cl_main.c
	cl_parse.c
	cl_tent.c
	cmd.c
	common.c
	common.h
	conproc.c
	console.c
	crc.c
	cvar.c
	host.c
	host_cmd.c
	in_win.c
	keys.c
	math.s
	mathlib.c
	menu.c
	net_dgrm.c
	net_loop.c
	net_main.c
	net_vcr.c
	net_win.c
	net_wins.c
	net_wipx.c
	pr_cmds.c
	pr_edict.c
	pr_exec.c
	r_part.c
	sbar.c
	snd_dma.c
	snd_mem.c
	snd_mix.c
	snd_mixa.s
	snd_win.c
	sv_main.c
	sv_move.c
	sv_phys.c
	sv_user.c
	sys_win.c
	sys_wina.s
	view.c
	wad.c
	world.c
	worlda.s
	zone.c
)

if( RENDERER_IS_SOFTWARE )
	set( PREP_SRCS
		${PREP_SRCS}
		d_draw.s
		d_draw16.s
		d_edge.c
		d_fill.c
		d_init.c
		d_modech.c
		d_part.c
		d_parta.s
		d_polysa.s
		d_polyse.c
		d_scan.c
		d_scana.s
		d_sky.c
		d_spr8.s
		d_sprite.c
		d_surf.c
		d_vars.c
		d_varsa.s
		d_zpoint.c
		draw.c
		model.c
		r_aclip.c
		r_aclipa.s
		r_alias.c
		r_aliasa.s
		r_bsp.c
		r_draw.c
		r_drawa.s
		r_edge.c
		r_edgea.s
		r_efrag.c
		r_light.c
		r_main.c
		r_misc.c
		r_sky.c
		r_sprite.c
		r_surf.c
		r_vars.c
		r_varsa.s
		screen.c
		surf8.s
		surf16.s
		vid_win.c
	)
elseif( RENDERER_IS_OPENGL )
	set( PREP_SRCS
		${PREP_SRCS}
		gl_draw.c
		gl_mesh.c
		gl_model.c
		gl_refrag.c
		gl_rlight.c
		gl_rmain.c
		gl_rmisc.c
		gl_rsurf.c
		gl_screen.c
		gl_test.c
		gl_vidnt.c
		gl_warp.c
	)
endif()

add_executable( winquake ${PREP_SRCS} )

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
	$<$<CXX_COMPILER_ID:MSVC>:$<$<CONFIG:Debug>:_DEBUG>>
)

if( RENDERER_IS_OPENGL )
	target_compile_definitions( winquake PRIVATE
		$<$<CXX_COMPILER_ID:MSVC>:_CRT_SECURE_NO_WARNINGS>
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
	set_target_properties( winquake PROPERTIES
		COMPILE_FLAGS "/W1"
		#Uses WinMain entry point
		# /SAFESEH:NO is needed because of assembly files. Can be removed once those are gone
		LINK_FLAGS "/SUBSYSTEM:WINDOWS /SAFESEH:NO"
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
	configure_asm_file( surf8 )
	configure_asm_file( surf16 )
endif()
