
if (DOXYGEN_FOUND)
if (LLVM_ENABLE_DOXYGEN)
  set(abs_top_srcdir ${LLVM_MAIN_SRC_DIR})
  set(abs_top_builddir ${LLVM_BINARY_DIR})
  
  if (HAVE_DOT)
    set(DOT ${LLVM_PATH_DOT})
  endif()

  if (LLVM_DOXYGEN_EXTERNAL_SEARCH)
    set(enable_searchengine "YES")
    set(searchengine_url "${LLVM_DOXYGEN_SEARCHENGINE_URL}")
    set(enable_server_based_search "YES")
    set(enable_external_search "YES")
    set(extra_search_mappings "${LLVM_DOXYGEN_SEARCH_MAPPINGS}")
  else()
    set(enable_searchengine "NO")
    set(searchengine_url "")
    set(enable_server_based_search "NO")
    set(enable_external_search "NO")
    set(extra_search_mappings "")
  endif()
  
  # If asked, configure doxygen for the creation of a Qt Compressed Help file.
  option(LLVM_ENABLE_DOXYGEN_QT_HELP
    "Generate a Qt Compressed Help file." OFF)
  if (LLVM_ENABLE_DOXYGEN_QT_HELP)
    set(LLVM_DOXYGEN_QCH_FILENAME "org.llvm.qch" CACHE STRING
      "Filename of the Qt Compressed help file")
    set(LLVM_DOXYGEN_QHP_NAMESPACE "org.llvm" CACHE STRING 
      "Namespace under which the intermediate Qt Help Project file lives")
    set(LLVM_DOXYGEN_QHP_CUST_FILTER_NAME "${PACKAGE_STRING}" CACHE STRING
      "See http://qt-project.org/doc/qt-4.8/qthelpproject.html#custom-filters")
    set(LLVM_DOXYGEN_QHP_CUST_FILTER_ATTRS "${PACKAGE_NAME},${PACKAGE_VERSION}" CACHE STRING
      "See http://qt-project.org/doc/qt-4.8/qthelpproject.html#filter-attributes")
    find_program(LLVM_DOXYGEN_QHELPGENERATOR_PATH qhelpgenerator 
      DOC "Path to the qhelpgenerator binary")
    if (NOT LLVM_DOXYGEN_QHELPGENERATOR_PATH)
      message(FATAL_ERROR "Failed to find qhelpgenerator binary")
    endif()

    set(llvm_doxygen_generate_qhp "YES")
    set(llvm_doxygen_qch_filename "${LLVM_DOXYGEN_QCH_FILENAME}")
    set(llvm_doxygen_qhp_namespace "${LLVM_DOXYGEN_QHP_NAMESPACE}")
    set(llvm_doxygen_qhelpgenerator_path "${LLVM_DOXYGEN_QHELPGENERATOR_PATH}")
    set(llvm_doxygen_qhp_cust_filter_name "${LLVM_DOXYGEN_QHP_CUST_FILTER_NAME}")
    set(llvm_doxygen_qhp_cust_filter_attrs "${LLVM_DOXYGEN_QHP_CUST_FILTER_ATTRS}")

  else()
    set(llvm_doxygen_generate_qhp "NO")
    set(llvm_doxygen_qch_filename "")
    set(llvm_doxygen_qhp_namespace "")
    set(llvm_doxygen_qhelpgenerator_path "")
    set(llvm_doxygen_qhp_cust_filter_name "")
    set(llvm_doxygen_qhp_cust_filter_attrs "")
  endif()
  
  configure_file(${CMAKE_CURRENT_SOURCE_DIR}/doxygen.cfg.in
    ${CMAKE_CURRENT_BINARY_DIR}/doxygen.cfg @ONLY)

  set(abs_top_srcdir)
  set(abs_top_builddir)
  set(DOT)
  set(enable_searchengine)
  set(searchengine_url)
  set(enable_server_based_search)
  set(enable_external_search)
  set(extra_search_mappings)
  set(llvm_doxygen_generate_qhp)
  set(llvm_doxygen_qch_filename)
  set(llvm_doxygen_qhp_namespace)
  set(llvm_doxygen_qhelpgenerator_path)
  set(llvm_doxygen_qhp_cust_filter_name)
  set(llvm_doxygen_qhp_cust_filter_attrs)

  add_custom_target(doxygen-llvm
    COMMAND ${DOXYGEN_EXECUTABLE} ${CMAKE_CURRENT_BINARY_DIR}/doxygen.cfg
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
    COMMENT "Generating llvm doxygen documentation." VERBATIM)

  if (LLVM_BUILD_DOCS)
    add_dependencies(doxygen doxygen-llvm)
  endif()

  if (NOT LLVM_INSTALL_TOOLCHAIN_ONLY)
    install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/doxygen/html
      DESTINATION docs/html)
  endif()
endif()
endif()

if (LLVM_ENABLE_SPHINX)
  if (SPHINX_FOUND)
    include(AddSphinxTarget)
    if (${SPHINX_OUTPUT_HTML})
      add_sphinx_target(html llvm)
    endif()


    if (${SPHINX_OUTPUT_MAN})
      add_sphinx_target(man llvm)
    endif()

  endif()
endif()

list(FIND LLVM_BINDINGS_LIST ocaml uses_ocaml)
if( NOT uses_ocaml LESS 0 )
  set(doc_targets
        ocaml_llvm
        ocaml_llvm_all_backends
        ocaml_llvm_analysis
        ocaml_llvm_bitreader
        ocaml_llvm_bitwriter
        ocaml_llvm_executionengine
        ocaml_llvm_irreader
        ocaml_llvm_linker
        ocaml_llvm_target
        ocaml_llvm_ipo
        ocaml_llvm_passmgr_builder
        ocaml_llvm_scalar_opts
        ocaml_llvm_transform_utils
        ocaml_llvm_vectorize
      )

  foreach(llvm_target ${LLVM_TARGETS_TO_BUILD})
    list(APPEND doc_targets ocaml_llvm_${llvm_target})
  endforeach()

  set(odoc_files)
  foreach( doc_target ${doc_targets} )
    get_target_property(odoc_file ${doc_target} OCAML_ODOC)
    list(APPEND odoc_files -load ${odoc_file})
  endforeach()

  add_custom_target(ocaml_doc
    COMMAND ${CMAKE_COMMAND} -E remove_directory ${CMAKE_CURRENT_BINARY_DIR}/ocamldoc/html
    COMMAND ${CMAKE_COMMAND} -E make_directory ${CMAKE_CURRENT_BINARY_DIR}/ocamldoc/html
    COMMAND ${OCAMLFIND} ocamldoc -d ${CMAKE_CURRENT_BINARY_DIR}/ocamldoc/html
                                  -sort -colorize-code -html ${odoc_files}
    COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_CURRENT_SOURCE_DIR}/_ocamldoc/style.css
                                     ${CMAKE_CURRENT_BINARY_DIR}/ocamldoc/html)

  add_dependencies(ocaml_doc ${doc_targets})

  if (NOT LLVM_INSTALL_TOOLCHAIN_ONLY)
    install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/ocamldoc/html
      DESTINATION docs/ocaml/html)
  endif()
endif()
