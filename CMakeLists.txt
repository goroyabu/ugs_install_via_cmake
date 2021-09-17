#/** @file    CMakeLists.txt  **/
#/** @author  Goro Yabu       **/
#/** @date    2020/03/10      **/

project(ugs Fortran C)

# ----------

enable_language(Fortran)
cmake_minimum_required(VERSION 3.15)
set(CMAKE_Fortran_FLAGS "-std=legacy")
#set(CMAKE_STATIC_LINKER_FLAGS "rsv")

# ----------

# X11
find_package(X11 REQUIRED)
message(STATUS "X11_FOUND = ${X11_FOUND}")
message(STATUS "X11_INCLUDE_DIR = ${X11_INCLUDE_DIR}")
message(STATUS "X11_LIBRARIES = ${X11_LIBRARIES}")
message(STATUS "X11_Xaw_LIB = ${X11_Xaw_LIB}")
message(STATUS "X11_Xmu_LIB = ${X11_Xmu_LIB}")
message(STATUS "X11_Xt_LIB = ${X11_Xt_LIB}")

# ----------

set(ugs_url http://ftp.riken.jp/iris/ugs/ugs.tar.gz)
find_program(wget_command wget)
message(STATUS ${wget_command})

add_custom_command(
    OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/ugs_source
    COMMAND if [ ! -e ugs.tar.gz ]\; then ${wget_command} ${ugs_url}\; fi
    COMMAND if [ -e ${CMAKE_CURRENT_BINARY_DIR}/ugs_source ]\; then rm -r ${CMAKE_CURRENT_BINARY_DIR}/ugs_source\; fi
    COMMAND ${CMAKE_COMMAND} -E tar xzf ${CMAKE_CURRENT_BINARY_DIR}/ugs.tar.gz
    COMMAND ${CMAKE_COMMAND} -E rename ${CMAKE_CURRENT_BINARY_DIR}/ugs ${CMAKE_CURRENT_BINARY_DIR}/ugs_source
)
add_custom_target(download
    DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/ugs_source
)

# ----------

add_custom_command(
    OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/ugs_build/copy_dir_is_done.bak
    # COMMAND cp ${CMAKE_CURRENT_BINARY_DIR}/ugs_source/src.2.10e ${CMAKE_CURRENT_BINARY_DIR}/
    COMMAND mkdir -p ${CMAKE_CURRENT_BINARY_DIR}/ugs_build
    COMMAND cp -r ${CMAKE_CURRENT_BINARY_DIR}/ugs_source/src.2.10e/* ${CMAKE_CURRENT_BINARY_DIR}/ugs_build
    COMMAND touch ${CMAKE_CURRENT_BINARY_DIR}/ugs_build/copy_dir_is_done.bak
    DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/ugs_source
)

set(ugsystem_files)
set(
    old_ugsystem_files
    UGC00CBK.FOR UGD00CBK.FOR UGDDACBK.FOR UGE00CBK.FOR UGEMSCBK.FOR UGERRCBK.FOR
    UGF00CBK.FOR UGG00CBK.FOR UGMCACBK.FOR UGPOTCBK.FOR UGPOTCBK.org UGPOTDCL.FOR
)
foreach(oldfile IN LISTS old_ugsystem_files)
    add_custom_command(
        OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/ugs_build/UGSYSTEM_${oldfile}
        COMMAND cp ${CMAKE_CURRENT_BINARY_DIR}/ugs_build/UGSYSTEM:${oldfile} ${CMAKE_CURRENT_BINARY_DIR}/ugs_build/UGSYSTEM_${oldfile}
        DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/ugs_build/copy_dir_is_done.bak
    )
    list(APPEND ugsystem_files ${CMAKE_CURRENT_BINARY_DIR}/ugs_build/UGSYSTEM_${oldfile})
endforeach()

set(
    old_ugsystem_files
    UGDDACBK.FOR UGDDXEPS.FOR UGDDXGIN.FOR UGDDXGRN.FOR UGDDXGSD.FOR UGDDXGSQ.FOR
    UGDDXIM3.FOR UGDDXIMX.FOR UGDDXMET.FOR UGDDXPDI.FOR UGDDXPDL.FOR UGDDXPDS.FOR
    UGDDXPDU.FOR UGDDXPRX.FOR UGDDXPSC.FOR UGDDXQMS.FOR UGDDXSKB.FOR UGDDXSKC.FOR
    UGDDXSKD.FOR UGDDXSKE.FOR UGDDXSSS.FOR UGDDXTAL.FOR UGDDXTIN.FOR UGDDXTIZ.FOR
    UGDDXTKA.FOR UGDDXTKB.FOR UGDDXTKC.FOR UGDDXTKD.FOR UGDDXTKE.FOR UGDDXTKZ.FOR
    UGDDXTSD.FOR UGDDXTSQ.FOR UGDDXTXA.FOR UGDDXTXB.FOR UGDDXTXC.FOR UGDDXUIN.FOR
    UGDDXUSD.FOR UGDDXUSQ.FOR UGDDXVI2.FOR UGDDXVPF.FOR UGDDXVS2.FOR UGDDXXWI.FOR
    UGDDXXWS.FOR UGIOPARM.FOR
)
foreach(oldfile IN LISTS old_ugsystem_files)
    add_custom_command(
        OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/ugs_build/drivers/UGSYSTEM_${oldfile}
        COMMAND cp ${CMAKE_CURRENT_BINARY_DIR}/ugs_build/drivers/UGSYSTEM:${oldfile} ${CMAKE_CURRENT_BINARY_DIR}/ugs_build/drivers/UGSYSTEM_${oldfile}
        DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/ugs_build/copy_dir_is_done.bak
    )
    list(APPEND ugsystem_files ${CMAKE_CURRENT_BINARY_DIR}/ugs_build/drivers/UGSYSTEM_${oldfile})
endforeach()

add_custom_command(
    OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/ugs_build/mod_files_is_done.bak
    COMMAND grep -l -r UGSYSTEM ${CMAKE_CURRENT_BINARY_DIR}/ugs_build | grep -v .bak | xargs sed -i.bak -e "s/UGSYSTEM\:/UGSYSTEM_/g"
    # COMMAND grep -l -r "'INTEGER\\*2\ \ \ \ \ CHC'" ugs_build/ | xargs echo
    # COMMAND echo "test1"
    COMMAND grep -l -r "'INTEGER\\*2\ \ \ \ \ CHC'" ugs_build/ | grep -v .bak | xargs sed -i.bak -e "'s/INTEGER\\*2     CHC/CHARACTER\\*2     CHC/g'" 
    # COMMAND echo "test2"
    # COMMAND echo xargs sed -i.bak -e "s/INTEGER\*2     CHC/CHARACTER\*2     CHC/g"
    COMMAND touch ${CMAKE_CURRENT_BINARY_DIR}/ugs_build/mod_files_is_done.bak
    DEPENDS ${ugsystem_files}
)
add_custom_target(
    pre_cmake ALL
    SOURCES ${CMAKE_CURRENT_BINARY_DIR}/ugs_build/mod_files_is_done.bak
)

# ----------

# ugs
set(ugs_source_paths)
set(
    ugs_source_names
    aux.c ran.f ug2dhg.f ug2dhp.f ug3lin.f ug3mrk.f ug3pln.f ug3pmk.f
    ug3trn.f ug3txt.f ug3wrd.f ugb001.f ugb002.f ugb003.f ugb004.f ugb005.f
    ugb006.f ugb007.f ugb008.f ugb009.f ugb010.f ugb011.f ugb012.f ugb013.f
    ugb014.f ugb015.f ugc001.f ugc002.f ugc003.f ugc004.f ugc005.f ugc006.f
    ugc007.f ugclos.f ugcnt1.f ugcnt2.f ugcnt3.f ugcnt4.f ugcntr.f ugcnvf.f
    ugctol.f ugd001.f ugd002.f ugd003.f ugddat.f ugdefl.f ugdsab.f ugdspc.f
    ugdupl.f uge001.f uge003.f ugectl.f ugenab.f ugevnt.f ugf001.f ugf002.f
    ugf003.f ugf004.f ugfont.f ugg001.f ugg002.f ugg003.f ugg004.f ugg005.f
    uginfo.f uginit.f uglgax.f uglgdx.f ugline.f uglnax.f uglndx.f ugmark.f
    ugmctl.f  ugmesh.f ugnucl.f ugoption.f ugpfil.f ugpict.f ugplin.f ugpmrk.f
    ugproj.f ugqctr.f ugrerr.f ugscin.f ugshld.f ugsimp.f ugslct.f ugtext.f
    ugtran.f ugwdow.f ugwrit.f ugxerr.f ugxhch.f ugxsym.f ugxtxt.f ugz001.f
    ugz002.f ugz003.f ugz006.f uge002.F ugfrev.F uggetv.F ugopen.F ugz005.F
    bit/btest.c bit/iand.c bit/ibclr.c bit/ibset.c bit/ior.c bit/ishft.c bit/ishftc.c
    drivers/epsf.f drivers/postscr.f drivers/rotated.c
    drivers/tek4010.f drivers/xwindow.f drivers/xwindowc.c
    dummies/postscr.f dummies/tek4010.f dummies/ugcw01.f dummies/uggd01.f dummies/uggi01.f
    dummies/uggks_dummy.f dummies/uggr01.f dummies/uggs01.f dummies/ugin01.f dummies/ugix01.f
    dummies/ugmt01.f dummies/ugpi01.f dummies/ugpl01.f dummies/ugpm01.f dummies/ugps01.f
    dummies/ugpu01.f dummies/ugpx01.f dummies/ugqm01.f dummies/ugsa01.f dummies/ugsb01.f
    dummies/ugsc01.f dummies/ugsd01.f dummies/ugse01.f dummies/ugsixel_dummy.f dummies/ugsx01.f
    dummies/ugta01.f dummies/ugtd01.f dummies/ugts01.f dummies/ugtx01.f dummies/ugud01.f
    dummies/uguis_dummy.f dummies/ugus01.f dummies/ugux01.f dummies/ugvf01.f dummies/ugvi01.f
    dummies/ugvs01.f dummies/ugwa01.f dummies/ugwb01.f dummies/ugwc01.f dummies/ugwd01.f
    dummies/ugwe01.f dummies/ugwz01.f dummies/ugxa01.f dummies/ugxb01.f dummies/ugxc01.f
    dummies/ugxs01.f dummies/ugzz01.f dummies/x-window.f
)
foreach(src_name IN LISTS ugs_source_names)
    add_custom_command(
        OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/ugs_build/${src_name}
        DEPENDS pre_cmake
    )
    list(APPEND ugs_source_paths ${CMAKE_CURRENT_BINARY_DIR}/ugs_build/${src_name})
endforeach()

add_library(${PROJECT_NAME} STATIC ${ugs_source_paths})
target_compile_options(${PROJECT_NAME} PRIVATE
    $<$<AND:$<PLATFORM_ID:Darwin>,$<COMPILE_LANGUAGE:Fortran>>: -O2 -fno-automatic -finit-local-zero -fno-backslash -funroll-loops -fomit-frame-pointer -ftree-vectorize  -flat_namespace>
    $<$<AND:$<PLATFORM_ID:Darwin>,$<COMPILE_LANGUAGE:C>>: -O2 -DIUCLC=0001000 -ftree-vectorize>
)
target_include_directories(${PROJECT_NAME} PRIVATE ${CMAKE_CURRENT_BINARY_DIR}/ugs_build/drivers ${X11_INCLUDE_DIR})
target_link_libraries(${PROJECT_NAME} ${X11_LIBRARIES} ${X11_Xaw_LIB} ${X11_Xmu_LIB} ${X11_Xt_LIB})

install(
    TARGETS ${PROJECT_NAME}
    EXPORT ${PROJECT_NAME}-config
    DESTINATION lib/${PROJECT_NAME}
    INCLUDES DESTINATION include
    PUBLIC_HEADER DESTINATION include/${PROJECT_NAME}
)
install(
    EXPORT ${PROJECT_NAME}-config
    DESTINATION lib/${PROJECT_NAME}
    EXPORT_LINK_INTERFACE_LIBRARIES
)

# FILES NOT USED
# src/gothic.f src/ubcfont.f src/ug2dh1.f src/ug2dh2.f src/ug2dh3.f
# src/ug2dh4.f src/ug2dh5.f src/ug2dh6.f src/ug2dh7.f src/ugmes1.f
# src/ugmes2.f src/ugmes3.f src/ugmes4.f src/ugqct1.f src/ugtrn1.f
# src/ugtrn2.f

# ----------

# xwtest
# add_library(xwindow.o source/drivers/xwindowc.c source/drivers/cursor1.bmp source/drivers/cursor2.bmp source/drivers/icon.bmp)
# target_include_directories(xwindow.o PRIVATE source/drivers ${X11_INCLUDE_DIR})
#
# add_library(rotated.o source/drivers/rotated.c)
# target_include_directories(rotated.o PRIVATE source/drivers ${X11_INCLUDE_DIR})

add_custom_command(
    OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/xwtest_src
    COMMAND mkdir -p ${CMAKE_CURRENT_BINARY_DIR}/xwtest_src
)

add_custom_command(
    OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/xwtest_src/xwindowc.c
    COMMAND cat ${CMAKE_CURRENT_BINARY_DIR}/ugs_build/drivers/xwindowc.c | sed -e "s/main/void main/g" > ${CMAKE_CURRENT_BINARY_DIR}/xwtest_src/xwindowc.c
    DEPENDS ${ugs_source_paths} ${CMAKE_CURRENT_BINARY_DIR}/xwtest_src
)

set(xwtest_source_names cursor1.bmp cursor2.bmp icon.bmp rotated.c)
foreach(src_name IN LISTS xwtest_source_names)
    add_custom_command(
        OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/ugs_build/drivers/${src_name}
        DEPENDS pre_cmake
    )
endforeach()

add_executable(
    xwtest
    ${CMAKE_CURRENT_BINARY_DIR}/xwtest_src/xwindowc.c
    ${CMAKE_CURRENT_BINARY_DIR}/ugs_build/drivers/cursor1.bmp
    ${CMAKE_CURRENT_BINARY_DIR}/ugs_build/drivers/cursor2.bmp
    ${CMAKE_CURRENT_BINARY_DIR}/ugs_build/drivers/icon.bmp
    ${CMAKE_CURRENT_BINARY_DIR}/ugs_build/drivers/rotated.c
)
target_compile_options(xwtest PRIVATE "-DSELFTEST")
target_include_directories(
    xwtest PRIVATE
    ${CMAKE_CURRENT_BINARY_DIR}/ugs_build/drivers
    ${X11_INCLUDE_DIR}
)
target_link_libraries(xwtest ${X11_LIBRARIES} ${X11_Xaw_LIB} ${X11_Xmu_LIB} ${X11_Xt_LIB})
install(TARGETS xwtest DESTINATION bin)

# ----------
# Uninstall
configure_file(cmake_uninstall.cmake.in cmake_uninstall.cmake IMMEDIATE @ONLY)
add_custom_target(uninstall COMMAND "${CMAKE_COMMAND}" -P cmake_uninstall.cmake)
