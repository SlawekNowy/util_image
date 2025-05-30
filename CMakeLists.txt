include(${CMAKE_SOURCE_DIR}/cmake/pr_common.cmake)

# TODO: Enable by default once it's fully integrated
option(USE_COMPRESSONATOR "Use Compressonator compressor instead of nvtt" OFF)

if(WIN32)
	# nvtt does not work with clang on windows
	if(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
		set(USE_COMPRESSONATOR ON CACHE BOOL "Use Compressonator compressor instead of nvtt" FORCE)
	endif()
endif()

set(PROJ_NAME util_image)
pr_add_library(${PROJ_NAME} SHARED)

pr_add_dependency(${PROJ_NAME} vfilesystem TARGET PUBLIC)
pr_add_dependency(${PROJ_NAME} lunasvg TARGET)

pr_add_external_dependency(${PROJ_NAME} libpng LIBRARY PRIVATE)
pr_add_external_dependency(${PROJ_NAME} libpng_build HEADER_ONLY PRIVATE)
pr_add_external_dependency(${PROJ_NAME} zlib LIBRARY PRIVATE)

pr_add_headers(${PROJ_NAME} "include/")
pr_add_sources(${PROJ_NAME} "src/")

pr_add_compile_definitions(${PROJ_NAME} -DUIMG_ENABLE_NVTT PUBLIC)
pr_add_compile_definitions(${PROJ_NAME} -DUIMG_ENABLE_SVG PUBLIC)
pr_add_compile_definitions(${PROJ_NAME} -DUIMG_DLL)

set(COMPRESSONATOR_BASE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/third_party_libs/compressonator")

set(DEPENDENCY_COMPRESSONATOR_INCLUDE "${COMPRESSONATOR_BASE_PATH}/cmp_compressonatorlib" CACHE PATH "Path to compressonator include directory.")
set(DEPENDENCY_COMPRESSONATOR_OPENEXR_HALF_INCLUDE "${COMPRESSONATOR_BASE_PATH}/../common/lib/ext/openexr/ilmbase-2.2.0/Half" CACHE PATH "Path to compressonator_openexr_half include directory.")

set(TEX_COMPRESSION_LIBRARY_NVTT 0)
set(TEX_COMPRESSION_LIBRARY_COMPRESSONATOR 1)

if(${USE_COMPRESSONATOR})
	set(TEX_COMPRESSION_LIBRARY TEX_COMPRESSION_LIBRARY_COMPRESSONATOR)
else()
	set(TEX_COMPRESSION_LIBRARY TEX_COMPRESSION_LIBRARY_NVTT)
endif()

pr_add_compile_definitions(${PROJ_NAME} -DTEX_COMPRESSION_LIBRARY=${TEX_COMPRESSION_LIBRARY} PUBLIC)


set(BUILD_SHARED_LIBS OFF CACHE BOOL OFF FORCE)
if(${TEX_COMPRESSION_LIBRARY} EQUAL ${TEX_COMPRESSION_LIBRARY_NVTT})
	add_subdirectory("third_party_libs/nvtt")

	set(DEPENDENCY_NVIDIA_TEXTURE_TOOLS_INCLUDE ${CMAKE_CURRENT_LIST_DIR}/third_party_libs/nvtt/src CACHE PATH "" FORCE)
	set_target_properties(nvtt PROPERTIES FOLDER third_party_libs/nvtt)
	set_target_properties(bc6h PROPERTIES FOLDER third_party_libs/nvtt)
	set_target_properties(bc7 PROPERTIES FOLDER third_party_libs/nvtt)
	set_target_properties(CMP_Core PROPERTIES FOLDER third_party_libs/nvtt)
	set_target_properties(cubemaptest PROPERTIES FOLDER third_party_libs/nvtt)
	set_target_properties(EtcLib PROPERTIES FOLDER third_party_libs/nvtt)
	set_target_properties(filtertest PROPERTIES FOLDER third_party_libs/nvtt)
	set_target_properties(imperativeapi PROPERTIES FOLDER third_party_libs/nvtt)
	set_target_properties(nvassemble PROPERTIES FOLDER third_party_libs/nvtt)
	set_target_properties(nvcompress PROPERTIES FOLDER third_party_libs/nvtt)
	set_target_properties(nvcore PROPERTIES FOLDER third_party_libs/nvtt)
	set_target_properties(nvddsinfo PROPERTIES FOLDER third_party_libs/nvtt)
	set_target_properties(nvdecompress PROPERTIES FOLDER third_party_libs/nvtt)
	set_target_properties(nv-gnome-thumbnailer PROPERTIES FOLDER third_party_libs/nvtt)
	set_target_properties(nvhdrtest PROPERTIES FOLDER third_party_libs/nvtt)
	set_target_properties(nvimage PROPERTIES FOLDER third_party_libs/nvtt)
	set_target_properties(nvimgdiff PROPERTIES FOLDER third_party_libs/nvtt)
	set_target_properties(nvmath PROPERTIES FOLDER third_party_libs/nvtt)
	set_target_properties(nvsquish PROPERTIES FOLDER third_party_libs/nvtt)
	set_target_properties(nvtestsuite PROPERTIES FOLDER third_party_libs/nvtt)
	set_target_properties(nvthread PROPERTIES FOLDER third_party_libs/nvtt)
	set_target_properties(nvzoom PROPERTIES FOLDER third_party_libs/nvtt)
	set_target_properties(posh PROPERTIES FOLDER third_party_libs/nvtt)
	set_target_properties(process_alpha_map PROPERTIES FOLDER third_party_libs/nvtt)
	set_target_properties(rg_etc1 PROPERTIES FOLDER third_party_libs/nvtt)
	set_target_properties(squish PROPERTIES FOLDER third_party_libs/nvtt)

	add_dependencies(${PROJ_NAME} nvtt)

	#HACK: Somehow this path was not included to util_image?
	target_include_directories(${PROJ_NAME} PRIVATE ${DEPENDENCY_NVIDIA_TEXTURE_TOOLS_INCLUDE})
	add_dependencies(${PROJ_NAME} squish)

	pr_add_dependency(${PROJ_NAME} nvtt TARGET PRIVATE)
	pr_add_dependency(${PROJ_NAME} nvsquish TARGET PRIVATE)
	pr_add_dependency(${PROJ_NAME} nvthread TARGET PRIVATE)
	pr_add_dependency(${PROJ_NAME} nvimage TARGET PRIVATE)
	pr_add_dependency(${PROJ_NAME} bc7 TARGET PRIVATE)
	pr_add_dependency(${PROJ_NAME} bc6h TARGET PRIVATE)
	pr_add_dependency(${PROJ_NAME} nvmath TARGET PRIVATE)
	pr_add_dependency(${PROJ_NAME} nvcore TARGET PRIVATE)
	#TODO: This would get shoved in. If TEX_COMPRESSION_LIBRARY would change to compressonator, this dep would be lost.
	pr_add_dependency(${PROJ_NAME} squish TARGET PRIVATE)
else()
	if(UNIX)
		set(DEPENDENCY_COMPRESSONATOR_FRAMEWORK_INCLUDE "${DEPENDENCY_COMPRESSONATOR_SOURCE_DIR}/cmp_compressonatorlib")
		set(DEPENDENCY_COMPRESSONATOR_FRAMEWORK_LIBRARY "${DEPENDENCY_COMPRESSONATOR_LIBRARY_DIR}/libCMP_Framework.so")
		pr_add_external_dependency(${PROJ_NAME} compressonator_framework LIBRARY PRIVATE)

		set(DEPENDENCY_COMPRESSONATOR_LIBRARY "${DEPENDENCY_COMPRESSONATOR_LIBRARY_DIR}/libCMP_Compressonator.so")
		pr_add_external_dependency(${PROJ_NAME} compressonator LIBRARY PRIVATE)
	else()
		set(DEPENDENCY_COMPRESSONATOR_FRAMEWORK_INCLUDE "${DEPENDENCY_COMPRESSONATOR_SOURCE_DIR}/cmp_compressonatorlib")
		set(DEPENDENCY_COMPRESSONATOR_FRAMEWORK_LIBRARY "${DEPENDENCY_COMPRESSONATOR_LIBRARY_DIR}/CMP_Framework.lib")
		pr_add_external_dependency(${PROJ_NAME} compressonator_framework LIBRARY PRIVATE)

		set(DEPENDENCY_COMPRESSONATOR_LIBRARY "${DEPENDENCY_COMPRESSONATOR_LIBRARY_DIR}/CMP_Compressonator.lib")
		pr_add_external_dependency(${PROJ_NAME} compressonator LIBRARY PRIVATE)

		set(DEPENDENCY_COMPRESSONATOR_CORE_LIBRARY "${DEPENDENCY_COMPRESSONATOR_LIBRARY_DIR}/CMP_Core.lib")
		pr_add_external_dependency(${PROJ_NAME} compressonator_core LIBRARY PRIVATE)

		set(DEPENDENCY_COMPRESSONATOR_COMMON_LIBRARY "${DEPENDENCY_COMPRESSONATOR_LIBRARY_DIR}/CMP_Common.lib")
		pr_add_external_dependency(${PROJ_NAME} compressonator_common LIBRARY PRIVATE)
	endif()

	set(DEPENDENCY_COMPRESSONATOR_HALF_INCLUDE "${DEPENDENCY_COMPRESSONATOR_SOURCE_DIR}/cmp_framework/common/half")
	pr_add_external_dependency(${PROJ_NAME} compressonator_half HEADER_ONLY PRIVATE)
endif()
set(BUILD_SHARED_LIBS ON CACHE BOOL ON FORCE)

pr_finalize(${PROJ_NAME})
