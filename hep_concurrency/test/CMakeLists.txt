include(CetTest)
cet_enable_asserts()

find_package(Catch2 REQUIRED)

# Put everything in a different export set.
cet_register_export_set(SET_NAME Testing
  NAMESPACE hep_concurrency_test SET_DEFAULT)

# Test script executes exec.
cet_make_exec(NAME ThreadSafeOutputFileStream_t NO_INSTALL
  LIBRARIES PRIVATE hep_concurrency::hep_concurrency Threads::Threads)
cet_test(runThreadSafeOutputFileStream_t.sh PREBUILT)

cet_test(assert_only_one_thread_test
  LIBRARIES PRIVATE
    simultaneous_function_spawner
    hep_concurrency::macros
    Threads::Threads
  TEST_PROPERTIES
    PASS_REGULAR_EXPRESSION
      "Failed assert--more than one thread accessing location"
)

foreach(test IN ITEMS
    serial_task_queue_chain_t
    serial_task_queue_t
    waiting_task_list_t
)
  cet_test(${test} USE_CATCH2_MAIN
    LIBRARIES PRIVATE hep_concurrency::hep_concurrency)
endforeach()

target_link_libraries(waiting_task_list_t PRIVATE TBB::tbb)

cet_test(simultaneous_function_spawner_t USE_CATCH2_MAIN
  LIBRARIES PRIVATE hep_concurrency::simultaneous_function_spawner
)

########################################################################
# Test concurrent caching facility.
########################################################################

foreach (target IN ITEMS cache_handle_t cache_mt_t cache_t)
  cet_test(${target} USE_CATCH2_MAIN
    LIBRARIES PRIVATE hep_concurrency::cache TBB::tbb)
endforeach()

target_link_libraries(cache_handle_t PRIVATE cetlib_except::Catch2Matchers)
target_link_libraries(cache_t PRIVATE cetlib_except::Catch2Matchers)

cet_test(bad_conversion_task_t COMPILE_ONLY TEST_PROPERTIES WILL_FAIL TRUE)
cet_test(chain_bad_conversion_task_t COMPILE_ONLY TEST_PROPERTIES WILL_FAIL TRUE)
cet_test(bad_waiting_task COMPILE_ONLY TEST_PROPERTIES WILL_FAIL TRUE)
foreach (target IN ITEMS bad_cache_no_eq_t bad_cache_hash_t
          bad_entry_for_t bad_entry_for_hint_t)
  cet_test(${target} COMPILE_ONLY
    LIBRARIES PRIVATE hep_concurrency::cache
    TEST_PROPERTIES WILL_FAIL TRUE)
endforeach()


cet_test(bad_sanitize_t COMPILE_ONLY TEST_PROPERTIES WILL_FAIL TRUE)
