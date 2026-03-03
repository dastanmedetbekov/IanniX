cmake_minimum_required(VERSION 3.16)
project(IanniX VERSION 0.9.20 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_AUTOUIC ON)

find_package(Qt5 REQUIRED COMPONENTS Core Gui Widgets OpenGL Network Script Xml)

if(UNIX AND NOT APPLE)
    find_package(ALSA REQUIRED)
    find_package(PkgConfig REQUIRED)
    pkg_check_modules(JACK jack)
endif()

set(SOURCES
    iannix.cpp
    iannixapp.cpp
    iannix_spec.cpp
    misc/help.cpp
    misc/application.cpp
    misc/options.cpp
    misc/applicationexecute.cpp
    messages/messagemanagerlogmini.cpp
    messages/messagemanagerlog.cpp
    messages/messagemanager.cpp
    messages/message.cpp
    transport/transport.cpp
    transport/uitimer.cpp
    transport/uiabout.cpp
    transport/uieditor.cpp
    render/uirender.cpp
    render/uirenderpreview.cpp
    geometry/nxpoint.cpp
    geometry/nxrect.cpp
    geometry/nxsize.cpp
    geometry/nxline.cpp
    geometry/nxpolygon.cpp
    geometry/nxeasing.cpp
    geometry/qmuparser/muParser.cpp
    geometry/qmuparser/muParserBase.cpp
    geometry/qmuparser/muParserBytecode.cpp
    geometry/qmuparser/muParserCallback.cpp
    geometry/qmuparser/muParserError.cpp
    geometry/qmuparser/muParserTokenReader.cpp
    objects/nxdocument.cpp
    objects/nxtrigger.cpp
    objects/nxgroup.cpp
    objects/nxcurve.cpp
    objects/nxcursor.cpp
    objects/nxobject.cpp
    gui/uiinspector.cpp
    gui/uiview.cpp
    gui/uihelp.cpp
    gui/uimessagebox.cpp
    gui/uisplashscreen.cpp
    gui/qjsedit/jsedit.cpp
    gui/uitreeview.cpp
    gui/uitreeviewwidget.cpp
    gui/uitreedelegate.cpp
    gui/uifileitem.cpp
    gui/uicoloritem.cpp
    gui/uipathpointsitem.cpp
    gui/uitextureitem.cpp
    interfaces/extscriptvariableask.cpp
    interfaces/extoscpatternask.cpp
    interfaces/extoscpatterneditor.cpp
    render/abstractionsgl.cpp
    interfaces/interfacehttp.cpp
    interfaces/interfacemidi.cpp
    interfaces/interfaceosc.cpp
    interfaces/interfaceserial.cpp
    interfaces/interfacetcp.cpp
    interfaces/interfaceudp.cpp
    interfaces/interfacedirect.cpp
    interfaces/qextserialport/qextserialport.cpp
    interfaces/qextserialport/qextserialenumerator.cpp
    interfaces/qrtmidi/RtMidi.cpp
    interfaces/qhttpserver/websocket.cpp
    interfaces/qhttpserver/websocketserver.cpp
    interfaces/qhttpserver/websocketprotocol.cpp
    interfaces/qhttpserver/handshakerequest.cpp
    interfaces/qhttpserver/handshakeresponse.cpp
    interfaces/qhttpserver/dataprocessor.cpp
)

set(HEADERS
    iannix.h
    iannixapp.h
    iannix_spec.h
    iannix_cmd.h
    abstractionsgl.h
)

set(UI_FILES
    messages/messagemanagerlogmini.ui
    messages/messagemanagerlog.ui
    transport/uitimer.ui
    transport/uiabout.ui
    transport/uieditor.ui
    render/uirender.ui
    gui/uiinspector.ui
    gui/uiview.ui
    gui/uihelp.ui
    gui/uimessagebox.ui
    interfaces/extscriptvariableask.ui
    interfaces/extoscpatternask.ui
    interfaces/extoscpatterneditor.ui
    interfaces/interfaceserial.ui
    interfaces/interfaceudp.ui
    interfaces/interfacetcp.ui
    interfaces/interfacehttp.ui
    interfaces/interfaceosc.ui
    interfaces/interfacemidi.ui
    interfaces/interfacedirect.ui
    interfaces/interfacesyphon.ui
)

set(RESOURCES
    icons/IanniX.qrc
)

if(UNIX AND NOT APPLE)
    list(APPEND SOURCES
        interfaces/qextserialport/qextserialport_unix.cpp
        interfaces/qextserialport/qextserialenumerator_linux.cpp
    )
endif()

if(WIN32)
    list(APPEND SOURCES
        interfaces/qextserialport/qextserialport_win.cpp
        interfaces/qextserialport/qextserialenumerator_win.cpp
    )
endif()

if(APPLE)
    list(APPEND SOURCES
        interfaces/qextserialport/qextserialenumerator_osx.cpp
    )
endif()

add_executable(iannix ${SOURCES} ${HEADERS} ${UI_FILES} ${RESOURCES})

target_include_directories(iannix PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}/geometry
    ${CMAKE_CURRENT_SOURCE_DIR}/geometry/qmuparser
    ${CMAKE_CURRENT_SOURCE_DIR}/gui
    ${CMAKE_CURRENT_SOURCE_DIR}/gui/qjsedit
    ${CMAKE_CURRENT_SOURCE_DIR}/interfaces
    ${CMAKE_CURRENT_SOURCE_DIR}/interfaces/qextserialport
    ${CMAKE_CURRENT_SOURCE_DIR}/interfaces/qrtmidi
    ${CMAKE_CURRENT_SOURCE_DIR}/interfaces/qhttpserver
    ${CMAKE_CURRENT_SOURCE_DIR}/messages
    ${CMAKE_CURRENT_SOURCE_DIR}/misc
    ${CMAKE_CURRENT_SOURCE_DIR}/objects
    ${CMAKE_CURRENT_SOURCE_DIR}/render
    ${CMAKE_CURRENT_SOURCE_DIR}/transport
)

target_link_libraries(iannix
    Qt5::Core
    Qt5::Gui
    Qt5::Widgets
    Qt5::OpenGL
    Qt5::Network
    Qt5::Script
    Qt5::Xml
)

target_compile_definitions(iannix PRIVATE
    QT5
    IANNIX_64
    IANNIX_32
    __RTMIDI_DEBUG__
)

if(UNIX AND NOT APPLE)
    target_compile_definitions(iannix PRIVATE
        __LINUX_ALSASEQ__
        __LINUX_ALSA__
        __UNIX_JACK__
        AVOID_TIMESTAMPING
        __linux__
        QESP_NO_UDEV
    )
    target_link_libraries(iannix ${ALSA_LIBRARIES})
    if(JACK_FOUND)
        target_link_libraries(iannix ${JACK_LIBRARIES})
        target_include_directories(iannix PRIVATE ${JACK_INCLUDE_DIRS})
    endif()
endif()

if(WIN32)
    target_compile_definitions(iannix PRIVATE __WINDOWS_MM__)
    target_link_libraries(iannix winmm setupapi advapi32 user32 opengl32)
endif()

if(APPLE)
    target_compile_definitions(iannix PRIVATE __MACOSX_CORE__)
    target_link_libraries(iannix
        "-framework CoreMidi"
        "-framework CoreAudio"
        "-framework CoreFoundation"
        "-framework IOKit"
        "-framework Carbon"
    )
endif()

if(UNIX AND NOT APPLE)
    install(TARGETS iannix DESTINATION bin)
    install(FILES iannix.desktop DESTINATION share/applications)
    install(FILES iannix.png DESTINATION share/pixmaps)
endif()
