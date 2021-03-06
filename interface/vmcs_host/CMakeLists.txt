
# interface/vmcs_host

# not working in release build
# add_definitions(-Werror)

# vc_vchi_gencmd.c has a type-punning problem in vc_gencmd_read_response
add_definitions(-fno-strict-aliasing)

include_directories(${VMCS_TARGET}/vcfiled)

if(WIN32)     
    set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DWIN32DLL_EXPORTS -DVCOS_PROVIDER")
    add_library(vchostif SHARED
            vc_vchi_gencmd.c
            vc_vchi_tvservice.c vc_vchi_cecservice.c
            vc_vchi_dispmanx.c vc_service_common.c)
else()
    add_library(vchostif
                ${VMCS_TARGET}/vcfilesys.c ${VMCS_TARGET}/vcmisc.c
                vc_vchi_gencmd.c vc_vchi_filesys.c
                vc_vchi_tvservice.c vc_vchi_cecservice.c
                vc_vchi_dispmanx.c vc_service_common.c)
endif()


#            ${VMCS_TARGET}/vmcs_main.c
#  vc_vchi_haud.c
#add_library(bufman            vc_vchi_bufman.c            )

# OpenMAX/IL component service
add_library(vcilcs
            vcilcs.c vcilcs_in.c vcilcs_out.c vcilcs_common.c)

# ILCS pulls in EGL for the ILCS/EGL surface API support
target_link_libraries(vcilcs EGL GLESv2 khrn_client vchiq_arm vcos)

# vchostif needs ilcore as well (vmcs_main pulls it in)
if(WIN32)
    target_link_libraries(vchostif vchiq_arm vcos)
else()
    target_link_libraries(vchostif vchiq_arm vcos vcfiled_check)
endif()
#bufman  vmcs_rpc_client

#target_link_libraries(bufman WFC)

if(WIN32)
    # TODO Determine if windows requires vcfiled
else()
    add_subdirectory(linux/vcfiled)
endif()

install(TARGETS vchostif vcilcs DESTINATION lib)

