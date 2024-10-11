# Copy Maui to have it available as an external (non-embedded) theme
file(COPY "../../src/greeter/theme/" DESTINATION "${CMAKE_CURRENT_SOURCE_DIR}/maui")

# Copy Main.qml and image resources from upstream Maui to debian-maui
file(COPY "../../src/greeter/theme/Main.qml" DESTINATION "${CMAKE_CURRENT_SOURCE_DIR}/debian-maui")
file(GLOB MAUI_PNGS "../../src/greeter/theme/*.png")
foreach(MAUI_PNG ${MAUI_PNGS})
    file(COPY "${MAUI_PNG}" DESTINATION "${CMAKE_CURRENT_SOURCE_DIR}/debian-maui")
endforeach(MAUI_PNG)
# Replace default background source by config key config.background
set(SED_EXPR "s=^\\([[:space:]]*source[[:space:]]*:[[:space:]]\\)\\\"qrc:/theme/background.png\\\"[[:space:]]*$=\\1config.background=")
execute_process(COMMAND sed -i -e "${SED_EXPR}" "${CMAKE_CURRENT_SOURCE_DIR}/debian-maui/Main.qml" RESULT_VARIABLE RES OUTPUT_VARIABLE OUT)
message("Result of sed replacement for Maui's background source: (Res: ${RES}, Out: ${OUT})")

# Copy selected files from upstream Elarun to debian-elarun
file(COPY "elarun/Main.qml" DESTINATION "${CMAKE_CURRENT_SOURCE_DIR}/debian-elarun")
file(COPY "elarun/angle-down.png" DESTINATION "${CMAKE_CURRENT_SOURCE_DIR}/debian-elarun")
file(COPY "elarun/elarun.jpg" DESTINATION "${CMAKE_CURRENT_SOURCE_DIR}/debian-elarun")
file(GLOB ELARUN_IMGS "elarun/images/*")
foreach(ELARUN_IMG ${ELARUN_IMGS})
    file(COPY "${ELARUN_IMG}" DESTINATION "${CMAKE_CURRENT_SOURCE_DIR}/debian-elarun/images/")
endforeach(ELARUN_IMG)

set(THEMES elarun maldives maya maui debian-maui debian-elarun)

foreach(THEME ${THEMES})
    # We assume that theme-specific translations are stored
    # in ${THEME}/translations directory
    set(TRANSLATIONS_DIR "${THEME}/translations")
    file(GLOB TRANSLATION_SOURCES "${TRANSLATIONS_DIR}/*.ts")

    set_source_files_properties(${TRANSLATION_SOURCES} PROPERTIES OUTPUT_LOCATION "${CMAKE_CURRENT_BINARY_DIR}/${TRANSLATIONS_DIR}")

    qt_add_translation(QM_FILES "${TRANSLATION_SOURCES}")

    configure_file("${THEME}/metadata.desktop.in" "${THEME}/metadata.desktop" @ONLY)

    install(DIRECTORY "${THEME}" DESTINATION "${DATA_INSTALL_DIR}/themes"
            PATTERN "${THEME}/*.in" EXCLUDE
            PATTERN "${THEME}/*.ts" EXCLUDE
            PATTERN "${THEME}/.gitattributes" EXCLUDE)

    install(FILES "${CMAKE_CURRENT_BINARY_DIR}/${THEME}/metadata.desktop"
            DESTINATION "${DATA_INSTALL_DIR}/themes/${THEME}/")

    list(APPEND THEMES_QM_FILES ${QM_FILES})
endforeach(THEME)

add_custom_target(themes-translation DEPENDS ${THEMES_QM_FILES})
