cmake_minimum_required(VERSION 3.16)

project(minorGems LANGUAGES CXX)

# TODO: compile for platforms other then linux
# TODO: figure out what's up with GAME_GRAPHICS variable

set(ROOT_PATH "${CMAKE_CURRENT_LIST_DIR}/..")

if(CMAKE_SYSTEM MATCHES Linux)
	set(PLATFORM "Linux")
	set(PLATFORM_PATH "linux")

	set(TIME_PLATFORM "Unix")
	set(TIME_PLATFORM_PATH "unix")

	set(DIRECTORY_PLATFORM "Unix")
	set(DIRECTORY_PLATFORM_PATH "unix")

	set(POLL_PLATFORM "Linux")
	set(POLL_PLATFORM_PATH "linux")
elseif(CMAKE_SYSTEM MATCHES Windows)
	set(PLATFORM "Win32")
	set(PLATFORM_PATH "win32")

	set(TIME_PLATFORM "Win32")
	set(TIME_PLATFORM_PATH "win32")

	set(DIRECTORY_PLATFORM "Win32")
	set(DIRECTORY_PLATFORM_PATH "win32")

	set(POLL_PLATFORM "Unix")
	set(POLL_PLATFORM_PATH "unix")
endif()

set(PLATFORM_DIRECTORY "${ROOT_PATH}/minorGems/io/file/${DIRECTORY_PLATFORM_PATH}/Directory${DIRECTORY_PLATFORM}")
set(PLATFORM_TIME "${ROOT_PATH}/minorGems/system/${TIME_PLATFORM_PATH}/Time${TIME_PLATFORM}")
set(PLATFORM_HOST_ADDRESS "${ROOT_PATH}/minorGems/network/${PLATFORM_PATH}/HostAddress${PLATFORM}")
set(PLATFORM_SOCKET "${ROOT_PATH}/minorGems/network/${PLATFORM_PATH}/Socket${PLATFORM}")
set(PLATFORM_SOCKET_CLIENT "${ROOT_PATH}/minorGems/network/${PLATFORM_PATH}/SocketClient${PLATFORM}")
set(PLATFORM_SOCKET_SERVER "${ROOT_PATH}/minorGems/network/${PLATFORM_PATH}/SocketServer${PLATFORM}")
set(PLATFORM_SOCKET_POLL "${ROOT_PATH}/minorGems/network/${POLL_PLATFORM_PATH}/SocketPoll${POLL_PLATFORM}")
set(PLATFORM_SOCKET_UDP "${ROOT_PATH}/minorGems/network/${SOCKET_UDP_PLATFORM_PATH}/SocketUDP${SOCKET_UDP_PLATFORM}")

set(PLATFORM_TYPE_IO "${ROOT_PATH}/minorGems/io/${PLATFORM_PATH}/TypeIO${PLATFORM}")
set(PLATFORM_THREAD "${ROOT_PATH}/minorGems/system/${PLATFORM_PATH}/Thread${PLATFORM}")
set(PLATFORM_MUTEX_LOCK "${ROOT_PATH}/minorGems/system/${PLATFORM_PATH}/MutexLock${PLATFORM}")
set(PLATFORM_BINARY_SEMAPHORE "${ROOT_PATH}/minorGems/system/${PLATFORM_PATH}/BinarySemaphore${PLATFORM}")

# base headless
add_library(MINOR_GEMS_BASE INTERFACE)
target_include_directories(MINOR_GEMS_BASE INTERFACE "${ROOT_PATH}")
find_package(Threads REQUIRED)
target_link_libraries(MINOR_GEMS_BASE INTERFACE Threads::Threads)

# graphics libs
# linking with this is equivalent to `LINK_HEADLESS = no` int the makeFileList
add_library(MINOR_GEMS_GRAPHICS_LIBS INTERFACE)

find_package(OpenGL REQUIRED)
find_package(SDL REQUIRED)
target_link_libraries(MINOR_GEMS_GRAPHICS_LIBS INTERFACE
	OpenGL::GL
	OpenGL::GLU
	# don't use SDL::SDL since cmake 1.16 doesn't support it
	${SDL_LIBRARY}
)

# libjpeg
# linking with this is equivalent to `LINK_AGAINST_LIBPNG = yes` in the makeFileList
add_library(MINOR_GEMS_LIBJPEG STATIC "${ROOT_PATH}/minorGems/graphics/converters/JPEGImageConverter.cpp")
target_link_libraries(MINOR_GEMS_LIBJPEG PRIVATE MINOR_GEMS_BASE)

find_package(JPEG) # FIXME? what happens if libjpeg isn't found on the system?
target_link_libraries(MINOR_GEMS_LIBJPEG PUBLIC JPEG::JPEG)
target_compile_definitions(MINOR_GEMS_LIBJPEG PUBLIC USE_JPEG)

# libpng
# linking with this is equivalent to `LINK_AGAINST_LIBPNG = yes` in the makeFileList
add_library(MINOR_GEMS_LIBPNG STATIC "${ROOT_PATH}/minorGems/graphics/converters/PNGImageConverter.cpp")
target_link_libraries(MINOR_GEMS_LIBPNG PRIVATE MINOR_GEMS_BASE)

find_package(PNG) # FIXME? what happens if libpng isn't found on the system?
target_link_libraries(MINOR_GEMS_LIBPNG PUBLIC PNG::PNG)
target_compile_definitions(MINOR_GEMS_LIBPNG PUBLIC USE_PNG)

# screen gl sdl
add_library(MINOR_GEMS_SCREEN_GL_SDL STATIC "${ROOT_PATH}/minorGems/graphics/openGL/ScreenGL_SDL.cpp")
target_link_libraries(MINOR_GEMS_SCREEN_GL_SDL PRIVATE
	MINOR_GEMS_BASE
	MINOR_GEMS_SINGLE_TEXTURE_GL
)

# single texture gl
add_library(MINOR_GEMS_SINGLE_TEXTURE_GL STATIC "${ROOT_PATH}/minorGems/graphics/openGL/SingleTextureGL.cpp")
target_link_libraries(MINOR_GEMS_SINGLE_TEXTURE_GL PRIVATE MINOR_GEMS_BASE MINOR_GEMS_GRAPHICS_LIBS)

# type io
add_library(MINOR_GEMS_TYPE_IO STATIC "${PLATFORM_TYPE_IO}.cpp")
target_link_libraries(MINOR_GEMS_TYPE_IO PRIVATE MINOR_GEMS_BASE)

# string utils
add_library(MINOR_GEMS_STRING_UTILS STATIC "${ROOT_PATH}/minorGems/util/stringUtils.cpp")
target_link_libraries(MINOR_GEMS_STRING_UTILS PRIVATE MINOR_GEMS_BASE)

# string buffer output stream
add_library(MINOR_GEMS_STRING_BUFFER_OUTPUT_STREAM STATIC "${ROOT_PATH}/minorGems/util/StringBufferOutputStream.cpp")
target_link_libraries(MINOR_GEMS_STRING_BUFFER_OUTPUT_STREAM PRIVATE MINOR_GEMS_BASE)

# byte buffer input stream
add_library(MINOR_GEMS_BYTE_BUFFER_INPUT_STREAM STATIC "${ROOT_PATH}/minorGems/util/ByteBufferInputStream.cpp")
target_link_libraries(MINOR_GEMS_BYTE_BUFFER_INPUT_STREAM PRIVATE MINOR_GEMS_BASE)

# path
add_library(MINOR_GEMS_PATH STATIC "${ROOT_PATH}/minorGems/io/file/${PLATFORM_PATH}/Path${PLATFORM}.cpp")
target_link_libraries(MINOR_GEMS_PATH PRIVATE MINOR_GEMS_BASE)

# time
add_library(MINOR_GEMS_TIME STATIC "${PLATFORM_TIME}.cpp")
target_link_libraries(MINOR_GEMS_TIME PRIVATE MINOR_GEMS_BASE)

# thread
add_library(MINOR_GEMS_THREAD STATIC "${PLATFORM_THREAD}.cpp")
target_link_libraries(MINOR_GEMS_THREAD PRIVATE MINOR_GEMS_BASE)

# mutex lock
add_library(MINOR_GEMS_MUTEX_LOCK STATIC "${PLATFORM_MUTEX_LOCK}.cpp")
target_link_libraries(MINOR_GEMS_MUTEX_LOCK PRIVATE MINOR_GEMS_BASE)

# binary semaphore
add_library(MINOR_GEMS_BINARY_SEMAPHORE STATIC "${PLATFORM_BINARY_SEMAPHORE}.cpp")
target_link_libraries(MINOR_GEMS_BINARY_SEMAPHORE PRIVATE MINOR_GEMS_BASE)

# translation manager
add_library(MINOR_GEMS_TRANSLATION_MANAGER STATIC "${ROOT_PATH}/minorGems/util/TranslationManager.cpp")
target_link_libraries(MINOR_GEMS_TRANSLATION_MANAGER PRIVATE MINOR_GEMS_BASE)

# socket
add_library(MINOR_GEMS_SOCKET STATIC "${PLATFORM_SOCKET}.cpp")
target_link_libraries(MINOR_GEMS_SOCKET PRIVATE
	MINOR_GEMS_BASE
	MINOR_GEMS_NETWORK_FUNCTION_LOCKS
)

# host address
add_library(MINOR_GEMS_HOST_ADDRESS STATIC "${PLATFORM_HOST_ADDRESS}.cpp")
target_link_libraries(MINOR_GEMS_HOST_ADDRESS PRIVATE MINOR_GEMS_BASE)

# socket client
add_library(MINOR_GEMS_SOCKET_CLIENT STATIC "${PLATFORM_SOCKET_CLIENT}.cpp")
target_link_libraries(MINOR_GEMS_SOCKET_CLIENT PRIVATE MINOR_GEMS_BASE)

# socket server
add_library(MINOR_GEMS_SOCKET_SERVER STATIC "${PLATFORM_SOCKET_SERVER}.cpp")
target_link_libraries(MINOR_GEMS_SOCKET_SERVER PRIVATE MINOR_GEMS_BASE)

# socket poll
add_library(MINOR_GEMS_SOCKET_POLL STATIC "${PLATFORM_SOCKET_POLL}.cpp")
target_link_libraries(MINOR_GEMS_SOCKET_POLL PRIVATE MINOR_GEMS_BASE)

# network function locks
add_library(MINOR_GEMS_NETWORK_FUNCTION_LOCKS STATIC "${ROOT_PATH}/minorGems/network/NetworkFunctionLocks.cpp")
target_link_libraries(MINOR_GEMS_NETWORK_FUNCTION_LOCKS PRIVATE MINOR_GEMS_BASE)

# lookup thread
add_library(MINOR_GEMS_LOOKUP_THREAD STATIC "${ROOT_PATH}/minorGems/network/LookupThread.cpp")
target_link_libraries(MINOR_GEMS_LOOKUP_THREAD PRIVATE
	MINOR_GEMS_BASE
	MINOR_GEMS_FINISHED_SIGNAL_THREAD
	MINOR_GEMS_HOST_ADDRESS
	MINOR_GEMS_THREAD
)

# web request
add_library(MINOR_GEMS_WEB_REQUEST STATIC "${ROOT_PATH}/minorGems/network/web/WebRequest.cpp")
target_link_libraries(MINOR_GEMS_WEB_REQUEST PRIVATE
	MINOR_GEMS_BASE
	MINOR_GEMS_LOOKUP_THREAD
	MINOR_GEMS_SOCKET
	MINOR_GEMS_STRING_BUFFER_OUTPUT_STREAM
)

# url utils
add_library(MINOR_GEMS_URL_UTILS STATIC "${ROOT_PATH}/minorGems/network/web/URLUtils.cpp")
target_link_libraries(MINOR_GEMS_URL_UTILS PRIVATE MINOR_GEMS_BASE)

# settings manager
add_library(MINOR_GEMS_SETTINGS_MANAGER STATIC "${ROOT_PATH}/minorGems/util/SettingsManager.cpp")
target_link_libraries(MINOR_GEMS_SETTINGS_MANAGER PRIVATE MINOR_GEMS_BASE)

# finished signal thread
add_library(MINOR_GEMS_FINISHED_SIGNAL_THREAD STATIC "${ROOT_PATH}/minorGems/system/FinishedSignalThread.cpp")
target_link_libraries(MINOR_GEMS_FINISHED_SIGNAL_THREAD PRIVATE MINOR_GEMS_BASE)

# stop signal thread
add_library(MINOR_GEMS_STOP_SIGNAL_THREAD STATIC "${ROOT_PATH}/minorGems/system/StopSignalThread.cpp")
target_link_libraries(MINOR_GEMS_STOP_SIGNAL_THREAD PRIVATE
	MINOR_GEMS_BASE
	MINOR_GEMS_BINARY_SEMAPHORE
)

# sha1
add_library(MINOR_GEMS_SHA1 STATIC "${ROOT_PATH}/minorGems/crypto/hashes/sha1.cpp")
target_link_libraries(MINOR_GEMS_SHA1 PRIVATE MINOR_GEMS_BASE)

# encoding utils
add_library(MINOR_GEMS_ENCODING_UTILS STATIC "${ROOT_PATH}/minorGems/formats/encodingUtils.cpp")
target_link_libraries(MINOR_GEMS_ENCODING_UTILS PRIVATE MINOR_GEMS_BASE)

# directory
add_library(MINOR_GEMS_DIRECTORY STATIC "${PLATFORM_DIRECTORY}.cpp")
target_link_libraries(MINOR_GEMS_DIRECTORY PRIVATE MINOR_GEMS_BASE)

# log
add_library(MINOR_GEMS_LOG STATIC "${ROOT_PATH}/minorGems/util/log/Log.cpp")
target_link_libraries(MINOR_GEMS_LOG PRIVATE MINOR_GEMS_BASE)

# app log
add_library(MINOR_GEMS_APP_LOG STATIC "${ROOT_PATH}/minorGems/util/log/AppLog.cpp")
target_link_libraries(MINOR_GEMS_APP_LOG PRIVATE MINOR_GEMS_BASE MINOR_GEMS_LOG)

# file log
add_library(MINOR_GEMS_FILE_LOG STATIC "${ROOT_PATH}/minorGems/util/log/FileLog.cpp")
target_link_libraries(MINOR_GEMS_FILE_LOG PRIVATE MINOR_GEMS_BASE)

# print log
add_library(MINOR_GEMS_PRINT_LOG STATIC "${ROOT_PATH}/minorGems/util/log/PrintLog.cpp")
target_link_libraries(MINOR_GEMS_PRINT_LOG PRIVATE MINOR_GEMS_BASE MINOR_GEMS_MUTEX_LOCK)

# print utils
add_library(MINOR_GEMS_PRINT_UTILS STATIC "${ROOT_PATH}/minorGems/util/printUtils.cpp")
target_link_libraries(MINOR_GEMS_PRINT_UTILS PRIVATE MINOR_GEMS_BASE)

# game sdl
add_library(MINOR_GEMS_GAME_SDL STATIC "${ROOT_PATH}/minorGems/game/platforms/SDL/gameSDL.cpp")
target_link_libraries(MINOR_GEMS_GAME_SDL PRIVATE
	MINOR_GEMS_BASE
	MINOR_GEMS_BYTE_BUFFER_INPUT_STREAM
	MINOR_GEMS_FILE_LOG
	MINOR_GEMS_SCREEN_GL_SDL
	MINOR_GEMS_SOCKET_CLIENT
	MINOR_GEMS_STOP_SIGNAL_THREAD
	MINOR_GEMS_TRANSLATION_MANAGER
	MINOR_GEMS_WEB_REQUEST
)

# game graphics gl
add_library(MINOR_GEMS_GAME_GRAPHICS_GL STATIC "${ROOT_PATH}/minorGems/game/platforms/openGL/gameGraphicsGL.cpp")
target_link_libraries(MINOR_GEMS_GAME_GRAPHICS_GL PRIVATE MINOR_GEMS_BASE)

# sprite gl
add_library(MINOR_GEMS_SPRITE_GL STATIC "${ROOT_PATH}/minorGems/game/platforms/openGL/SpriteGL.cpp")
target_link_libraries(MINOR_GEMS_SPRITE_GL PRIVATE
	MINOR_GEMS_BASE
	MINOR_GEMS_SINGLE_TEXTURE_GL
)

# double pair
add_library(MINOR_GEMS_DOUBLE_PAIR STATIC "${ROOT_PATH}/minorGems/game/doublePair.cpp")
target_link_libraries(MINOR_GEMS_DOUBLE_PAIR PRIVATE MINOR_GEMS_BASE)

# font
add_library(MINOR_GEMS_FONT STATIC "${ROOT_PATH}/minorGems/game/Font.cpp")
target_link_libraries(MINOR_GEMS_FONT PRIVATE MINOR_GEMS_BASE)

# draw utils
add_library(MINOR_GEMS_DRAW_UTILS STATIC "${ROOT_PATH}/minorGems/game/drawUtils.cpp")
target_link_libraries(MINOR_GEMS_DRAW_UTILS PRIVATE MINOR_GEMS_BASE)

# demo code checker
add_library(MINOR_GEMS_DEMO_CODE_CHECKER STATIC "${ROOT_PATH}/minorGems/game/platforms/SDL/DemoCodeChecker.cpp")
target_link_libraries(MINOR_GEMS_DEMO_CODE_CHECKER PRIVATE MINOR_GEMS_BASE)

# aiff
add_library(MINOR_GEMS_AIFF STATIC "${ROOT_PATH}/minorGems/sound/formats/aiff.cpp")
target_link_libraries(MINOR_GEMS_AIFF PRIVATE MINOR_GEMS_BASE)

# audio no clip
add_library(MINOR_GEMS_AUDIO_NO_CLIP STATIC "${ROOT_PATH}/minorGems/sound/audioNoClip.cpp")
target_link_libraries(MINOR_GEMS_AUDIO_NO_CLIP PRIVATE MINOR_GEMS_BASE)

# sound samples
add_library(MINOR_GEMS_SOUND_SAMPLES STATIC "${ROOT_PATH}/minorGems/sound/filters/SoundSamples.cpp")
target_link_libraries(MINOR_GEMS_SOUND_SAMPLES PRIVATE MINOR_GEMS_BASE)

# reverb sound filter
add_library(MINOR_GEMS_REVERB_SOUND_FILTER STATIC "${ROOT_PATH}/minorGems/sound/filters/ReverbSoundFilter.cpp")
target_link_libraries(MINOR_GEMS_REVERB_SOUND_FILTER PRIVATE MINOR_GEMS_BASE)

# coefficient filters
add_library(MINOR_GEMS_COEFFICIENT_FILTERS STATIC "${ROOT_PATH}/minorGems/sound/filters/coefficientFilters.cpp")
target_link_libraries(MINOR_GEMS_COEFFICIENT_FILTERS PRIVATE MINOR_GEMS_BASE)

# curve 25519
add_library(MINOR_GEMS_CURVE_25519 STATIC "${ROOT_PATH}/minorGems/crypto/keyExchange/curve25519.cpp")
target_link_libraries(MINOR_GEMS_CURVE_25519 PRIVATE MINOR_GEMS_BASE)

# crypto random
add_library(MINOR_GEMS_CRYPTO_RANDOM STATIC "${ROOT_PATH}/minorGems/crypto/cryptoRandom.cpp")
target_link_libraries(MINOR_GEMS_CRYPTO_RANDOM PRIVATE MINOR_GEMS_BASE)

# diff bundle client
add_library(MINOR_GEMS_DIFF_BUNDLE_CLIENT STATIC "${ROOT_PATH}/minorGems/game/diffBundle/client/diffBundleClient.cpp")
target_link_libraries(MINOR_GEMS_DIFF_BUNDLE_CLIENT PRIVATE MINOR_GEMS_BASE)

# string tree
add_library(MINOR_GEMS_STRING_TREE STATIC "${ROOT_PATH}/minorGems/util/StringTree.cpp")
target_link_libraries(MINOR_GEMS_STRING_TREE PRIVATE MINOR_GEMS_BASE)

# crc32
add_library(MINOR_GEMS_CRC32 STATIC "${ROOT_PATH}/minorGems/util/crc32.cpp")
target_link_libraries(MINOR_GEMS_CRC32 PRIVATE MINOR_GEMS_BASE)

if(CMAKE_SYSTEM MATCHES Linux)
	target_compile_definitions(MINOR_GEMS_BASE INTERFACE LINUX)

	find_package(X11 REQUIRED)
	target_link_libraries(MINOR_GEMS_GRAPHICS_LIBS INTERFACE X11::X11)
elseif(CMAKE_SYSTEM MATCHES Windows)
	target_compile_definitions(MINOR_GEMS_BASE INTERFACE WIN_32 STATICLIB)

	target_link_libraries(MINOR_GEMS_BASE INTERFACE wsock32 winmm -static)
endif()

