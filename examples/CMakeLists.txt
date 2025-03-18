file(GLOB SOURCES "*.c")

foreach(source_file IN LISTS SOURCES)
  # Extract the base name of the file (without extension)
  get_filename_component(executable_name ${source_file} NAME_WE)

  # Add an executable for each .c file
  add_executable(${executable_name} ${source_file})
  target_link_libraries(${executable_name} PRIVATE uSockets)
endforeach()
