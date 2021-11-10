find_package(imgui REQUIRED)
add_library(ui STATIC uiMainDlg.cpp uiSettings.cpp)
target_link_libraries(ui PUBLIC imgui::imgui)
target_include_directories(ui PUBLIC ${imgui_RES_DIRS}/bindings)
