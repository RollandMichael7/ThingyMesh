set(header_globbing_expression
    "${CMAKE_SOURCE_DIR}/mesh/core/api/*.h"
    "${CMAKE_SOURCE_DIR}/mesh/core/include/*.h"
    "${CMAKE_SOURCE_DIR}/mesh/access/api/*.h"
    "${CMAKE_SOURCE_DIR}/mesh/access/include/*.h"
    "${CMAKE_SOURCE_DIR}/mesh/bearer/include/*.h"
    "${CMAKE_SOURCE_DIR}/mesh/bearer/api/*.h"
    "${CMAKE_SOURCE_DIR}/mesh/prov/api/*.h"
    "${CMAKE_SOURCE_DIR}/mesh/prov/include/*.h"
    "${CMAKE_SOURCE_DIR}/mesh/dfu/api/*.h"
    "${CMAKE_SOURCE_DIR}/mesh/dfu/include/*.h"
    "${CMAKE_SOURCE_DIR}/mesh/serial/api/*.h"
    "${CMAKE_SOURCE_DIR}/mesh/serial/include/*.h"
    "${CMAKE_SOURCE_DIR}/mesh/test/include/device_wrappers/*.h"
    "${CMAKE_SOURCE_DIR}/models/config/include/*.h"
    "${CMAKE_SOURCE_DIR}/models/health/include/*.h"
    "${CMAKE_SOURCE_DIR}/models/remote_provisioning/include/*.h"
    "${CMAKE_SOURCE_DIR}/external/micro-ecc/*.h")

set(header_exclude_regex "nrf_mesh_hw")
set(CMOCK_SETTINGS_FILE_IN "${CMAKE_CURRENT_SOURCE_DIR}/cmock_config.yaml.in")
set(CMOCK_SETTINGS_FILE "${CMAKE_CURRENT_BINARY_DIR}/cmock_config.yaml")
configure_file(${CMOCK_SETTINGS_FILE_IN} ${CMOCK_SETTINGS_FILE})
generate_mock_targets("${header_globbing_expression}" "${header_exclude_regex}")

set(include_directories
    "${CMAKE_SOURCE_DIR}/mesh/core/api"
    "${CMAKE_SOURCE_DIR}/mesh/core/include"
    "${CMAKE_SOURCE_DIR}/mesh/access/api"
    "${CMAKE_SOURCE_DIR}/mesh/access/include"
    "${CMAKE_SOURCE_DIR}/mesh/bearer/api"
    "${CMAKE_SOURCE_DIR}/mesh/bearer/include"
    "${CMAKE_SOURCE_DIR}/mesh/prov/api"
    "${CMAKE_SOURCE_DIR}/mesh/prov/include"
    "${CMAKE_SOURCE_DIR}/mesh/dfu/api"
    "${CMAKE_SOURCE_DIR}/mesh/dfu/include"
    "${CMAKE_SOURCE_DIR}/mesh/serial/api"
    "${CMAKE_SOURCE_DIR}/mesh/serial/include"
    "${CMAKE_SOURCE_DIR}/mesh/test/include"
    "${CMAKE_SOURCE_DIR}/mesh/test/include/device_wrappers"
    "${CMAKE_SOURCE_DIR}/models/config/include"
    "${CMAKE_SOURCE_DIR}/models/remote_provisioning/include"
    "${CMAKE_SOURCE_DIR}/models/health/include"
    "${CMAKE_SOURCE_DIR}/external/micro-ecc"
    "${SDK_ROOT}/components/libraries/util"
    "${CMOCK_BIN}"
    "${CMOCK_ROOT}/src"
    "${UNITY_ROOT}/src"
    ${${PLATFORM}_INCLUDE_DIRS}
    ${${SOFTDEVICE}_INCLUDE_DIRS})

set(compile_options
    ${${PLATFORM}_DEFINES}
    "-DNRF_MESH_LOG_ENABLE=1"
    "-DLOG_CALLBACK_DEFAULT=log_callback_stdout"
    "-DUNIT_TEST=1"
    "-DINTERNAL_EVT_ENABLE=0")

target_sources(unit_test_common PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}/src/test_assert.c)

target_include_directories(unit_test_common PUBLIC ${include_directories})
target_compile_options(unit_test_common PUBLIC ${compile_options})

add_subdirectory(mttest)

set(packet_mgr_mtt_srcs
    src/mtt_packet_mgr.c
    ${CMAKE_CURRENT_SOURCE_DIR}/../core/src/packet_mgr.c
    ${CMAKE_CURRENT_SOURCE_DIR}/../core/src/toolchain.c
    ${CMAKE_CURRENT_SOURCE_DIR}/../core/src/log.c)
add_mtt_test(mtt_packet_mgr "${packet_mgr_mtt_srcs}" "${include_directories}"
    "${${PLATFORM}_DEFINES};-DNRF_MESH_LOG_ENABLE=1;;-DLOG_CALLBACK_DEFAULT=log_callback_stdout;-DMTT_TEST=1")

# Transport Layer - transport
set(transport_test_srcs
    src/ut_transport.c
    ../core/src/transport.c
    ../core/src/rand.c
    ../core/src/toolchain.c
    ../core/src/log.c
    ../core/src/nrf_mesh_utils.c
    ${CMOCK_BIN}/bearer_event_mock.c
    ${CMOCK_BIN}/network_mock.c
    ${CMOCK_BIN}/nrf_mesh_mock.c
    ${CMOCK_BIN}/enc_mock.c
    ${CMOCK_BIN}/timer_mock.c
    ${CMOCK_BIN}/timer_scheduler_mock.c
    ${CMOCK_BIN}/event_mock.c
    ${CMOCK_BIN}/replay_cache_mock.c
    ${CMOCK_BIN}/nrf_mesh_externs_mock.c
    ${CMOCK_BIN}/core_tx_mock.c
    ${CMOCK_BIN}/net_state_mock.c
    )
add_unit_test(transport "${transport_test_srcs}" "${include_directories}" "${compile_options}")

# Network Layer - network
set(network_test_srcs
    src/ut_network.c
    ../core/src/network.c
    ../core/src/toolchain.c
    ../core/src/log.c
    ${CMOCK_BIN}/msg_cache_mock.c
    ${CMOCK_BIN}/transport_mock.c
    ${CMOCK_BIN}/net_state_mock.c
    ${CMOCK_BIN}/net_beacon_mock.c
    ${CMOCK_BIN}/core_tx_mock.c
    ${CMOCK_BIN}/core_tx_adv_mock.c
    ${CMOCK_BIN}/enc_mock.c
    ${CMOCK_BIN}/heartbeat_mock.c
    ${CMOCK_BIN}/nrf_mesh_externs_mock.c
    )
add_unit_test(network "${network_test_srcs}" "${include_directories}" "${compile_options}")


# Network Layer - network vectors
set(network_vectors_test_srcs
    src/ut_network_vectors.c
    src/aes_soft.c
    ../core/src/network.c
    ../core/src/toolchain.c
    ../core/src/log.c
    ../core/src/enc.c
    ../core/src/ccm_soft.c
    ../core/src/aes_cmac.c
    ../core/src/rand.c
    ../core/src/nrf_mesh_utils.c
    ${CMOCK_BIN}/msg_cache_mock.c
    ${CMOCK_BIN}/transport_mock.c
    ${CMOCK_BIN}/net_state_mock.c
    ${CMOCK_BIN}/net_beacon_mock.c
    ${CMOCK_BIN}/core_tx_mock.c
    ${CMOCK_BIN}/core_tx_adv_mock.c
    ${CMOCK_BIN}/heartbeat_mock.c
    )
add_unit_test(network_vectors "${network_vectors_test_srcs}" "${include_directories}" "${compile_options}")

# Timer - timer
set(timer_test_srcs
    src/ut_timer.c
    ../core/src/timer.c
    ../core/src/toolchain.c
    )
add_unit_test(timer "${timer_test_srcs}" "${include_directories}" "${compile_options};-DNRF51")

# Message Cache - msg_cache
set(msg_cache_test_srcs
    src/ut_msg_cache.c
    ../core/src/msg_cache.c
    ../core/src/toolchain.c
    )
add_unit_test(msg_cache "${msg_cache_test_srcs}" "${include_directories}" "${compile_options}")

# Packet Module - packet
set(packet_test_srcs
    src/ut_packet.c
    ../core/src/toolchain.c
    )
add_unit_test(packet "${packet_test_srcs}" "${include_directories}" "${compile_options}")

# Radio config
set(radio_config_test_srcs
    src/ut_radio_config.c
    ../bearer/src/radio_config.c
    )
add_unit_test(radio_config "${radio_config_test_srcs}" "${include_directories}" "${compile_options};-DNRF51")

# Timer scheduler
set(timer_sch_test_srcs
    src/ut_timer_scheduler.c
    ../core/src/timer_scheduler.c
    ../core/src/fifo.c
    ../core/src/toolchain.c
    )
add_unit_test(timer_scheduler "${timer_sch_test_srcs}" "${include_directories}" "${compile_options}")

# Packet Manager - packet_mgr
set(packet_mgr_test_srcs
    src/ut_packet_mgr.c
    ../core/src/packet_mgr.c
    ../core/src/toolchain.c
    ../core/src/log.c
    )
add_unit_test(packet_mgr "${packet_mgr_test_srcs}" "${include_directories}" "${compile_options};-DPACKET_MGR_DEBUG_MODE=1")

# Packet Buffer - packet_buffer - DISABLED, PENDING MBTLE-1971
# set(packet_buffer_test_srcs
#     src/ut_packet_buffer.c
#     ../core/src/packet_buffer.c
#     ../core/src/toolchain.c
#     ../core/src/log.c
#     )
# add_unit_test(packet_buffer "${packet_buffer_test_srcs}" "${include_directories}" "${compile_options};-DPACKET_BUFFER_DEBUG_MODE=1")

# CCM Software implementation - ccm_soft
set(ccm_soft_test_srcs
    src/ut_ccm_soft.c
    src/aes_soft.c
    ../core/src/ccm_soft.c
    ../core/src/log.c
    )
add_unit_test(ccm_soft "${ccm_soft_test_srcs}" "${include_directories}" "${compile_options}")

# AES-CMAC - aes_cmac
set(aes_cmac_test_srcs
    src/ut_aes_cmac.c
    ../core/src/aes_cmac.c
    src/aes_soft.c
    ../core/src/toolchain.c
    ../core/src/log.c
    )
add_unit_test(aes_cmac "${aes_cmac_test_srcs}" "${include_directories}" "${compile_options}")

# Timeslot
set(timeslot_test_srcs
  src/ut_timeslot.c
  ../core/src/timeslot.c
  ${CMOCK_BIN}/timer_mock.c
  ${CMOCK_BIN}/bearer_handler_mock.c
  ${CMOCK_BIN}/nrf_mesh_cmsis_mock_mock.c
  )
add_unit_test(timeslot "${timeslot_test_srcs}" "${include_directories}" "${compile_options};-DNRF51;-DS130;-DSD_BLE_API_VERSION=2")

# Encryption
set(enc_test_srcs
    src/ut_enc.c
    ../core/src/enc.c
    ../core/src/rand.c
    src/aes_soft.c
    ../core/src/aes_cmac.c
    ../core/src/ccm_soft.c
    ../core/src/toolchain.c
    ../core/src/log.c
    )
add_unit_test(enc "${enc_test_srcs}" "${include_directories}" "${compile_options}")

# Keygen
set(keygen_srcs
    src/ut_keygen.c
    ../core/src/nrf_mesh_keygen.c
    ../core/src/enc.c
    ../core/src/rand.c
    src/aes_soft.c
    ../core/src/ccm_soft.c
    ../core/src/aes_cmac.c
    ../core/src/log.c
    )
add_unit_test(keygen "${keygen_srcs}" "${include_directories}" "${compile_options}")

# Beacon
set(beacon_test_srcs
    src/ut_beacon.c
    ../core/src/beacon.c
    ${CMOCK_BIN}/net_beacon_mock.c
    ${CMOCK_BIN}/prov_beacon_mock.c
    ${CMOCK_BIN}/advertiser_mock.c
    ../core/src/log.c
    )
add_unit_test(beacon "${beacon_test_srcs}" "${include_directories}" "${compile_options}")

# fifo
set(fifo_srcs
    src/ut_fifo.c
    ../core/src/fifo.c
    )
add_unit_test(fifo "${fifo_srcs}" "${include_directories}" "${compile_options}")

# CCM with additional data
set(ccm_ad_srcs
    src/ut_ccm_ad.c
    src/aes_soft.c
    ../core/src/ccm_soft.c
    ../core/src/log.c
    )
add_unit_test(ccm_ad "${ccm_ad_srcs}" "${include_directories}" "${compile_options};-DUSE_ADDITIONAL_DATA=1")

# Provisioning bearer - ADV
set(prov_bearer_adv_test_srcs
    src/ut_prov_bearer_adv.c
    ../prov/src/prov_bearer_adv.c
    ../core/src/log.c
    ${CMOCK_BIN}/timer_mock.c
    ${CMOCK_BIN}/timer_scheduler_mock.c
    ${CMOCK_BIN}/provisioning_mock.c
    ${CMOCK_BIN}/rand_mock.c
    ${CMOCK_BIN}/prov_beacon_mock.c
    ${CMOCK_BIN}/nrf_mesh_mock.c
    ${CMOCK_BIN}/advertiser_mock.c
    ${CMOCK_BIN}/bearer_event_mock.c
    ${CMOCK_BIN}/nrf_mesh_configure_mock.c
    )
add_unit_test(prov_bearer_adv "${prov_bearer_adv_test_srcs}" "${include_directories}" "${compile_options}")

# Provisioning procedure:
set(prov_provisioning_test_srcs
    src/ut_prov_provisioning.c
    ../prov/src/prov_provisioner.c
    ../prov/src/prov_provisionee.c
    ../core/src/log.c
    ${CMOCK_BIN}/provisioning_mock.c
    ${CMOCK_BIN}/prov_utils_mock.c
    ${CMOCK_BIN}/enc_mock.c
    ${CMOCK_BIN}/nrf_mesh_utils_mock.c
    )
add_unit_test(prov_provisioning "${prov_provisioning_test_srcs}" "${include_directories}" "${compile_options}")
target_link_libraries(ut_prov_provisioning)

# Linked list:
set(linked_list_srcs
    ../core/src/list.c
    src/ut_linked_list.c
    )
add_unit_test(linked_list "${linked_list_srcs}" "${include_directories}" "${compile_options}")

# Replay Cache:
set(replay_cache_srcs
    src/ut_replay_cache.c
    ../core/src/replay_cache.c
    )
add_unit_test(replay_cache "${replay_cache_srcs}" "${include_directories}" "${compile_options}")

set(serial_packet_srcs
    src/ut_serial_packet.c
    )
add_unit_test(serial_packet "${serial_packet_srcs}" "${include_directories}" "${compile_options}")

# Serial handler access module unit test
set(serial_handler_access_srcs
    src/ut_serial_handler_access.c
    ../serial/src/serial_handler_access.c
    ../serial/src/serial_handler_common.c
    ${CMOCK_BIN}/serial_mock.c
    ${CMOCK_BIN}/access_config_mock.c
    ${CMOCK_BIN}/access_mock.c
    )
add_unit_test(serial_handler_access "${serial_handler_access_srcs}" "${include_directories}" "${compile_options}")

# Serial handler models module unit test
set(serial_handler_models_srcs
    src/ut_serial_handler_models.c
    ../serial/src/serial_handler_models.c
    ../serial/src/serial_handler_common.c
    ${CMOCK_BIN}/serial_mock.c
    ${CMOCK_BIN}/access_config_mock.c
    )
add_unit_test(serial_handler_models "${serial_handler_models_srcs}" "${include_directories}" "${compile_options};-DACCESS_MODEL_COUNT=2")

# Serial handler for the pb remote client
#set(serial_pb_remote_client_srcs
#    src/ut_serial_pb_remote_client.c
#    ${CMAKE_SOURCE_DIR}/models/pb_remote/src/serial_pb_remote_client.c
#    ${CMOCK_BIN}/serial_handler_models_mock.c
#    ${CMOCK_BIN}/serial_mock.c
#    ${CMOCK_BIN}/pb_remote_client_mock.c
#    ${CMOCK_BIN}/access_config_mock.c
#    )
#add_unit_test(serial_pb_remote_client "${serial_pb_remote_client_srcs}" "${include_directories}" "${compile_options};-DSERIAL_PB_REMOTE_CLIENT_INSTANCE_COUNT=2")

# Serial handler mesh
set(serial_handler_mesh_srcs
    src/ut_serial_handler_mesh.c
    ../serial/src/serial_handler_mesh.c
    ../serial/src/serial_handler_common.c
    ../core/src/log.c
    ${CMOCK_BIN}/nrf_mesh_mock.c
    ${CMOCK_BIN}/nrf_mesh_events_mock.c
    ${CMOCK_BIN}/serial_mock.c
    ${CMOCK_BIN}/access_mock.c
    ${CMOCK_BIN}/net_state_mock.c
    ${CMOCK_BIN}/device_state_manager_mock.c
    )
add_unit_test(serial_handler_mesh "${serial_handler_mesh_srcs}" "${include_directories}" "${compile_options}")

# Serial handler device
set(serial_handler_device_srcs
    src/ut_serial_handler_device.c
    ../serial/src/serial_handler_device.c
    ../serial/src/serial_handler_common.c
    ../core/src/log.c
    ${CMOCK_BIN}/hal_mock.c
    ${CMOCK_BIN}/serial_mock.c
    ${CMOCK_BIN}/advertiser_mock.c
    )
add_unit_test(serial_handler_device "${serial_handler_device_srcs}" "${include_directories}" "${compile_options};-DNRF_MESH_SERIAL_BEACON_SLOTS=3")

# Serial handler DFU
set(serial_handler_dfu_srcs
    src/ut_serial_handler_dfu.c
    ../serial/src/serial_handler_dfu.c
    ${CMOCK_BIN}/serial_mock.c
    ${CMOCK_BIN}/nrf_mesh_dfu_mock.c
    ${CMOCK_BIN}/nrf_mesh_events_mock.c
    )
add_unit_test(serial_handler_dfu "${serial_handler_dfu_srcs}" "${include_directories}" "${compile_options};-DNRF_MESH_DFU_ENABLE")

# Application serial handler
set(serial_handler_app_srcs
    src/ut_serial_handler_app.c
    ../serial/src/serial_handler_app.c
    ${CMOCK_BIN}/serial_mock.c
    )
add_unit_test(serial_handler_app "${serial_handler_app_srcs}" "${include_directories}" "${compile_options}")

# Config serial handler
set(serial_handler_config_srcs
    src/ut_serial_handler_config.c
    ../serial/src/serial_handler_config.c
    ${CMOCK_BIN}/advertiser_mock.c
    ${CMOCK_BIN}/serial_mock.c
    ${CMOCK_BIN}/nrf_mesh_configure_mock.c
    )
add_unit_test(serial_handler_config "${serial_handler_config_srcs}" "${include_directories}" "${compile_options}")

# OpenMesh serial handler
set(serial_handler_openmesh_srcs
    src/ut_serial_handler_openmesh.c
    ../serial/src/serial_handler_openmesh.c
    ${CMOCK_BIN}/serial_mock.c
    ${CMOCK_BIN}/nrf_mesh_dfu_mock.c
    ${CMOCK_BIN}/hal_mock.c
    )
add_unit_test(serial_handler_openmesh "${serial_handler_openmesh_srcs}" "${include_directories}" "${compile_options};-DNRF_MESH_DFU_ENABLE")

# Flash module
set(mesh_flash_srcs
    src/ut_mesh_flash.c
    ../core/src/mesh_flash.c
    ../core/src/msqueue.c
    ${CMOCK_BIN}/nrf_flash_mock.c
    ${CMOCK_BIN}/timer_mock.c
    ${CMOCK_BIN}/bearer_handler_mock.c
    )
add_unit_test(mesh_flash "${mesh_flash_srcs}" "${include_directories}" "${compile_options}")

# nRF Mesh API
set(nrf_mesh_srcs
    src/ut_nrf_mesh.c
    ../core/src/nrf_mesh.c
    ../core/src/log.c
    ../core/src/nrf_mesh_utils.c
    ${CMOCK_BIN}/heartbeat_mock.c
    ${CMOCK_BIN}/toolchain_mock.c
    ${CMOCK_BIN}/timer_scheduler_mock.c
    ${CMOCK_BIN}/transport_mock.c
    ${CMOCK_BIN}/network_mock.c
    ${CMOCK_BIN}/msg_cache_mock.c
    ${CMOCK_BIN}/enc_mock.c
    ${CMOCK_BIN}/rand_mock.c
    ${CMOCK_BIN}/nrf_mesh_dfu_mock.c
    ${CMOCK_BIN}/nrf_mesh_configure_mock.c
    ${CMOCK_BIN}/bearer_event_mock.c
    ${CMOCK_BIN}/beacon_mock.c
    ${CMOCK_BIN}/event_mock.c
    ${CMOCK_BIN}/prov_bearer_adv_mock.c
    ${CMOCK_BIN}/ticker_mock.c
    ${CMOCK_BIN}/mesh_flash_mock.c
    ${CMOCK_BIN}/flash_manager_mock.c
    ${CMOCK_BIN}/bearer_handler_mock.c
    ${CMOCK_BIN}/scanner_mock.c
    ${CMOCK_BIN}/timeslot_mock.c
    ${CMOCK_BIN}/advertiser_mock.c
    ${CMOCK_BIN}/packet_mgr_mock.c
    ${CMOCK_BIN}/core_tx_adv_mock.c
    ${CMOCK_BIN}/ad_listener_mock.c
    )
add_unit_test(nrf_mesh "${nrf_mesh_srcs}" "${include_directories}" "${compile_options}")

set(serial_srcs
    src/ut_serial.c
    ../serial/src/serial.c
    ../core/src/log.c
    ${CMOCK_BIN}/serial_handler_access_mock.c
    ${CMOCK_BIN}/serial_handler_models_mock.c
    ${CMOCK_BIN}/bearer_event_mock.c
    ${CMOCK_BIN}/serial_bearer_mock.c
    ${CMOCK_BIN}/serial_handler_app_mock.c
    ${CMOCK_BIN}/serial_handler_config_mock.c
    ${CMOCK_BIN}/serial_handler_dfu_mock.c
    ${CMOCK_BIN}/serial_handler_mesh_mock.c
    ${CMOCK_BIN}/serial_handler_openmesh_mock.c
    ${CMOCK_BIN}/serial_handler_device_mock.c
    ${CMOCK_BIN}/serial_handler_prov_mock.c
    ${CMOCK_BIN}/serial_handler_dfu_mock.c
    )
add_unit_test(serial "${serial_srcs}" "${include_directories}" "${compile_options};-DNRF51")

# Serial bearer unit test - serial_bearer_slip_enc
set(serial_bearer_slip_enc_srcs
    src/ut_serial_bearer_slip_enc.c
    ../serial/src/serial_bearer.c
    ${CMOCK_BIN}/serial_mock.c
    ${CMOCK_BIN}/serial_uart_mock.c
    ${CMOCK_BIN}/packet_buffer_mock.c
    ${CMOCK_BIN}/bearer_event_mock.c
    )
add_unit_test(serial_bearer_slip_enc "${serial_bearer_slip_enc_srcs}" "${include_directories}" "${compile_options};-DNRF51;-DSERIAL_SLIP_ENCODING")
target_sources(ut_serial_bearer_slip_enc PUBLIC
    src/test_serial_bearer_common.c)

# Serial bearer unit test - serial_bearer_simple
set(serial_bearer_simple_srcs
    src/ut_serial_bearer_simple.c
    ../serial/src/serial_bearer.c
    ${CMOCK_BIN}/serial_mock.c
    ${CMOCK_BIN}/serial_uart_mock.c
    ${CMOCK_BIN}/packet_buffer_mock.c
    ${CMOCK_BIN}/bearer_event_mock.c
    )
add_unit_test(serial_bearer_simple "${serial_bearer_simple_srcs}" "${include_directories}" "${compile_options};-DNRF51")
target_sources(ut_serial_bearer_simple PUBLIC
    src/test_serial_bearer_common.c)

set(ticker_srcs
    src/ut_ticker.c
    ../core/src/ticker.c
    ${CMOCK_BIN}/timer_mock.c
    ${CMOCK_BIN}/timer_scheduler_mock.c
    )
add_unit_test(ticker "${ticker_srcs}" "${include_directories}" "${compile_options};-DNRF51")

set(queue_srcs
  src/ut_queue.c
  ../core/src/queue.c
  )
add_unit_test(queue "${queue_srcs}" "${include_directories}" "${compile_options}")

set(bearer_handler_srcs
  src/ut_bearer_handler.c
  ../bearer/src/bearer_handler.c
  ${CMOCK_BIN}/timer_mock.c
  ${CMOCK_BIN}/timeslot_mock.c
  ${CMOCK_BIN}/scanner_mock.c
  ${CMOCK_BIN}/queue_mock.c
  ${CMOCK_BIN}/nrf_mesh_cmsis_mock_mock.c
  )
add_unit_test(bearer_handler "${bearer_handler_srcs}" "${include_directories}" "${compile_options}")

set(net_beacon_srcs
    src/ut_net_beacon.c
    ../core/src/net_beacon.c
    ${CMOCK_BIN}/timer_mock.c
    ${CMOCK_BIN}/timer_scheduler_mock.c
    ${CMOCK_BIN}/beacon_mock.c
    ${CMOCK_BIN}/enc_mock.c
    ${CMOCK_BIN}/rand_mock.c
    ${CMOCK_BIN}/net_state_mock.c
    ../core/src/log.c
    )
add_unit_test(net_beacon "${net_beacon_srcs}" "${include_directories}" "${compile_options};-DNRF51")

set(prov_beacon_srcs
    src/ut_prov_beacon.c
    ../prov/src/prov_beacon.c
    ${CMOCK_BIN}/beacon_mock.c
    ${CMOCK_BIN}/nrf_mesh_configure_mock.c
    ${CMOCK_BIN}/event_mock.c
    ${CMOCK_BIN}/enc_mock.c
    ../core/src/log.c
    )
add_unit_test(prov_beacon "${prov_beacon_srcs}" "${include_directories}" "${compile_options};-DNRF51")

set(provisioning_srcs
    src/ut_provisioning.c
    ../prov/src/provisioning.c
    ${CMOCK_BIN}/nrf_mesh_mock.c
    ${CMOCK_BIN}/prov_bearer_adv_mock.c
    ${CMOCK_BIN}/event_mock.c
    ${CMOCK_BIN}/timer_scheduler_mock.c
    ${CMOCK_BIN}/enc_mock.c
    ${CMOCK_BIN}/timer_mock.c
    ../core/src/log.c
    )
add_unit_test(provisioning "${provisioning_srcs}" "${include_directories}" "${compile_options}")

set(prov_utils_srcs
    src/ut_prov_utils.c
    ../prov/src/prov_utils.c
    ../core/src/log.c
    ${CMOCK_BIN}/enc_mock.c
    ${CMOCK_BIN}/uECC_mock.c
    ${CMOCK_BIN}/rand_mock.c
    )
add_unit_test(prov_utils "${prov_utils_srcs}" "${include_directories}" "${compile_options}")

set(access_srcs
    src/ut_access.c
    ../access/src/access.c
    ../core/src/log.c
    ../core/src/fifo.c
    ${CMOCK_BIN}/device_state_manager_mock.c
    ${CMOCK_BIN}/flash_manager_mock.c
    ${CMOCK_BIN}/nrf_mesh_mock.c
    ${CMOCK_BIN}/nrf_mesh_events_mock.c
    ${CMOCK_BIN}/access_publish_mock.c
    ${CMOCK_BIN}/access_reliable_mock.c
    ${CMOCK_BIN}/bearer_event_mock.c
    ${CMOCK_BIN}/event_mock.c
    )
set(access_defines
    -DACCESS_ELEMENT_COUNT=2
    -DACCESS_MODEL_COUNT=16
    -DACCESS_SUBSCRIPTION_LIST_COUNT=15    # One less than the number of models
    -DDSM_NONVIRTUAL_ADDR_MAX=30)
add_unit_test(access "${access_srcs}" "${include_directories}" "${compile_options};${access_defines}")

set(access_reliable_srcs
    src/ut_access_reliable.c
    ${CMOCK_BIN}/access_mock.c
    ${CMOCK_BIN}/access_config_mock.c
    ${CMOCK_BIN}/timer_mock.c
    ${CMOCK_BIN}/timer_scheduler_mock.c
    ${CMOCK_BIN}/bearer_event_mock.c
    ../access/src/access_reliable.c)
set(access_reliable_defines
    -DACCESS_MODEL_COUNT=16
    -DACCESS_RELIABLE_TRANSFER_COUNT=8)
add_unit_test(access_reliable "${access_reliable_srcs}" "${include_directories}" "${compile_options};${access_reliable_defines}")

set(access_publish_srcs
    src/ut_access_publish.c
    ${CMOCK_BIN}/bearer_event_mock.c
    ${CMOCK_BIN}/timer_scheduler_mock.c
    ../access/src/access_publish.c
    )
add_unit_test(access_publish "${access_publish_srcs}" "${include_directories}" "${compile_options}")

set(config_client_srcs
    src/ut_config_client.c
    ${CMOCK_BIN}/access_mock.c
    ${CMOCK_BIN}/access_reliable_mock.c
    ${CMOCK_BIN}/access_config_mock.c
    ${CMOCK_BIN}/packet_mgr_mock.c
    ${CMAKE_SOURCE_DIR}/models/config/src/config_client.c
    ${CMAKE_SOURCE_DIR}/models/config/src/packed_index_list.c
    )
add_unit_test(config_client "${config_client_srcs}" "${include_directories}" "${compile_options}")

set(config_server_srcs
    src/ut_config_server.c
    ${CMAKE_SOURCE_DIR}/models/config/src/config_server.c
    ${CMAKE_SOURCE_DIR}/models/config/src/packed_index_list.c
    ../core/src/nrf_mesh_utils.c
    ${CMOCK_BIN}/composition_data_mock.c
    ${CMOCK_BIN}/access_mock.c
    ${CMOCK_BIN}/access_config_mock.c
    ${CMOCK_BIN}/device_state_manager_mock.c
    ${CMOCK_BIN}/heartbeat_mock.c
    ${CMOCK_BIN}/net_beacon_mock.c
    ${CMOCK_BIN}/net_state_mock.c
    ${CMOCK_BIN}/nrf_mesh_opt_mock.c
    ${CMOCK_BIN}/nrf_mesh_keygen_mock.c
    ${CMOCK_BIN}/rand_mock.c
    )
add_unit_test(config_server "${config_server_srcs}" "${include_directories}" "${compile_options};-DACCESS_ELEMENT_COUNT=1024;-DACCESS_SUBSCRIPTION_LIST_COUNT=3")

set(composition_data_srcs
    src/ut_composition_data.c
    ${CMOCK_BIN}/access_config_mock.c
    ${CMAKE_SOURCE_DIR}/models/config/src/composition_data.c
    )
set(composition_data_defines
    -DDEVICE_COMPANY_ID=0x000C
    -DDEVICE_PRODUCT_ID=0x001A
    -DDEVICE_VERSION_ID=0x0001
    -DREPLAY_CACHE_ENTRIES=0x0008
    -DDEVICE_FEATURES=0x0003
    -DACCESS_ELEMENT_COUNT=1
    -DACCESS_MODEL_COUNT=7)
add_unit_test(composition_data "${composition_data_srcs}" "${include_directories}" "${compile_options};${composition_data_defines}")

set(device_state_manager_srcs
    src/ut_device_state_manager.c
    ../access/src/device_state_manager.c
    ../core/src/nrf_mesh_utils.c
    ${CMOCK_BIN}/rand_mock.c
    ${CMOCK_BIN}/nrf_mesh_mock.c
    ${CMOCK_BIN}/nrf_mesh_keygen_mock.c
    ${CMOCK_BIN}/net_state_mock.c
    ${CMOCK_BIN}/flash_manager_mock.c
    ${CMOCK_BIN}/event_mock.c
    ${CMOCK_BIN}/bearer_event_mock.c
    )
add_unit_test(device_state_manager "${device_state_manager_srcs}" "${include_directories}" "${compile_options}")

set(net_state_srcs
    src/ut_net_state.c
    ../core/src/net_state.c
    ${CMOCK_BIN}/event_mock.c
    ${CMOCK_BIN}/flash_manager_mock.c
    )
add_unit_test(net_state "${net_state_srcs}" "${include_directories}" "${compile_options}")

set(bitfield_srcs
    src/ut_bitfield.c
    )
add_unit_test(bitfield "${bitfield_srcs}" "${include_directories}" "${compile_options}")

set(nrf_mesh_configure_srcs
    src/ut_nrf_mesh_configure.c
    ../core/src/nrf_mesh_configure.c
    )
add_unit_test(nrf_mesh_configure "${nrf_mesh_configure_srcs}" "${include_directories}" "${compile_options};-DNRF51")

set(bearer_event_srcs
    src/ut_bearer_event.c
    ../core/src/bearer_event.c
    ../core/src/fifo.c
    ../core/src/queue.c
    ${CMOCK_BIN}/nrf_mesh_cmsis_mock_mock.c
    )
add_unit_test(bearer_event "${bearer_event_srcs}" "${include_directories}" "${compile_options};-DNRF51")

set(nrf_mesh_node_config_srcs
    src/ut_nrf_mesh_node_config.c
    ../access/src/nrf_mesh_node_config.c
    ../core/src/log.c
    ${CMOCK_BIN}/config_server_mock.c
    ${CMOCK_BIN}/device_state_manager_mock.c
    ${CMOCK_BIN}/health_server_mock.c
    ${CMOCK_BIN}/nrf_mesh_mock.c
    ${CMOCK_BIN}/nrf_mesh_events_mock.c
    ${CMOCK_BIN}/nrf_mesh_prov_mock.c
    ${CMOCK_BIN}/nrf_mesh_prov_bearer_adv_mock.c
    ${CMOCK_BIN}/net_state_mock.c
    ${CMOCK_BIN}/access_mock.c
    ${CMOCK_BIN}/access_config_mock.c
    ${CMOCK_BIN}/flash_manager_mock.c
    )
add_unit_test(nrf_mesh_node_config "${nrf_mesh_node_config_srcs}" "${include_directories}" "${compile_options};-DNRF51")

set(flash_manager_srcs
    src/ut_flash_manager.c
    src/flash_manager_test_util.c
    ../core/src/flash_manager.c
    ../core/src/flash_manager_internal.c
    ../core/src/packet_buffer.c
    ../core/src/fifo.c
    ../core/src/queue.c
    ${CMOCK_BIN}/flash_manager_defrag_mock.c
    )
add_unit_test(flash_manager "${flash_manager_srcs}" "${include_directories}" "${compile_options}")

set(flash_manager_defrag_srcs
    src/ut_flash_manager_defrag.c
    src/flash_manager_test_util.c
    ../core/src/flash_manager_defrag.c
    ../core/src/fifo.c
    ../core/src/flash_manager_internal.c
    ${CMOCK_BIN}/flash_manager_mock.c
    )
add_unit_test(flash_manager_defrag "${flash_manager_defrag_srcs}" "${include_directories}" "${compile_options};-DNRF51")

set(msqueue_srcs
    src/ut_msqueue.c
    ../core/src/msqueue.c
    ${CMOCK_BIN}/mesh_flash_mock.c
    )
add_unit_test(msqueue "${msqueue_srcs}" "${include_directories}" "${compile_options}")

set(advertiser_srcs
    src/ut_advertiser.c
    ../bearer/src/advertiser.c
    ${CMOCK_BIN}/radio_config_mock.c
    ${CMOCK_BIN}/packet_buffer_mock.c
    ${CMOCK_BIN}/bearer_handler_mock.c
    ${CMOCK_BIN}/timer_mock.c
    ${CMOCK_BIN}/timer_scheduler_mock.c
    ${CMOCK_BIN}/broadcast_mock.c
    ${CMOCK_BIN}/rand_mock.c
    ${CMOCK_BIN}/bearer_event_mock.c
    )
add_unit_test(advertiser "${advertiser_srcs}" "${include_directories}" "${compile_options}")

set(health_server_srcs
    src/ut_health_server.c
    ${CMAKE_SOURCE_DIR}/models/health/src/health_server.c
    ${CMOCK_BIN}/access_mock.c
    ${CMOCK_BIN}/access_config_mock.c
    ${CMOCK_BIN}/access_reliable_mock.c
    ${CMOCK_BIN}/bearer_event_mock.c
    ${CMOCK_BIN}/timer_scheduler_mock.c
    )
add_unit_test(health_server "${health_server_srcs}" "${include_directories}" "${compile_options}")

set(health_client_srcs
    src/ut_health_client.c
    ${CMAKE_SOURCE_DIR}/models/health/src/health_client.c
    src/packet_mgr_linux.c
    ${CMOCK_BIN}/access_mock.c
    ${CMOCK_BIN}/access_config_mock.c
    ${CMOCK_BIN}/access_reliable_mock.c
    )
add_unit_test(health_client "${health_client_srcs}" "${include_directories}" "${compile_options}")

# Scanner
set(scanner_srcs
    src/ut_scanner.c
    ${CMOCK_BIN}/timer_scheduler_mock.c
    ${CMOCK_BIN}/packet_buffer_mock.c
    ${CMOCK_BIN}/toolchain_mock.c
    ${CMOCK_BIN}/timeslot_mock.c
    ${CMOCK_BIN}/radio_config_mock.c
    ${CMOCK_BIN}/timer_mock.c
    ${CMOCK_BIN}/nrf_mesh_cmsis_mock_mock.c
    ${CMOCK_BIN}/filter_engine_mock.c
    ${CMOCK_BIN}/bearer_event_mock.c
    )
add_unit_test(scanner "${scanner_srcs}" "${include_directories}" "${compile_options}")

# set(virtual_addressing_srcs
# src/ut_virtual_addressing.c
# ../core/src/transport.c
# ../core/src/log.c
# ../core/src/nrf_mesh_utils.c
# ${CMOCK_BIN}/enc_mock.c
# ${CMOCK_BIN}/event_mock.c
# ${CMOCK_BIN}/timer_mock.c
# ${CMOCK_BIN}/timer_scheduler_mock.c
# ${CMOCK_BIN}/net_state_mock.c
# ${CMOCK_BIN}/bearer_event_mock.c
# ${CMOCK_BIN}/network_mock.c
# ${CMOCK_BIN}/rand_mock.c
# ${CMOCK_BIN}/core_tx_mock.c
# )
# add_unit_test(virtual_addressing "${virtual_addressing_srcs}" "${include_directories}" "${compile_options}")

set(core_tx_srcs
    src/ut_core_tx.c
    ../core/src/core_tx.c
)
add_unit_test(core_tx "${core_tx_srcs}" "${include_directories}" "${compile_options}")

set(core_tx_adv_srcs
src/ut_core_tx_adv.c
../core/src/core_tx_adv.c
${CMOCK_BIN}/advertiser_mock.c
${CMOCK_BIN}/core_tx_mock.c
)
add_unit_test(core_tx_adv "${core_tx_adv_srcs}" "${include_directories}" "${compile_options}")

# Filters
set(filters_srcs
    src/ut_filters.c
    ../bearer/src/filter_engine.c
    ../bearer/src/ad_type_filter.c
    ../bearer/src/adv_packet_filter.c
    ../bearer/src/gap_address_filter.c
    ../bearer/src/rssi_filter.c
    ../core/src/list.c
    )
add_unit_test(filters "${filters_srcs}" "${include_directories}" "${compile_options}")

# Heartbeat module
set(heartbeat_test_srcs
	src/ut_heartbeat.c
    ../core/src/heartbeat.c
	../core/src/nrf_mesh_utils.c
    ${CMOCK_BIN}/timer_scheduler_mock.c
    ${CMOCK_BIN}/event_mock.c
	${CMOCK_BIN}/rand_mock.c
	${CMOCK_BIN}/timer_mock.c
	${CMOCK_BIN}/nrf_mesh_opt_mock.c
	${CMOCK_BIN}/transport_mock.c
    ${CMOCK_BIN}/config_server_mock.c
    ../core/src/log.c
    )
add_unit_test(heartbeat "${heartbeat_test_srcs}" "${include_directories}" "${compile_options}")

# AD listener (scanner mux)
set(ad_listener_srcs
    src/ut_ad_listener.c
    ../bearer/src/ad_listener.c
    ${CMOCK_BIN}/ad_type_filter_mock.c
    ${CMOCK_BIN}/list_mock.c
    )
add_unit_test(ad_listener "${ad_listener_srcs}" "${include_directories}" "${compile_options};-DAD_LISTENER_DEBUG_MODE")
