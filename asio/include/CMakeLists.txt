# Install headers
file(GLOB HEADERS *.hpp *.ipp)
install(FILES ${HEADERS}
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/asio)