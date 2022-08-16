cmake_minimum_required(VERSION 3.0)

set(CMAKE_TOOLCHAIN_FILE "$ENV{VITASDK}/share/vita.toolchain.cmake" CACHE PATH "VitaSDK toolchain file")
set(VITASDK_CMAKE_FILE "$ENV{VITASDK}/share/vita.cmake" CACHE PATH "VitaSDK CMake functions file")
include("${VITASDK_CMAKE_FILE}" REQUIRED)

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -ffast-math -O2 -fsigned-char -Wall")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -ffast-math -O2 -fsigned-char -fpermissive -std=c++17")
set(VITA_MKSFOEX_FLAGS "${VITA_MKSFOEX_FLAGS} -d PARENTAL_LEVEL=1 -d ATTRIBUTE2=12")
set(VITA_ELF_CREATE_FLAGS "")

project(Sonic12-Vita)

include_directories(
  RSDKv5/
  dependencies/android/libogg/include/
  dependencies/android/libtheora/include/
  dependencies/android/
  dependencies/all/
  dependencies/all/tinyxml2/
  dependencies/all/iniparser/
  "$ENV{VITASDK}/arm-vita-eabi/include/SDL2"
  "$ENV{VITASDK}/arm-vita-eabi/include/freetype2"
  Game/
  Game/Objects/
)

# Build self

set(ELF "sonicmania.elf")
set(SELF "eboot.bin")

file(GLOB_RECURSE Game_SRC
    RSDKv5/*.cpp
)

add_executable("${ELF}"
  Game/Game.c
  Game/Objects/All.c
  RSDKv5/main.cpp
  RSDKv5/RSDK/Core/RetroEngine.cpp
  RSDKv5/RSDK/Core/Math.cpp
  RSDKv5/RSDK/Core/Reader.cpp
  RSDKv5/RSDK/Core/Link.cpp
  RSDKv5/RSDK/Core/ModAPI.cpp
  RSDKv5/RSDK/Dev/Debug.cpp
  RSDKv5/RSDK/Storage/Storage.cpp
  RSDKv5/RSDK/Storage/Text.cpp
  RSDKv5/RSDK/Graphics/Drawing.cpp
  RSDKv5/RSDK/Graphics/Scene3D.cpp
  RSDKv5/RSDK/Graphics/Animation.cpp
  RSDKv5/RSDK/Graphics/Sprite.cpp
  RSDKv5/RSDK/Graphics/Palette.cpp
  RSDKv5/RSDK/Graphics/Video.cpp
  RSDKv5/RSDK/Audio/Audio.cpp
  RSDKv5/RSDK/Input/Input.cpp
  RSDKv5/RSDK/Scene/Scene.cpp
  RSDKv5/RSDK/Scene/Collision.cpp
  RSDKv5/RSDK/Scene/Object.cpp
  RSDKv5/RSDK/Scene/Objects/DefaultObject.cpp
  RSDKv5/RSDK/Scene/Objects/DevOutput.cpp
  RSDKv5/RSDK/User/Core/UserAchievements.cpp
  RSDKv5/RSDK/User/Core/UserCore.cpp
  RSDKv5/RSDK/User/Core/UserLeaderboards.cpp
  RSDKv5/RSDK/User/Core/UserPresence.cpp
  RSDKv5/RSDK/User/Core/UserStats.cpp
  RSDKv5/RSDK/User/Core/UserStorage.cpp
  dependencies/all/tinyxml2/tinyxml2.cpp
  dependencies/all/iniparser/iniparser.cpp
  dependencies/all/iniparser/dictionary.cpp
  dependencies/all/miniz/miniz.c
)

target_compile_definitions("${ELF}" PUBLIC -DBASE_PATH="ux0:data/Mania/" -DGAME_MAIN=main -DRETRO_REVISION=3 -DRETRO_STANDALONE=0 -DRETRO_USE_ORIGINAL_CODE=0 -DRETRO_USE_MOD_LOADER=0 -DRETRO_DISABLE_LOG=1 -D__VITA__)

target_link_libraries("${ELF}"
  SDL2
  SDL2_image
  vorbisfile
  vorbis
  ogg
  mpg123
  FLAC
  mikmod
  opus
  freetype
  jpeg
  png
  theora
  SceMotion_stub
  SceCommonDialog_stub
  SceHid_stub
  SceGxm_stub
  SceAudio_stub
  SceAudioIn_stub
  ScePower_stub
  SceDisplay_stub
  SceCtrl_stub
  SceTouch_stub
  SceIofilemgr_stub
  SceSsl_stub
  SceSysmodule_stub
  pthread
  ${SDL2_LIBRARIES}
  gcc
  g
  m
)

vita_create_self("${SELF}"
  "${ELF}"
  UNSAFE
  NOASLR
)

# Build VPK

set(VPK "SonicMania.vpk")
set(TITLE_NAME "Sonic Mania")
set(TITLE_ID "MANIA0001")
set(TITLE_VER "01.00")

vita_create_vpk("${VPK}" "${TITLE_ID}" "${SELF}"
  NAME "${TITLE_NAME}"
  VERSION "${TITLE_VER}"
  FILE mania_livearea sce_sys
)