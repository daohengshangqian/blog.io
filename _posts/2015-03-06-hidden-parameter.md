---
layout: post
title:   Oracle隐含参数大全
date: 2015-03-06
categories: blog
tags: [Oracle]
description: Oracle隐含参数大全
---



参数名称 | 参数值 |默认值 | 参数描述
---|---|---|---
|_4030_dump_bitvec|4095|TRUE|bitvec to specify dumps prior to 4030 error|
|_4031_dump_bitvec|67194879|TRUE|bitvec to specify dumps prior to 4031 error|
|_4031_dump_interval|300|TRUE|Dump 4031 error once for each n-second interval|
|_4031_max_dumps|100|TRUE|Maximum number of 4031 dumps for this process|
|_4031_sga_dump_interval|3600|TRUE|Dump 4031 SGA heapdump error once for each n-second interval|
|_4031_sga_max_dumps|10|TRUE|Maximum number of SGA heapdumps|
|_NUMA_instance_mapping|Not specified|TRUE|Set of nodes that this instance should run on|
|_NUMA_pool_size|Not specified|TRUE|aggregate size in bytes of NUMA pool|
|_PX_use_large_pool|FALSE|TRUE|Use Large Pool as source of PX buffers|
|__db_cache_size|163577856|FALSE|Actual size of DEFAULT buffer pool for standard block size buffers|
|__dg_broker_service_names|%NULL%|TRUE|service names for broker use|
|__java_pool_size|67108864|FALSE|Actual size in bytes of java pool|
|__large_pool_size|8388608|FALSE|Actual size in bytes of large pool|
|__oracle_base|/u01/app/oracle|FALSE|ORACLE_BASE|
|__pga_aggregate_target|209715200|FALSE|Current target size for the aggregate PGA memory consumed|
|__sga_target|662700032|FALSE|Actual size of SGA|
|__shared_io_pool_size|0|FALSE|Actual size of shared IO pool|
|__shared_pool_size|394264576|FALSE|Actual size in bytes of shared pool|
|__streams_pool_size|8388608|FALSE|Actual size in bytes of streams pool|
|_abort_recovery_on_join|FALSE|TRUE|if TRUE, abort recovery on join reconfigurations|
|_accept_versions|%NULL%|TRUE|List of parameters for rolling operation|
|_active_instance_count|%NULL%|TRUE|number of active instances in the cluster database|
|_active_session_idle_limit|0|TRUE|active session idle limit|
|_active_session_legacy_behavior|FALSE|TRUE|active session legacy behavior|
|_active_standby_fast_reconfiguration|TRUE|TRUE|if TRUE optimize dlm reconfiguration for active/standby OPS|
|_adaptive_direct_read|TRUE|TRUE|Adaptive Direct Read|
|_adaptive_direct_write|TRUE|TRUE|Adaptive Direct Write|
|_adaptive_fetch_enabled|TRUE|TRUE|enable/disable adaptive fetch in parallel group by|
|_add_col_optim_enabled|TRUE|TRUE|Allows new add column optimization|
|_add_stale_mv_to_dependency_list|TRUE|TRUE|add stale mv to dependency list|
|_add_trim_for_nlssort|TRUE|TRUE|add trimming for fixed char semantics|
|_addm_auto_enable|TRUE|TRUE|governs whether ADDM gets run automatically after every AWR snapshot|
|_addm_skiprules|%NULL%|TRUE|comma-separated list of ADDM nodes to skip|
|_addm_version_check|TRUE|TRUE|governs whether ADDM checks the input AWR snapshot version|
|_adjust_literal_replacement|FALSE|TRUE|If TRUE, we will adjust the SQL/PLUS output|
|_adr_migrate_runonce|TRUE|TRUE|Enable/disable ADR Migrate Runonce action|
|_affinity_on|TRUE|TRUE|enable/disable affinity at run time|
|_aggregation_optimization_settings|0|FALSE|settings for aggregation optimizations|
|_aiowait_timeouts|100|TRUE|Number of aiowait timeouts before error is reported|
|_alert_expiration|604800|TRUE|seconds before an alert message is moved to exception queue|
|_alert_message_cleanup|1|TRUE|Enable Alert Message Cleanup|
|_alert_message_purge|1|TRUE|Enable Alert Message Purge|
|_alert_post_background|1|TRUE|Enable Background Alert Posting|
|_all_shared_dblinks|%NULL%|TRUE|treat all dblinks as shared|
|_allocate_creation_order|FALSE|TRUE|should files be examined in creation order during allocation|
|_allocation_update_interval|3|TRUE|interval at which successful search in L1 should be updated|
|_allow_cell_smart_scan_attr|TRUE|TRUE|Allow checking smart_scan_capable Attr|
|_allow_commutativity|TRUE|TRUE|allow for commutativity of +, * when comparing expressions|
|_allow_drop_snapshot_standby_grsp|FALSE|TRUE|Allow dropping snapshot standby guaranteed restore point|
|_allow_error_simulation|FALSE|TRUE|Allow error simulation for testing|
|_allow_level_without_connect_by|FALSE|TRUE|allow level without connect by|
|_allow_read_only_corruption|FALSE|TRUE|allow read-only open even if database is corrupt|
|_allow_resetlogs_corruption|FALSE|TRUE|allow resetlogs even if it will cause corruption|
|_allow_terminal_recovery_corruption|FALSE|TRUE|Finish terminal recovery even if it may cause corruption|
|_alternate_iot_leaf_block_split_points|TRUE|TRUE|enable alternate index-organized table leaf-block split-points|
|_always_anti_join|CHOOSE|FALSE|always use this method for anti-join when possible|
|_always_semi_join|CHOOSE|FALSE|always use this method for semi-join when possible|
|_always_star_transformation|FALSE|TRUE|always favor use of star transformation|
|_and_pruning_enabled|TRUE|FALSE|allow partition pruning based on multiple mechanisms|
|_app_ctx_vers|FALSE|TRUE|enable app ctx versioning|
|_appqos_qt|10|TRUE|System Queue time retrieval interval|
|_aq_tm_deqcountinterval|0|TRUE|dequeue count interval for Time Managers to cleanup DEQ IOT BLOCKS|
|_aq_tm_scanlimit|0|TRUE|scan limit for Time Managers to clean up IOT|
|_aq_tm_statistics_duration|0|TRUE|statistics collection window duration|
|_arch_comp_dbg_scan|0|TRUE|archive compression scan debug|
|_arch_comp_dec_block_check_dump|1|TRUE|decompress archive compression blocks for checking and dumping|
|_arch_compress_checksums|FALSE|TRUE|enable/disable row checksums for archive compressed blocks|
|_arch_compression|TRUE|TRUE|archive compression enabled|
|_arch_io_slaves|0|TRUE|ARCH I/O slaves|
|_arch_sim_mode|0|TRUE|Change behavior of local archiving|
|_array_update_vector_read_enabled|FALSE|TRUE|Enable array update vector read|
|_ash_disk_filter_ratio|10|TRUE|Ratio of the number of in-memory samples to the number of samples actually written to disk|
|_ash_disk_write_enable|TRUE|TRUE|To enable or disable Active Session History flushing|
|_ash_dummy_test_param|0|TRUE|Oracle internal dummy ASH parameter used ONLY for testing!|
|_ash_eflush_trigger|66|TRUE|The percentage above which if the in-memory ASH is full the emergency flusher will be triggered|
|_ash_enable|TRUE|TRUE|To enable or disable Active Session sampling and flushing|
|_ash_min_mmnl_dump|90|TRUE|Minimum Time interval passed to consider MMNL Dump|
|_ash_sample_all|FALSE|TRUE|To enable or disable sampling every connected session including ones waiting for idle waits|
|_ash_sampling_interval|1000|TRUE|Time interval between two successive Active Session samples in millisecs|
|_ash_size|1048618|TRUE|To set the size of the in-memory Active Session History buffers|
|_asm_acd_chunks|1|TRUE|initial ACD chunks created|
|_asm_admin_with_sysdba|FALSE|TRUE|Does the sysdba role have administrative privileges on ASM?|
|_asm_allow_lvm_resilvering|FALSE|TRUE|Enable disk resilvering for external redundancy|
|_asm_allow_only_raw_disks|TRUE|TRUE|Discovery only raw devices|
|_asm_ausize|1048576|TRUE|allocation unit size|
|_asm_automatic_rezone|TRUE|TRUE|automatically rebalance free space across zones|
|_asm_blksize|4096|TRUE|metadata block size|
|_asm_compatibility|10.1|TRUE|default ASM compatibility level|
|_asm_dbmsdg_nohdrchk|FALSE|TRUE|dbms_diskgroup.checkfile does not check block headers|
|_asm_direct_con_expire_time|120|TRUE|Expire time for idle direct connection to ASM instance|
|_asm_disable_amdu_dump|FALSE|TRUE|Disable AMDU dump|
|_asm_disable_multiple_instance_check|FALSE|TRUE|Disable checking for multiple ASM instances on a given node|
|_asm_disable_profilediscovery|FALSE|TRUE|disable profile query for discovery|
|_asm_disable_smr_creation|FALSE|TRUE|Do Not create smr|
|_asm_disk_repair_time|14400|TRUE|seconds to wait before dropping a failing disk|
|_asm_emulate_nfs_disk|FALSE|TRUE|Emulate NFS disk test event|
|_asm_emulmax|10000|TRUE|max number of concurrent disks to emulate I/O errors|
|_asm_emultimeout|0|TRUE|timeout before emulation begins (in 3s ticks)|
|_asm_fob_tac_frequency|9|TRUE|Timeout frequency for FOB cleanup|
|_asm_force_quiesce|FALSE|TRUE|Force diskgroup quiescing|
|_asm_hbeatwaitquantum|2|TRUE|quantum used to compute time-to-wait for a PST Hbeat check|
|_asm_imbalance_tolerance|3|TRUE|hundredths of a percentage of inter-disk imbalance to tolerate|
|_asm_instlock_quota|0|TRUE|ASM Instance Lock Quota|
|_asm_kfdpevent|0|TRUE|KFDP event|
|_asm_kfioevent|0|TRUE|KFIO event|
|_asm_libraries|ufs|TRUE|library search order for discovery|
|_asm_lsod_bucket_size|13|TRUE|ASM lsod bucket size|
|_asm_max_cod_strides|5|TRUE|maximum number of COD strides|
|_asm_maxio|1048576|TRUE|Maximum size of individual I/O request|
|_asm_partner_target_disk_part|8|TRUE|target maximum number of disk partners for repartnering|
|_asm_partner_target_fg_rel|4|TRUE|target maximum number of failure group relationships for repartnering|
|_asm_primary_load|1|TRUE|Number of cycles/extents to load for non-mirrored files|
|_asm_primary_load_cycles|TRUE|TRUE|True if primary load is in cycles, false if extent counts|
|_asm_random_zone|FALSE|TRUE|Random zones for new files|
|_asm_repairquantum|60|TRUE|quantum (in 3s) used to compute elapsed time for disk drop|
|_asm_root_directory|ASM|TRUE|ASM default root directory|
|_asm_runtime_capability_volume_support|FALSE|TRUE|runtime capability for volume support returns supported|
|_asm_secondary_load|10000|TRUE|Number of cycles/extents to load for mirrored files|
|_asm_secondary_load_cycles|FALSE|TRUE|True if secondary load is in cycles, false if extent counts|
|_asm_serialize_volume_rebalance|FALSE|TRUE|Serialize volume rebalance|
|_asm_shadow_cycle|3|TRUE|Inverse shadow cycle requirement|
|_asm_skip_rename_check|FALSE|TRUE|skip the checking of the clients for s/w compatibility for rename|
|_asm_skip_resize_check|FALSE|TRUE|skip the checking of the clients for s/w compatibility for resize|
|_asm_storagemaysplit|FALSE|TRUE|PST Split Possible|
|_asm_stripesize|131072|TRUE|ASM file stripe size|
|_asm_stripewidth|8|TRUE|ASM file stripe width|
|_asm_wait_time|18|TRUE|Max/imum time to wait before asmb exits|
|_asmlib_test|0|TRUE|Osmlib test event|
|_asmsid|asm|TRUE|ASM instance id|
|_assm_default|TRUE|TRUE|ASSM default|
|_assm_force_fetchmeta|FALSE|TRUE|enable metadata block fetching in ASSM segment scan|
|_async_recovery_claims|TRUE|TRUE|if TRUE, issue recovery claims asynchronously|
|_async_recovery_reads|TRUE|TRUE|if TRUE, issue recovery reads asynchronously|
|_async_ts_threshold|1|TRUE|check tablespace thresholds asynchronously|
|_auto_assign_cg_for_sessions|FALSE|TRUE|auto assign CGs for sessions|
|_auto_bmr|enabled|TRUE|enable/disable Auto BMR|
|_auto_bmr_bg_time|3600|TRUE|Auto BMR Process Run Time|
|_auto_bmr_fc_time|60|TRUE|Auto BMR Flood Control Time|
|_auto_bmr_pub_timeout|10|TRUE|Auto BMR Publish Timeout|
|_auto_bmr_req_timeout|60|TRUE|Auto BMR Requester Timeout|
|_auto_bmr_sess_threshold|30|TRUE|Auto BMR Request Session Threshold|
|_auto_bmr_sys_threshold|100|TRUE|Auto BMR Request System Threshold|
|_auto_manage_enable_offline_check|FALSE|TRUE|perodically check for OFFLINE disks and attempt to ONLINE|
|_auto_manage_exadata_disks|TRUE|TRUE|Automate Exadata disk management|
|_auto_manage_ioctl_bufsz|2048|TRUE|oss_ioctl buffer size, to read and respond to cell notifications|
|_auto_manage_num_tries|3|TRUE|Num. tries before giving up on a automation operation|
|_automatic_maintenance_test|0|TRUE|Enable AUTOTASK Test Mode|
|_automemory_broker_interval|3|TRUE|memory broker statistics gathering interval for auto memory|
|_autotask_max_window|480|TRUE|Maximum Logical Maintenance Window Length in minutes|
|_autotask_min_window|15|TRUE|Minimum Maintenance Window Length in minutes|
|_autotune_gtx_idle_time|600|TRUE|idle time to trigger auto-shutdown a gtx background process|
|_autotune_gtx_interval|5|TRUE|interval to autotune global transaction background processes|
|_autotune_gtx_threshold|60|TRUE|auto-tune threshold for degree of global transaction concurrency|
|_avoid_prepare|TRUE|TRUE|if TRUE, do not prepare a buffer when the master is local|
|_awr_corrupt_mode|FALSE|TRUE|AWR Corrupt Mode|
|_awr_disabled_flush_tables|%NULL%|TRUE|Disable flushing of specified AWR tables|
|_awr_flush_threshold_metrics|TRUE|TRUE|Enable/Disable Flushing AWR Threshold Metrics|
|_awr_flush_workload_metrics|FALSE|TRUE|Enable/Disable Flushing AWR Workload Metrics|
|_awr_mmon_cpuusage|TRUE|TRUE|Enable/disable AWR MMON CPU Usage Tracking|
|_awr_mmon_deep_purge_extent|7|TRUE|Set extent of rows to check each deep purge run|
|_awr_mmon_deep_purge_interval|7|TRUE|Set interval for deep purge of AWR contents|
|_awr_mmon_deep_purge_numrows|5000|TRUE|Set max number of rows per table to delete each deep purge run|
|_awr_restrict_mode|FALSE|TRUE|AWR Restrict Mode|
|_awr_sql_child_limit|200|TRUE|Setting for AWR SQL Child Limit|
|_b_tree_bitmap_plans|TRUE|FALSE|enable the use of bitmap plans for tables w. only B-tree indexes|
|_backup_align_write_io|TRUE|TRUE|align backup write I/Os|
|_backup_disk_bufcnt|0|TRUE|number of buffers used for DISK channels|
|_backup_disk_bufsz|0|TRUE|size of buffers used for DISK channels|
|_backup_disk_io_slaves|0|TRUE|BACKUP Disk I/O slaves|
|_backup_dynamic_buffers|TRUE|TRUE|dynamically compute backup/restore buffer sizes|
|_backup_encrypt_opt_mode|4294967294|TRUE|specifies encryption block optimization mode|
|_backup_file_bufcnt|0|TRUE|number of buffers used for file access|
|_backup_file_bufsz|0|TRUE|size of buffers used for file access|
|_backup_io_pool_size|1048576|TRUE|memory to reserve from the large pool|
|_backup_kgc_blksiz|9|TRUE|specifies buffer size to be used by HIGH compression|
|_backup_kgc_bufsz|0|TRUE|specifies buffer size to be used by BASIC compression|
|_backup_kgc_memlevel|8|TRUE|specifies memory level for MEDIUM compression|
|_backup_kgc_niters|0|TRUE|specifies number of iterations done by BASIC compression|
|_backup_kgc_perflevel|1|TRUE|specifies compression (performance) level for MEDIUM compression|
|_backup_kgc_scheme|ZLIB|TRUE|specifies compression scheme|
|_backup_kgc_type|0|TRUE|specifies compression type used by kgc BASIC compression|
|_backup_kgc_windowbits|15|TRUE|specifies window size for MEDIUM compression|
|_backup_ksfq_bufcnt|0|TRUE|number of buffers used for backup/restore|
|_backup_ksfq_bufcnt_max|64|TRUE|maximum number of buffers used for backup/restore|
|_backup_ksfq_bufsz|0|TRUE|size of buffers used for backup/restore|
|_backup_lzo_size|262144|TRUE|specifies buffer size for LOW compression|
|_backup_max_gap_size|4294967294|TRUE|largest gap in an incremental/optimized backup buffer, in bytes|
|_backup_seq_bufcnt|0|TRUE|number of buffers used for non-DISK channels|
|_backup_seq_bufsz|0|TRUE|size of buffers used for non-DISK channels|
|_bct_bitmaps_per_file|8|TRUE|number of bitmaps to store for each datafile|
|_bct_buffer_allocation_max|104857600|TRUE|maximum size of all change tracking buffer allocations, in bytes|
|_bct_buffer_allocation_min_extents|1|TRUE|mininum number of extents to allocate per buffer allocation|
|_bct_buffer_allocation_size|2097152|TRUE|size of one change tracking buffer allocation, in bytes|
|_bct_chunk_size|0|TRUE|change tracking datafile chunk size, in bytes|
|_bct_crash_reserve_size|262144|TRUE|change tracking reserved crash recovery SGA space, in bytes|
|_bct_file_block_size|0|TRUE|block size of change tracking file, in bytes|
|_bct_file_extent_size|0|TRUE|extent size of change tracking file, in bytes|
|_bct_fixtab_file|%NULL%|TRUE|change tracking file for fixed tables|
|_bct_initial_private_dba_buffer_size|0|TRUE|initial number of entries in the private change tracking dba buffers|
|_bct_public_dba_buffer_size|0|TRUE|total size of all public change tracking dba buffers, in bytes|
|_bitmap_or_improvement_enabled|TRUE|TRUE|controls extensions to partition pruning for general predicates|
|_block_sample_readahead_prob_threshold|10|TRUE|controls readahead value during block sampling|
|_blocking_sess_graph_cache_size|%NULL%|TRUE|blocking session graph cache size in bytes|
|_blocks_per_cache_server|32|TRUE|number of consecutive blocks per global cache server|
|_bloom_filter_debug|0|TRUE|debug level for bloom filtering|
|_bloom_filter_enabled|TRUE|FALSE|enables or disables bloom filter|
|_bloom_folding_density|16|TRUE|bloom filter folding density lower bound|
|_bloom_folding_enabled|TRUE|FALSE|Enable folding of bloom filter|
|_bloom_folding_min|131072|TRUE|bloom filter folding size lower bound|
|_bloom_max_size|268435456|TRUE|bloom filter maximum size in bytes|
|_bloom_predicate_enabled|TRUE|TRUE|enables or disables bloom filter predicate pushdown|
|_bloom_predicate_pushdown_to_storage|TRUE|TRUE|enables or disables bloom filter predicate pushdown to storage|
|_bloom_pruning_enabled|TRUE|FALSE|Enable partition pruning using bloom filtering|
|_bloom_pushing_max|524288|TRUE|bloom filter pushing size upper bound|
|_bloom_vector_elements|0|TRUE|number of elements in a bloom filter vector|
|_bmr_prefered_standby|%NULL%|TRUE|standby db_unique_name prefered for standby BMR|
|_branch_tagging|TRUE|TRUE|enable branch tagging for distributed transaction|
|_broadcast_scn_wait_timeout|10|TRUE|broadcast-on-commit scn wait timeout in centiseconds|
|_bsln_adaptive_thresholds_enabled|TRUE|TRUE|Adaptive Thresholds Enabled|
|_bt_mmv_query_rewrite_enabled|TRUE|TRUE|allow rewrites with multiple MVs and base tables|
|_buffer_busy_wait_timeout|100|TRUE|buffer busy wait time in centiseconds|
|_buffered_message_spill_age|300|TRUE|Buffered message spill age|
|_buffered_publisher_flow_control_threshold|0|TRUE|Flow control threshold for buffered publishers except capture|
|_bufq_stop_flow_control|FALSE|TRUE|Stop enforcing flow control for buffered queues|
|_build_deferred_mv_skipping_mvlog_update|FALSE|TRUE|DEFERRED MV creation skipping MV log setup update|
|_bump_highwater_mark_count|0|TRUE|how many blocks should we allocate per free list on advancing HWM|
|_bwr_for_flushed_pi|TRUE|TRUE|if TRUE, generate a BWR for a flushed PI|
|_cache_stats_monitor|FALSE|TRUE|if TRUE, enable cache stats monitoring|
|_capture_buffer_size|65536|TRUE|To set the size of the PGA I/O recording buffers|
|_capture_publisher_flow_control_threshold|0|TRUE|Flow control threshold for capture publishers|
|_case_sensitive_logon|TRUE|TRUE|case sensitive logon enabled|
|_causal_standby_wait_timeout|20|TRUE|Causal standby wait timeout|
|_cdmp_diagnostic_level|2|TRUE|cdmp directory diagnostic level|
|_cell_fast_file_create|TRUE|TRUE|Allow optimized file creation path for Cells|
|_cell_fast_file_restore|TRUE|TRUE|Allow optimized rman restore for Cells|
|_cell_file_format_chunk_size|0|TRUE|Cell file format chunk size in MB|
|_cell_index_scan_enabled|TRUE|TRUE|enable CELL processing of index FFS|
|_cell_offload_capabilities_enabled|0|TRUE|specifies capability table to load|
|_cell_offload_hybridcolumnar|TRUE|TRUE|Query offloading of hybrid columnar compressed tables to exadata|
|_cell_offload_predicate_reordering_enabled|FALSE|TRUE|enable out-of-order SQL processing offload to cells|
|_cell_offload_timezone|TRUE|TRUE|enable timezone related SQL processing offload to cells|
|_cell_range_scan_enabled|TRUE|TRUE|enable CELL processing of index range scans|
|_cgs_node_kill_escalation|TRUE|TRUE|CGS node kill escalation to CSS|
|_cgs_node_kill_escalation_wait|0|TRUE|CGS wait time to escalate node kill to CSS in seconds|
|_cgs_reconfig_extra_wait|3|TRUE|CGS reconfiguration extra wait time for CSS in seconds|
|_cgs_reconfig_timeout|0|TRUE|CGS reconfiguration timeout interval|
|_cgs_send_timeout|300|TRUE|CGS send timeout value|
|_cgs_tickets|1000|TRUE|CGS messaging tickets|
|_cgs_zombie_member_kill_wait|20|TRUE|CGS zombie member kill wait time in seconds|
|_check_block_after_checksum|TRUE|TRUE|perform block check after checksum if both are turned on|
|_check_block_new_invariant_for_flashback|FALSE|TRUE|check block new invariant for flashback|
|_check_ts_threshold|0|TRUE|check tablespace thresholds|
|_cleanup_rollback_entries|100|TRUE|no. of undo entries to apply per transaction cleanup|
|_cleanup_timeout|150|TRUE|timeout value for PMON cleanup|
|_cleanup_timeout_flags|2|TRUE|flags for PMON cleanup timeout|
|_clear_buffer_before_reuse|FALSE|TRUE|Always zero-out buffer before reuse for security|
|_client_ntfn_cleanup_interval|2400|TRUE|interval after which dead client registration cleanup task repeats|
|_client_ntfn_pinginterval|75|TRUE|time between pings to unreachable notification clients|
|_client_ntfn_pingretries|6|TRUE|number of times to ping unreachable notification clients|
|_client_ntfn_pingtimeout|30000|TRUE|timeout to connect to unreachable notification clients|
|_client_result_cache_bypass|FALSE|TRUE|bypass the client result cache|
|_client_tstz_error_check|TRUE|TRUE|Should Client give error for suspect Timestamp with Timezone operations|
|_close_cached_open_cursors|FALSE|TRUE|close cursors cached by PL/SQL at each commit|
|_close_deq_by_cond_curs|FALSE|TRUE|Close Dequeue By Condition Cursors|
|_cluster_library|clss|TRUE|cluster library selection|
|_clusterwide_global_transactions|TRUE|TRUE|enable/disable clusterwide global transactions|
|_collapse_wait_history|FALSE|TRUE|collapse wait history|
|_collect_undo_stats|TRUE|TRUE|Collect Statistics v$undostat|
|_column_compression_factor|0|TRUE|Column compression ratio|
|_column_elimination_off|FALSE|TRUE|turn off predicate-only column elimination|
|_column_tracking_level|1|TRUE|column usage tracking|
|_compilation_call_heap_extent_size|16384|TRUE|Size of the compilation call heaps extents|
|_complex_view_merging|TRUE|FALSE|enable complex view merging|
|_compression_above_cache|0|TRUE|number of recompression above cache for sanity check|
|_compression_advisor|0|TRUE|Compression advisor|
|_compression_chain|90|TRUE|percentage of chained rows allowed for Compression|
|_compression_compatibility|11.2.0.0.0|FALSE|Compression compatability|
|_connect_by_use_union_all|TRUE|FALSE|use union all for connect by|
|_controlfile_autobackup_delay|300|TRUE|time delay (in seconds) for performing controlfile autobackups|
|_controlfile_block_size|0|TRUE|control file block size in bytes|
|_controlfile_enqueue_dump|FALSE|TRUE|dump the system states after controlfile enqueue timeout|
|_controlfile_enqueue_holding_time|120|TRUE|control file enqueue max holding time in seconds|
|_controlfile_enqueue_holding_time_tracking_size|10|TRUE|control file enqueue holding time tracking size|
|_controlfile_enqueue_timeout|900|TRUE|control file enqueue timeout in seconds|
|_controlfile_section_init_size|%NULL%|TRUE|control file initial section size|
|_controlfile_section_max_expand|%NULL%|TRUE|control file max expansion rate|
|_controlfile_update_check|OFF|TRUE|controlfile update sanity check|
|_convert_set_to_join|FALSE|FALSE|enables conversion of set operator to join|
|_coord_message_buffer|0|TRUE|parallel recovery coordinator side extra message buffer size|
|_corrupted_rollback_segments|%NULL%|TRUE|corrupted undo segment list|
|_cost_equality_semi_join|TRUE|FALSE|enables costing of equality semi-join|
|_cp_num_hash_latches|1|TRUE|connection pool number of hash latches|
|_cpu_to_io|0|FALSE|divisor for converting CPU cost to I/O cost|
|_cr_grant_global_role|TRUE|TRUE|if TRUE, grant lock for CR requests when block is in global role|
|_cr_grant_local_role|AUTO|TRUE|turn 3-way CR grants off, make it automatic, or turn it on|
|_cr_grant_only|FALSE|TRUE|if TRUE, grant locks when possible and do not send the block|
|_cr_server_log_flush|TRUE|TRUE|if TRUE, flush redo log before serving a CR buffer|
|_create_table_in_any_cluster|FALSE|TRUE|allow creation of table in a cluster not owned by the user|
|_cursor_bind_capture_area_size|400|TRUE|maximum size of the cursor bind capture area|
|_cursor_bind_capture_interval|900|TRUE|interval (in seconds) between two bind capture for a cursor|
|_cursor_cache_time|1800|TRUE|number of seconds a session cached cursor stay in cache.|
|_cursor_db_buffers_pinned|171|TRUE|additional number of buffers a cursor can pin at once|
|_cursor_features_enabled|2|TRUE|Shared cursor features enabled bits.|
|_cursor_plan_enabled|TRUE|TRUE|enable collection and display of cursor plans|
|_cursor_plan_hash_version|1|TRUE|version of cursor plan hash value|
|_cursor_plan_unparse_enabled|TRUE|TRUE|enables/disables using unparse to build projection/predicates|
|_cursor_stats_bucket|15|TRUE|Hash table buckets for cursor stats (2^).|
|_cursor_stats_heap|4|TRUE|Heaps/latches for cursor stats.|
|_cvw_enable_weak_checking|TRUE|TRUE|enable weak view checking|
|_datafile_cow|FALSE|TRUE|Use copy on write snapshot for the renamed file|
|_datafile_write_errors_crash_instance|FALSE|TRUE|datafile write errors always crash instance|
|_db_16k_flash_cache_file|%NULL%|TRUE|flash cache file for 16k block size|
|_db_16k_flash_cache_size|0|TRUE|flash cache size for _db_16k_flash_cache_file|
|_db_2k_flash_cache_file|%NULL%|TRUE|flash cache file for 2k block size|
|_db_2k_flash_cache_size|0|TRUE|flash cache size for _db_2k_flash_cache_file|
|_db_32k_flash_cache_file|%NULL%|TRUE|flash cache file for 32k block size|
|_db_32k_flash_cache_size|0|TRUE|flash cache size for _db_32k_flash_cache_file|
|_db_4k_flash_cache_file|%NULL%|TRUE|flash cache file for 4k block size|
|_db_4k_flash_cache_size|0|TRUE|flash cache size for _db_4k_flash_cache_file|
|_db_8k_flash_cache_file|%NULL%|TRUE|flash cache file for 8k block size|
|_db_8k_flash_cache_size|0|TRUE|flash cache size for _db_8k_flash_cache_file|
|_db_aging_cool_count|1|TRUE|Touch count set when buffer cooled|
|_db_aging_freeze_cr|FALSE|TRUE|Make CR buffers always be too cold to keep in cache|
|_db_aging_hot_criteria|2|TRUE|Touch count which sends a buffer to head of replacement list|
|_db_aging_stay_count|0|TRUE|Touch count set when buffer moved to head of replacement list|
|_db_aging_touch_time|3|TRUE|Touch count which sends a buffer to head of replacement list|
|_db_always_check_system_ts|TRUE|TRUE|Always perform block check and checksum for System tablespace|
|_db_block_adjcheck|TRUE|TRUE|adjacent cache buffer checks - low blkchk overwrite parameter|
|_db_block_adjchk_level|0|TRUE|adjacent cache buffer check level|
|_db_block_align_direct_read|TRUE|TRUE|Align Direct Reads|
|_db_block_bad_write_check|FALSE|TRUE|enable bad write checks|
|_db_block_buffers|26023|TRUE|Number of database blocks cached in memory: hidden parameter|
|_db_block_cache_clone|FALSE|TRUE|Always clone data blocks on get (for debugging)|
|_db_block_cache_history|0|TRUE|buffer header tracing (non-zero only when debugging)|
|_db_block_cache_history_level|2|TRUE|buffer header tracing level|
|_db_block_cache_num_umap|0|TRUE|number of unmapped buffers (for tracking swap calls on blocks)|
|_db_block_cache_protect|FALSE|TRUE|protect database blocks (true only when debugging)|
|_db_block_cache_protect_internal|0|TRUE|protect database blocks (for strictly internal use only)|
|_db_block_check_for_debug|FALSE|TRUE|Check more and dump block before image for debugging|
|_db_block_check_objtyp|TRUE|TRUE|check objd and typ on cache disk read|
|_db_block_chunkify_ncmbr|FALSE|TRUE|chunkify noncontig multi block reads|
|_db_block_corruption_recovery_threshold|5|TRUE|threshold number of block recovery attempts|
|_db_block_do_full_mbreads|FALSE|TRUE|do full block read even if some blocks are in cache|
|_db_block_hash_buckets|65536|TRUE|Number of database block hash buckets|
|_db_block_hash_latches|2048|TRUE|Number of database block hash latches|
|_db_block_header_guard_level|0|TRUE|number of extra buffer headers to use as guard pages|
|_db_block_hi_priority_batch_size|0|TRUE|Fraction of writes for high priority reasons|
|_db_block_known_clean_pct|2|TRUE|Initial Percentage of buffers to maintain known clean|
|_db_block_lru_latches|8|TRUE|number of lru latches|
|_db_block_max_cr_dba|6|TRUE|Maximum Allowed Number of CR buffers per dba|
|_db_block_max_scan_pct|40|TRUE|Percentage of buffers to inspect when looking for free|
|_db_block_med_priority_batch_size|0|TRUE|Fraction of writes for medium priority reasons|
|_db_block_numa|1|TRUE|Number of NUMA nodes|
|_db_block_prefetch_fast_longjumps_enabled|TRUE|TRUE|Batched IO enable fast longjumps|
|_db_block_prefetch_limit|0|TRUE|Prefetch limit in blocks|
|_db_block_prefetch_override|0|TRUE|Prefetch force override in blocks|
|_db_block_prefetch_private_cache_enabled|TRUE|TRUE|Batched IO enable private cache|
|_db_block_prefetch_quota|10|TRUE|Prefetch quota as a percent of cache size|
|_db_block_prefetch_skip_reading_enabled|TRUE|TRUE|Batched IO enable skip reading buffers|
|_db_block_table_scan_buffer_size|4194304|TRUE|Size of shared table scan read buffer|
|_db_block_temp_redo|FALSE|TRUE|generate redo for temp blocks|
|_db_block_trace_protect|FALSE|TRUE|trace buffer protect calls|
|_db_block_vlm_check|FALSE|TRUE|check of rvlm mapping leaks (for debugging)|
|_db_block_vlm_leak_threshold|3|TRUE|Threshold for allowable vlm leaks|
|_db_blocks_per_hash_latch|%NULL%|TRUE|Number of blocks per hash latch|
|_db_cache_advice_max_size_factor|2|TRUE|cache advisory maximum multiple of current size to similate|
|_db_cache_advice_sample_factor|4|TRUE|cache advisory sampling factor|
|_db_cache_crx_check|FALSE|TRUE|check for costly crx examination functions|
|_db_cache_miss_check_les|FALSE|TRUE|check LEs after cache miss|
|_db_cache_pre_warm|TRUE|TRUE|Buffer Cache Pre-Warm Enabled : hidden parameter|
|_db_cache_process_cr_pin_max|%NULL%|TRUE|maximum number of cr pins a process may have|
|_db_cache_wait_debug|0|TRUE|trace new kslwaits|
|_db_change_notification_enable|TRUE|TRUE|enable db change notification|
|_db_check_cell_hints|FALSE|TRUE||
|_db_disable_temp_encryption|FALSE|TRUE|Diable Temp Encryption for Spills|
|_db_fast_obj_check|FALSE|TRUE|enable fast object drop sanity check|
|_db_fast_obj_ckpt|TRUE|TRUE|enable fast object checkpoint|
|_db_fast_obj_truncate|TRUE|TRUE|enable fast object truncate|
|_db_file_direct_io_count|1048576|TRUE|Sequential I/O buf size|
|_db_file_exec_read_count|105|TRUE|multiblock read count for regular clients|
|_db_file_format_io_buffers|4|TRUE|Block formatting I/O buf count|
|_db_file_noncontig_mblock_read_count|11|TRUE|number of noncontiguous db blocks to be prefetched|
|_db_file_optimizer_read_count|8|TRUE|multiblock read count for regular clients|
|_db_flash_cache_keep_limit|80|TRUE|Flash cache keep buffer upper limit in percentage|
|_db_handles|750|TRUE|System-wide simultaneous buffer operations|
|_db_handles_cached|5|TRUE|Buffer handles cached each process|
|_db_hot_block_tracking|FALSE|TRUE|track hot blocks for hash latch contention|
|_db_index_block_checking|TRUE|TRUE|index block checking override parameter|
|_db_l2_tracing|0|TRUE|flash cache debug tracing|
|_db_large_dirty_queue|25|TRUE|Number of buffers which force dirty queue to be written|
|_db_lost_write_checking|2|TRUE|Enable scn based lost write detection mechanism|
|_db_lost_write_tracing|FALSE|TRUE|Enable _db_lost_write_checking tracing|
|_db_mttr_advice|ON|TRUE|MTTR advisory|
|_db_mttr_partitions|0|TRUE|number of partitions for MTTR advisory|
|_db_mttr_sample_factor|64|TRUE|MTTR simulation sampling factor|
|_db_mttr_sim_target|%NULL%|TRUE|MTTR simulation targets|
|_db_mttr_sim_trace_size|256|TRUE|MTTR simulation trace size|
|_db_mttr_trace_to_alert|FALSE|TRUE|dump trace entries to alert file|
|_db_noarch_disble_optim|FALSE|TRUE|Image redo logging (NOARCHIVEMODE)|
|_db_num_evict_waitevents|64|TRUE|number of evict wait events|
|_db_obj_enable_ksr|TRUE|TRUE|enable ksr in object checkpoint/reuse|
|_db_percent_hot_default|50|TRUE|Percent of default buffer pool considered hot|
|_db_percent_hot_keep|0|TRUE|Percent of keep buffer pool considered hot|
|_db_percent_hot_recycle|0|TRUE|Percent of recycle buffer pool considered hot|
|_db_percpu_create_cachesize|2|TRUE|size of cache created per cpu in deferred cache create|
|_db_prefetch_histogram_statistics|FALSE|TRUE|maintain prefetch histogram statistics in x$kcbprfhs|
|_db_recovery_temporal_file_dest|%NULL%|TRUE|default database recovery temporal file location|
|_db_required_percent_fairshare_usage|10|TRUE|percent of fairshare a processor group should always use|
|_db_row_overlap_checking|TRUE|TRUE|row overlap checking override parameter for data/index blocks|
|_db_todefer_cache_create|TRUE|TRUE|buffer cache deferred create|
|_db_writer_chunk_writes|0|TRUE|Number of writes DBWR should wait for|
|_db_writer_coalesce_area_size|2211840|TRUE|Size of memory allocated to dbwriter for coalescing writes|
|_db_writer_coalesce_write_limit|131072|TRUE|Limit on size of coalesced write|
|_db_writer_flush_imu|TRUE|TRUE|If FALSE, DBWR will not downgrade IMU txns for AGING|
|_db_writer_histogram_statistics|FALSE|TRUE|maintain dbwr histogram statistics in x$kcbbhs|
|_db_writer_max_writes|0|TRUE|Max number of outstanding DB Writer IOs|
|_db_writer_nomemcopy_coalesce|FALSE|TRUE|Enable DBWR no-memcopy coalescing|
|_db_writer_verify_writes|FALSE|TRUE|Enable lost write detection mechanism|
|_dbg_proc_startup|FALSE|TRUE|debug process startup|
|_dbms_sql_security_level|1|TRUE|Security level in DBMS_SQL|
|_dbrm_dynamic_threshold|33489896|TRUE|DBRM dynamic threshold setting|
|_dbrm_quantum|%NULL%|TRUE|DBRM quantum|
|_dbrm_readylist_chk|1|TRUE|Resource Manager Diagnostic Ready List Check|
|_dbrm_runchk|0|TRUE|Resource Manager Diagnostic Running Thread Check|
|_dbwr_async_io|TRUE|TRUE|Enable dbwriter asynchronous writes|
|_dbwr_scan_interval|300|TRUE|dbwriter scan interval|
|_dbwr_tracing|0|TRUE|Enable dbwriter tracing|
|_dde_flood_control_init|TRUE|TRUE|Initialize Flood Control at database open|
|_dead_process_scan_interval|60|TRUE|PMON dead process scan interval|
|_deadlock_diagnostic_level|2|TRUE|automatic deadlock resolution diagnostics level|
|_deadlock_resolution_incidents_enabled|TRUE|TRUE|create incidents during deadlock resolution|
|_deadlock_resolution_level|1|TRUE|automatic deadlock resolution level|
|_deadlock_resolution_min_wait_timeout_secs|60|TRUE|the minimum wait timeout required for deadlock resolution|
|_deadlock_resolution_signal_process_thresh_secs|60|TRUE|the amount of time given to process a deadlock resolution signal|
|_dedicated_server_poll_count|10|TRUE|dedicated server poll count|
|_dedicated_server_post_wait|FALSE|TRUE|dedicated server post/wait|
|_dedicated_server_post_wait_call|FALSE|TRUE|dedicated server post/wait call|
|_default_encrypt_alg|0|TRUE|default encryption algorithm|
|_default_non_equality_sel_check|TRUE|TRUE|sanity check on default selectivity for like/range predicate|
|_defer_log_boundary_ckpt|TRUE|TRUE|defer media recovery checkpoint at log boundary|
|_defer_log_count|2|TRUE|Number of log boundaries media recovery checkpoint lags behind|
|_deferred_constant_folding_mode|DEFAULT|TRUE|Deferred constant folding mode|
|_deferred_log_dest_is_valid|TRUE|TRUE|consider deferred log dest as valid for log deletion (TRUE/FALSE)|
|_delay_index_maintain|TRUE|TRUE|delays index maintenance until after MV is refreshed|
|_delta_push_share_blockers|0|TRUE|enable delta push if greater than the # of share blockers|
|_deq_execute_reset_time|30|TRUE|deq execute reset time|
|_deq_ht_child_latches|8|TRUE|deq ht child latches|
|_deq_ht_max_elements|100000|TRUE|deq ht max elements|
|_deq_large_txn_size|25000|TRUE|deq large txn size|
|_deq_log_array_size|10000|TRUE|deq log array size|
|_deq_max_fetch_count|10|TRUE|deq max fetch count|
|_deq_maxwait_time|0|TRUE|Change wait times between dequeue calls|
|_desired_readmem_rate|90|TRUE|The desired percentage of redo reading from memory|
|_dg_corrupt_redo_log|0|TRUE|Corrupt redo log validation during archivals|
|_diag_adr_auto_purge|TRUE|TRUE|Enable/disable ADR MMON Auto Purging|
|_diag_adr_enabled|TRUE|TRUE|Parameter to enable/disable Diag ADR|
|_diag_adr_test_param|0|TRUE|Test parameter for Diagnosability|
|_diag_arb_before_kill|FALSE|TRUE|dump diagnostics before killing unresponsive ARBs|
|_diag_backward_compat|TRUE|TRUE|Backward Compatibility for Diagnosability|
|_diag_cc_enabled|TRUE|TRUE|Parameter to enable/disable Diag Call Context|
|_diag_conf_cap_enabled|TRUE|TRUE|Parameter to enable/disable Diag Configuration Capture|
|_diag_daemon|TRUE|TRUE|start DIAG daemon|
|_diag_dde_async_age_limit|300|TRUE|diag dde async actions: message age limit (in seconds)|
|_diag_dde_async_cputime_limit|300|TRUE|diag dde async actions: action cputime limit (in seconds)|
|_diag_dde_async_mode|1|TRUE|diag dde async actions: dispatch mode|
|_diag_dde_async_msg_capacity|1024|TRUE|diag dde async actions: message buffer capacity|
|_diag_dde_async_msgs|50|TRUE|diag dde async actions: number of preallocated message buffers|
|_diag_dde_async_process_rate|5|TRUE|diag dde async actions: message processing rate - per loop|
|_diag_dde_async_runtime_limit|900|TRUE|diag dde async actions: action runtime limit (in seconds)|
|_diag_dde_async_slaves|5|TRUE|diag dde async actions: max number of concurrent slave processes|
|_diag_dde_enabled|TRUE|TRUE|enable DDE handling of critical errors|
|_diag_dde_fc_enabled|TRUE|TRUE|Parameter to enable/disable Diag Flood Control|
|_diag_dde_fc_implicit_time|0|TRUE|Override Implicit Error Flood Control time parameter|
|_diag_dde_fc_macro_time|0|TRUE|Override Macro Error Flood Control time parameter|
|_diag_dde_inc_proc_delay|1|TRUE|The minimum delay between two MMON incident sweeps (minutes)|
|_diag_diagnostics|TRUE|TRUE|Turn off diag diagnostics|
|_diag_dump_timeout|30|TRUE|timeout parameter for SYNC dump|
|_diag_enable_startup_events|FALSE|TRUE|enable events in instance startup notifiers|
|_diag_hm_rc_enabled|TRUE|TRUE|Parameter to enable/disable Diag HM Reactive Checks|
|_diag_hm_tc_enabled|FALSE|TRUE|Parameter to enable/disable Diag HM Test(dummy) Checks|
|_diag_proc_enabled|TRUE|TRUE|enable hung process diagnostic API|
|_diag_proc_max_time_ms|30000|TRUE|hung process diagnostic API max wait time in milliseconds|
|_diag_proc_stack_capture_type|1|TRUE|hung process diagnostic API stack capture type|
|_diag_uts_control|0|TRUE|UTS control parameter|
|_diag_verbose_error_on_init|0|TRUE|Allow verbose error tracing on diag init|
|_dimension_skip_null|TRUE|FALSE|control dimension skip when null feature|
|_direct_io_skip_cur_slot_on_error|TRUE|TRUE|Skip current slot on error|
|_direct_io_slots|0|TRUE|number of slots for direct path I/O|
|_direct_io_wslots|0|TRUE|number of write slots for direct path I/O|
|_direct_path_insert_features|0|TRUE|disable direct path insert features|
|_disable_12751|FALSE|TRUE|disable policy timeout error (ORA-12751)|
|_disable_active_influx_move|FALSE|TRUE|disable active influx move during parallel media recovery|
|_disable_adaptive_shrunk_aggregation|FALSE|TRUE|adaptive shrunk aggregation|
|_disable_autotune_gtx|FALSE|TRUE|disable autotune global transaction background processes|
|_disable_cell_optimized_backups|FALSE|TRUE|disable cell optimized backups|
|_disable_cpu_check|FALSE|TRUE|disable cpu_count check|
|_disable_datalayer_sampling|FALSE|TRUE|disable datalayer sampling|
|_disable_duplex_link|TRUE|TRUE|Turn off connection duplexing|
|_disable_fast_aggregation|FALSE|TRUE|fast aggregation|
|_disable_fast_validate|FALSE|TRUE|disable PL/SQL fast validation|
|_disable_fastopen|FALSE|TRUE|Do Not Use Fastopen|
|_disable_fba_qrw|0|TRUE|disable flashback archiver query rewrite|
|_disable_fba_wpr|0|TRUE|disable flashback archiver wait for prepared transactions|
|_disable_file_locks|FALSE|TRUE|disable file locks for control, data, redo log files|
|_disable_flashback_archiver|0|TRUE|disable flashback archiver |
|_disable_flashback_wait_callback|FALSE|TRUE|Disable flashback wait callback|
|_disable_function_based_index|FALSE|TRUE|disable function-based index matching|
|_disable_gvaq_cache|FALSE|TRUE|Disable cache|
|_disable_health_check|FALSE|TRUE|Disable Health Check|
|_disable_highres_ticks|FALSE|TRUE|disable high-res tick counter|
|_disable_image_check|FALSE|TRUE|Disable Oracle executable image checking|
|_disable_implicit_row_movement|FALSE|TRUE|disable implicit row movement|
|_disable_incremental_checkpoints|FALSE|TRUE|Disable incremental checkpoints for thread recovery|
|_disable_incremental_recovery_ckpt|FALSE|TRUE|Disable incremental recovery checkpoint mechanism|
|_disable_index_block_prefetching|FALSE|TRUE|disable index block prefetching|
|_disable_initial_block_compression|FALSE|TRUE|disable initial block compression|
|_disable_instance_params_check|FALSE|TRUE|disable instance type check for ksp|
|_disable_interface_checking|FALSE|TRUE|disable interface checking at startup|
|_disable_kcb_flashback_blocknew_opt|FALSE|TRUE|Disable KCB flashback block new optimization|
|_disable_kcbhxor_osd|FALSE|TRUE|disable kcbh(c)xor OSD functionality|
|_disable_kcbl_flashback_blocknew_opt|FALSE|TRUE|Disable KCBL flashback block new optimization|
|_disable_latch_free_SCN_writes_via_32cas|FALSE|TRUE|disable latch-free SCN writes using 32-bit compare & swap|
|_disable_latch_free_SCN_writes_via_64cas|FALSE|TRUE|disable latch-free SCN writes using 64-bit compare & swap|
|_disable_logging|FALSE|TRUE|Disable logging|
|_disable_metrics_group|0|TRUE|Disable Metrics Group (or all Metrics Groups)|
|_disable_multiple_block_sizes|FALSE|TRUE|disable multiple block size support (for debugging)|
|_disable_odm|FALSE|TRUE|disable odm feature|
|_disable_parallel_conventional_load|FALSE|TRUE|Disable parallel conventional loads|
|_disable_primary_bitmap_switch|FALSE|TRUE|disable primary bitmap switch|
|_disable_read_only_open_dict_check|FALSE|TRUE|Disable read-only open dictionary check|
|_disable_rebalance_compact|FALSE|TRUE|disable space usage checks for storage reconfiguration|
|_disable_rebalance_space_check|FALSE|TRUE|disable space usage checks for storage reconfiguration|
|_disable_recovery_read_skip|FALSE|TRUE|Disable the read optimization during media recovery|
|_disable_sample_io_optim|FALSE|TRUE|disable row sampling IO optimization|
|_disable_savepoint_reset|FALSE|TRUE|disable the fix for bug 1402161|
|_disable_sec_ticks|FALSE|TRUE|disable low-res (sec) counter|
|_disable_selftune_checkpointing|FALSE|TRUE|Disable self-tune checkpointing|
|_disable_streams_pool_auto_tuning|FALSE|TRUE|disable streams pool auto tuning|
|_disable_sun_rsm|TRUE|TRUE|Disable IPC OSD support for Sun RSMAPI|
|_disable_system_state|4294967294|TRUE|disable system state dump|
|_disable_system_state_wait_samples|FALSE|TRUE|Disable system state dump - wait samples|
|_disable_temp_tablespace_alerts|FALSE|TRUE|disable tablespace alerts for TEMPORARY tablespaces|
|_disable_thread_internal_disable|FALSE|TRUE|Disable thread internal disable feature|
|_disable_txn_alert|0|TRUE|disable txn layer alert|
|_disable_undo_tablespace_alerts|FALSE|TRUE|disable tablespace alerts for UNDO tablespaces|
|_disable_vktm|FALSE|TRUE|disable vktm process|
|_disable_wait_state|%NULL%|TRUE|Disable wait state|
|_discrete_transactions_enabled|FALSE|TRUE|enable OLTP mode|
|_disk_sector_size_override|FALSE|TRUE|if TRUE, OSD sector size could be overridden|
|_diskmon_pipe_name|%NULL%|TRUE|DiSKMon skgznp pipe name|
|_dispatcher_rate_scale|%NULL%|TRUE|scale to display rate statistic (100ths of a second)|
|_dispatcher_rate_ttl|%NULL%|TRUE|time-to-live for rate statistic (100ths of a second)|
|_distinct_view_unnesting|FALSE|TRUE|enables unnesting of in subquery into distinct view|
|_distributed_recovery_connection_hold_time|200|TRUE|number of seconds RECO holds outbound connections open|
|_dlmtrace|%NULL%|TRUE|Trace string of global enqueue type(s)|
|_dm_max_shared_pool_pct|1|TRUE|max percentage of the shared pool to use for a mining model|
|_dml_batch_error_limit|0|TRUE|number or error handles allocated for DML in batch mode|
|_dml_monitoring_enabled|TRUE|TRUE|enable modification monitoring|
|_domain_index_batch_size|2000|TRUE|maximum number of rows from one call to domain index fetch routine|
|_domain_index_dml_batch_size|200|TRUE|maximum number of rows for one call to domain index dml routines|
|_dra_bmr_number_threshold|1000|TRUE|Maximum number of BMRs that can be done to a file|
|_dra_bmr_percent_threshold|10|TRUE|Maximum percentage of blocks in a file that can be BMR-ed|
|_dra_enable_offline_dictionary|FALSE|TRUE|Enable the periodic creation of the offline dictionary for DRA|
|_drop_flashback_logical_operations_enq|FALSE|TRUE|Drop logical operations enqueue immediately during flashback marker generation|
|_drop_table_granule|256|TRUE|drop_table_granule|
|_drop_table_optimization_enabled|TRUE|TRUE|reduce SGA memory use during drop of a partitioned table|
|_ds_enable_auto_txn|FALSE|TRUE|Dynamic Sampling Service Autonomous Transaction control parameter|
|_ds_iocount_iosize|6553664|TRUE|Dynamic Sampling Service defaults: #IOs and IO Size|
|_ds_parse_model|2|TRUE|Dynamic Sampling Service Parse Model control parameter|
|_dsc_feature_level|0|TRUE|controls the feature level for deferred segment creation|
|_dss_cache_flush|FALSE|TRUE|enable full cache flush for parallel execution|
|_dtree_area_size|131072|TRUE|size of Decision Tree Classification work area|
|_dtree_binning_enabled|TRUE|TRUE|Decision Tree Binning Enabled|
|_dtree_bintest_id|0|TRUE|Decision Tree Binning Test ID|
|_dtree_compressbmp_enabled|TRUE|TRUE|Decision Tree Using Compressed Bitmaps Enabled|
|_dtree_max_surrogates|1|TRUE|maximum number of surrogates|
|_dtree_pruning_enabled|TRUE|TRUE|Decision Tree Pruning Enabled|
|_dummy_instance|FALSE|TRUE|dummy instance started by RMAN|
|_dump_common_subexpressions|FALSE|TRUE|dump common subexpressions|
|_dump_connect_by_loop_data|FALSE|TRUE|dump connect by loop error message into trc file|
|_dump_cursor_heap_sizes|FALSE|TRUE|dump comp/exec heap sizes to tryace file|
|_dump_interval_limit|120|TRUE|trace dump time interval limit (in seconds)|
|_dump_max_limit|5|TRUE|max number of dump within dump interval|
|_dump_qbc_tree|0|TRUE|dump top level query parse tree to trace|
|_dump_rcvr_ipc|TRUE|TRUE|if TRUE enables IPC dump at instance eviction time|
|_dump_system_state_scope|local|TRUE|scope of sysstate dump during instance termination|
|_dump_trace_scope|global|TRUE|scope of trace dump during a process crash|
|_dynamic_rls_policies|TRUE|TRUE|rls policies are dynamic|
|_dynamic_stats_threshold|30|TRUE|delay threshold (in seconds) between sending statistics messages|
|_eliminate_common_subexpr|TRUE|FALSE|enables elimination of common sub-expressions|
|_emon_max_active_connections|256|TRUE|maximum open connections to clients per emon|
|_emon_outbound_connect_timeout|30000|TRUE|timeout for completing connection set up to clients|
|_emon_regular_ntfn_slaves|4|TRUE|number of EMON slaves doing regular database notifications|
|_enable_Front_End_View_Optimization|1|TRUE|enable front end view optimization|
|_enable_NUMA_optimization|FALSE|TRUE|Enable NUMA specific optimizations|
|_enable_NUMA_support|FALSE|TRUE|Enable NUMA support and optimizations|
|_enable_asyncvio|FALSE|TRUE|enable asynch vectored I/O|
|_enable_automatic_maintenance|1|TRUE|if 1, Automated Maintenance Is Enabled|
|_enable_automatic_sqltune|TRUE|TRUE|Automatic SQL Tuning Advisory enabled parameter|
|_enable_block_level_transaction_recovery|TRUE|TRUE|enable block level recovery|
|_enable_check_truncate|TRUE|TRUE|enable checking of corruption caused by canceled truncate|
|_enable_cscn_caching|FALSE|TRUE|enable commit SCN caching for all transactions|
|_enable_ddl_wait_lock|TRUE|TRUE|use this to turn off ddls with wait semantics|
|_enable_default_affinity|0|TRUE|enable default implementation of hard affinity osds|
|_enable_default_temp_threshold|TRUE|TRUE|Enable Default Tablespace Utilization Threshold for UNDO Tablespaces|
|_enable_default_undo_threshold|TRUE|TRUE|Enable Default Tablespace Utilization Threshold for TEMPORARY Tablespaces|
|_enable_dml_lock_escalation|TRUE|TRUE|enable dml lock escalation against partitioned tables if TRUE|
|_enable_editions_for_users|FALSE|TRUE|enable editions for all users|
|_enable_exchange_validation_using_check|TRUE|TRUE|use check constraints on the table for validation|
|_enable_fast_ref_after_mv_tbs|FALSE|TRUE|enable fast refresh after move tablespace|
|_enable_hash_overflow|FALSE|TRUE|TRUE - enable hash cluster overflow based on SIZE|
|_enable_hwm_sync|TRUE|TRUE|enable HWM synchronization|
|_enable_kqf_purge|TRUE|TRUE|Enable KQF fixed runtime table purge|
|_enable_list_io|FALSE|TRUE|Enable List I/O|
|_enable_midtier_affinity|TRUE|TRUE|enable midtier affinity metrics processing|
|_enable_minscn_cr|TRUE|TRUE|enable/disable minscn optimization for CR|
|_enable_nativenet_tcpip|FALSE|TRUE|Enable skgxp driver usage for native net|
|_enable_obj_queues|TRUE|TRUE|enable object queues|
|_enable_online_index_without_s_locking|TRUE|TRUE|Allow online index creation algorithm without S DML lock|
|_enable_query_rewrite_on_remote_objs|TRUE|TRUE|mv rewrite on remote table/view|
|_enable_refresh_schedule|TRUE|TRUE|enable or disable MV refresh scheduling (revert to 9.2 behavior)|
|_enable_reliable_latch_waits|TRUE|TRUE|Enable reliable latch waits|
|_enable_rlb|TRUE|TRUE|enable RLB metrics processing|
|_enable_row_shipping|TRUE|TRUE|use the row shipping optimization for wide table selects|
|_enable_sb_detection|TRUE|TRUE|Split Brain Detection|
|_enable_scn_wait_interface|TRUE|TRUE|use this to turn off scn wait interface in kta|
|_enable_separable_transactions|FALSE|TRUE|enable/disable separable transactions|
|_enable_shared_pool_durations|TRUE|TRUE|temporary to disable/enable kgh policy|
|_enable_shared_server_vector_io|FALSE|TRUE|Enable shared server vector I/O|
|_enable_space_preallocation|3|TRUE|enable space pre-allocation|
|_enable_spacebg|TRUE|TRUE|enable space management background task|
|_enable_tablespace_alerts|TRUE|TRUE|enable tablespace alerts|
|_enable_type_dep_selectivity|TRUE|FALSE|enable type dependent selectivity estimates|
|_endprot_chunk_comment|chk 10235 dflt|TRUE|chunk comment for selective overrun protection|
|_endprot_heap_comment|hp 10235 dflt|TRUE|heap comment for selective overrun protection|
|_endprot_subheaps|TRUE|TRUE|selective overrun protection for subeheaps|
|_enqueue_deadlock_scan_secs|0|TRUE|deadlock scan interval|
|_enqueue_deadlock_time_sec|5|TRUE|requests with timeout <= this will not have deadlock detection|
|_enqueue_debug_multi_instance|FALSE|TRUE|debug enqueue multi instance|
|_enqueue_hash|529|TRUE|enqueue hash table length|
|_enqueue_hash_chain_latches|1|TRUE|enqueue hash chain latches|
|_enqueue_locks|3010|TRUE|locks for managed enqueues|
|_enqueue_paranoia_mode_enabled|FALSE|TRUE|enable enqueue layer advanced debugging checks|
|_enqueue_resources|1304|TRUE|resources for enqueues|
|_evolve_plan_baseline_report_level|typical|TRUE|Level of detail to show in plan verification/evolution report|
|_evt_system_event_propagation|TRUE|TRUE|disable system event propagation|
|_expand_aggregates|TRUE|TRUE|expand aggregates|
|_explain_rewrite_mode|FALSE|TRUE|allow additional messages to be generated during explain rewrite|
|_extended_pruning_enabled|TRUE|TRUE|do runtime pruning in iterator if set to TRUE|
|_fair_remote_cvt|FALSE|TRUE|if TRUE enables fair remote convert|
|_fairness_threshold|2|TRUE|number of times to CR serve before downgrading lock|
|_fast_cursor_reexecute|FALSE|TRUE|use more memory in order to get faster execution|
|_fast_dual_enabled|TRUE|TRUE|enable/disable fast dual|
|_fast_full_scan_enabled|TRUE|FALSE|enable/disable index fast full scan|
|_fastpin_enable|1|TRUE|enable reference count based fast pins|
|_fbda_busy_percentage|0|TRUE|flashback archiver busy percentage|
|_fbda_debug_assert|0|TRUE|flashback archiver debug assert for testing|
|_fbda_debug_mode|0|TRUE|flashback archiver debug event for testing|
|_fbda_global_bscn_lag|0|TRUE|flashback archiver global barrier scn lag|
|_fbda_inline_percentage|0|TRUE|flashback archiver inline percentage|
|_fbda_rac_inactive_limit|0|TRUE|flashback archiver rac inactive limit|
|_fg_iorm_slaves|1|TRUE|ForeGround I/O slaves for IORM|
|_fg_log_checksum|TRUE|TRUE|Checksum redo in foreground process|
|_fg_sync_sleep_usecs|0|TRUE|Log file sync via usleep|
|_fic_algorithm_set|automatic|TRUE|Set Frequent Itemset Counting Algorithm|
|_fic_area_size|131072|TRUE|size of Frequent Itemset Counting work area|
|_fic_max_length|20|TRUE|Frequent Itemset Counting Maximum Itemset Length|
|_fic_min_bmsize|1024|TRUE|Frequent Itemset Counting Minimum BITMAP Size|
|_fic_outofmem_candidates|FALSE|TRUE|Frequent Itemset Counting Out Of Memory Candidates Generation|
|_fifth_spare_parameter|%NULL%|TRUE|fifth spare parameter - string|
|_file_size_increase_increment|52428800|TRUE|Amount of file size increase increment, in bytes|
|_filemap_dir|%NULL%|TRUE|FILEMAP directory|
|_first_k_rows_dynamic_proration|TRUE|FALSE|enable the use of dynamic proration of join cardinalities|
|_first_spare_parameter|%NULL%|TRUE|first spare parameter - integer|
|_fix_control|%NULL%|TRUE|bug fix control parameter|
|_flashback_11.1_block_new_opt|FALSE|TRUE|use 11.1 flashback block new optimization scheme|
|_flashback_allow_noarchivelog|FALSE|TRUE|Allow enabling flashback on noarchivelog database|
|_flashback_archiver_partition_size|0|TRUE|flashback archiver table partition size|
|_flashback_barrier_interval|1800|TRUE|Flashback barrier interval in seconds|
|_flashback_copy_latches|10|TRUE|Number of flashback copy latches|
|_flashback_database_test_only|FALSE|TRUE|Run Flashback Database in test mode|
|_flashback_dynamic_enable|TRUE|TRUE|enable flashback enable code path|
|_flashback_dynamic_enable_failure|0|TRUE|Simulate failures during dynamic enable|
|_flashback_format_chunk_mb|4|TRUE|Chunk mega-bytes for formatting flashback logs using sync write|
|_flashback_format_chunk_mb_dwrite|16|TRUE|Chunk mega-bytes for formatting flashback logs using delayed write|
|_flashback_fuzzy_barrier|TRUE|TRUE|Use flashback fuzzy barrier|
|_flashback_generation_buffer_size|21045248|TRUE|flashback generation buffer size|
|_flashback_hint_barrier_percent|20|TRUE|Flashback hint barrier percent|
|_flashback_log_io_error_behavior|0|TRUE|Specify Flashback log I/O error behavior|
|_flashback_log_min_size|100|TRUE|Minimum flashback log size|
|_flashback_log_rac_balance_factor|10|TRUE|flashback log rac balance factor|
|_flashback_log_size|1000|TRUE|Flashback log size|
|_flashback_logfile_enqueue_timeout|600|TRUE|flashback logfile enqueue timeout for opens|
|_flashback_marker_cache_enabled|TRUE|TRUE|Enable flashback database marker cache|
|_flashback_marker_cache_size|328|TRUE|Size of flashback database marker cache|
|_flashback_max_log_size|0|TRUE|Maximum flashback log size in bytes (OS limit)|
|_flashback_max_n_log_per_thread|2048|TRUE|Maximum number of flashback logs per flashback thread|
|_flashback_max_standby_sync_span|300|TRUE|Maximum time span between standby recovery sync for flashback|
|_flashback_n_log_per_thread|128|TRUE|Desired number of flashback logs per flashback thread|
|_flashback_standby_barrier_interval|1|TRUE|Flashback standby barrier interval in seconds|
|_flashback_verbose_info|FALSE|TRUE|Print verbose information about flashback database|
|_flashback_write_max_loop_limit|10|TRUE|Flashback writer loop limit before it returns|
|_flush_redo_to_standby|0|TRUE|Flush redo to standby test event parameter|
|_flush_undo_after_tx_recovery|TRUE|TRUE|if TRUE, flush undo buffers after TX recovery|
|_force_arch_compress|0|TRUE|Archive Compress all newly created compressed tables|
|_force_datefold_trunc|FALSE|TRUE|force use of trunc for datefolding rewrite|
|_force_hsc_compress|FALSE|TRUE|compress all newly created tables|
|_force_oltp_compress|FALSE|TRUE|OLTP Compress all newly created compressed tables|
|_force_rcv_info_ping|0|TRUE|Force recovery info ping to stdby|
|_force_rewrite_enable|FALSE|TRUE|control new query rewrite features|
|_force_slave_mapping_intra_part_loads|FALSE|TRUE|Force slave mapping for intra partition loads|
|_force_temptables_for_gsets|FALSE|TRUE|executes concatenation of rollups using temp tables|
|_force_tmp_segment_loads|FALSE|TRUE|Force tmp segment loads|
|_forwarded_2pc_threshold|10|TRUE|auto-tune threshold for two-phase commit rate across RAC instances|
|_fourth_spare_parameter|%NULL%|TRUE|fourth spare parameter - string|
|_frame_cache_time|0|TRUE|number of seconds a cached frame page stay in cache.|
|_full_pwise_join_enabled|TRUE|TRUE|enable full partition-wise join when TRUE|
|_fusion_security|FALSE|TRUE|Fusion Security|
|_gby_hash_aggregation_enabled|TRUE|FALSE|enable group-by and aggregation using hash scheme|
|_gby_onekey_enabled|TRUE|TRUE|enable use of one comparison of all group by keys|
|_gc_affinity_locking|TRUE|TRUE|if TRUE, enable object affinity|
|_gc_affinity_locks|TRUE|TRUE|if TRUE, get affinity locks|
|_gc_affinity_ratio|50|TRUE|dynamic object affinity ratio|
|_gc_async_memcpy|FALSE|TRUE|if TRUE, use async memcpy|
|_gc_bypass_readers|TRUE|TRUE|if TRUE, modifications bypass readers|
|_gc_check_bscn|TRUE|TRUE|if TRUE, check for stale blocks|
|_gc_coalesce_recovery_reads|TRUE|TRUE|if TRUE, coalesce recovery reads|
|_gc_cr_server_read_wait|TRUE|TRUE|if TRUE, cr server waits for a read to complete|
|_gc_defer_ping_index_only|TRUE|TRUE|if TRUE, restrict deferred ping to index blocks only|
|_gc_defer_time|3|TRUE|how long to defer pings for hot buffers in milliseconds|
|_gc_delta_push_compression|3072|TRUE|if delta >= K bytes, compress before push|
|_gc_delta_push_max_level|100|TRUE|max delta level for delta push|
|_gc_delta_push_objects|0|TRUE|objects which use delta push|
|_gc_down_convert_after_keep|TRUE|TRUE|if TRUE, down-convert lock after recovery|
|_gc_element_percent|110|TRUE|global cache element percent|
|_gc_flush_during_affinity|TRUE|TRUE|if TRUE, flush during affinity|
|_gc_fusion_compression|1536|TRUE|compress fusion blocks if there is free space|
|_gc_global_checkpoint_scn|TRUE|TRUE|if TRUE, enable global checkpoint scn|
|_gc_global_lru|AUTO|TRUE|turn global lru off, make it automatic, or turn it on|
|_gc_global_lru_touch_count|5|TRUE|global lru touch count|
|_gc_global_lru_touch_time|60|TRUE|global lru touch time in seconds|
|_gc_integrity_checks|1|TRUE|set the integrity check level|
|_gc_keep_recovery_buffers|TRUE|TRUE|if TRUE, make single instance crash recovery buffers current|
|_gc_latches|8|TRUE|number of latches per LMS process|
|_gc_long_query_threshold|0|TRUE|threshold for long running query|
|_gc_max_downcvt|256|TRUE|maximum downconverts to process at one time|
|_gc_maximum_bids|0|TRUE|maximum number of bids which can be prepared|
|_gc_policy_minimum|1500|TRUE|dynamic object policy minimum activity per minute|
|_gc_policy_time|10|TRUE|how often to make object policy decisions in minutes|
|_gc_read_mostly_flush_check|FALSE|TRUE|if TRUE, optimize flushes for read mostly objects|
|_gc_read_mostly_locking|TRUE|TRUE|if TRUE, enable read-mostly locking|
|_gc_statistics|TRUE|TRUE|if TRUE, kcl statistics are maintained|
|_gc_transfer_ratio|2|TRUE|dynamic object read-mostly transfer ratio|
|_gc_tsn_undo_affinity|TRUE|TRUE|if TRUE, use TSN undo affinity|
|_gc_undo_affinity|TRUE|TRUE|if TRUE, enable dynamic undo affinity|
|_gc_undo_block_disk_reads|TRUE|TRUE|if TRUE, enable undo block disk reads|
|_gc_use_cr|TRUE|TRUE|if TRUE, allow CR pins on PI and WRITING buffers|
|_gc_vector_read|TRUE|TRUE|if TRUE, vector read current buffers|
|_gcs_disable_remote_handles|FALSE|TRUE|disable remote client/shadow handles|
|_gcs_fast_reconfig|TRUE|TRUE|if TRUE, enable fast reconfiguration for gcs locks|
|_gcs_latches|0|TRUE|number of gcs resource hash latches to be allocated per LMS process|
|_gcs_pkey_history|4000|TRUE|number of pkey remastering history|
|_gcs_process_in_recovery|TRUE|TRUE|if TRUE, process gcs requests during instance recovery|
|_gcs_res_per_bucket|16|TRUE|number of gcs resource per hash bucket|
|_gcs_resources|%NULL%|TRUE|number of gcs resources to be allocated|
|_gcs_shadow_locks|%NULL%|TRUE|number of pcm shadow locks to be allocated|
|_gcs_testing|0|TRUE|GCS testing parameter|
|_generalized_pruning_enabled|TRUE|FALSE|controls extensions to partition pruning for general predicates|
|_ges_dd_debug|1|TRUE|if 1 or higher enables GES deadlock detection debug diagnostics|
|_ges_diagnostics|TRUE|TRUE|if TRUE enables GES diagnostics|
|_ges_diagnostics_asm_dump_level|11|TRUE|systemstate level on global enqueue diagnostics blocked by ASM|
|_ges_health_check|1|TRUE|if greater than 0 enables GES system health check|
|_ges_num_blockers_to_kill|1|TRUE|number of blockers to be killed for hang resolution|
|_global_hang_analysis_interval_secs|10|TRUE|the interval at which global hang analysis is run|
|_globalindex_pnum_filter_enabled|TRUE|FALSE|enables filter for global index with partition extended syntax|
|_groupby_nopushdown_cut_ratio|3|TRUE|groupby nopushdown cut ratio|
|_groupby_orderby_combine|5000|TRUE|groupby/orderby don't combine threshold|
|_gs_anti_semi_join_allowed|TRUE|FALSE|enable anti/semi join for the GS query|
|_hang_analysis_num_call_stacks|3|TRUE|hang analysis num call stacks|
|_hang_detection|30|TRUE|Hang Management detection interval|
|_hang_ignored_hangs_interval|30|TRUE|Time ignored hangs must persist after initial verification|
|_hang_log_incidents|FALSE|TRUE|Hang Manager incident logging|
|_hang_long_wait_time_threshold|0|TRUE|Long session wait time threshold in seconds|
|_hang_lws_file_count|5|TRUE|Number of trace files for long waiting sessions|
|_hang_lws_file_time_limit|1800|TRUE|Amount of time to save long waiting session traces to file|
|_hang_msg_checksum_enabled|TRUE|TRUE|enable hang graph message checksum|
|_hang_output_suspected_hangs|FALSE|TRUE|Hang Management enabled output of suspected hangs|
|_hang_resolution_confidence_promotion|FALSE|TRUE|Hang Management hang resolution confidence promotion|
|_hang_resolution_policy|HIGH|TRUE|Hang Management hang resolution policy|
|_hang_resolution_scope|OFF|TRUE|Hang Management hang resolution scope|
|_hang_signature_list_output_frequency|25|TRUE|Hang Signature List output frequency|
|_hang_statistics_collection_interval|0|TRUE|Hang Management statistics collection interval in seconds|
|_hang_statistics_collection_react_ma_alpha|30|TRUE|Hang Management statistics collection reactive moving average alpha|
|_hang_statistics_collection_react_ma_sum_delta_cnt|15|TRUE|Hang Management statistics collection react moving average sum of deltas count|
|_hang_statistics_collection_smooth_ma_alpha|8|TRUE|Hang Management statistics collection smoothed moving average alpha|
|_hard_protection|FALSE|TRUE|if TRUE enable H.A.R.D specific format changes|
|_hash_join_enabled|TRUE|TRUE|enable/disable hash join|
|_hash_multiblock_io_count|0|TRUE|number of blocks hash join will read/write at once|
|_heur_deadlock_resolution_secs|0|TRUE|the heuristic wait time per node for deadlock resolution|
|_high_priority_process_num_yields_before_sleep|1000|TRUE|the number of yields performed by high priority processesbefore they sleep|
|_high_priority_processes|LMS*&#124;VKTM|TRUE|High Priority Process Name Mask|
|_high_server_threshold|0|TRUE|high server thresholds|
|_highthreshold_undoretention|4294967294|TRUE|high threshold undo_retention in seconds|
|_hj_bit_filter_threshold|50|TRUE|hash-join bit filtering threshold (0 always enabled)|
|_hm_analysis_oradebug_node_dump_level|0|TRUE|the oradebug node dump level for hang manager hang analysis|
|_hm_analysis_oradebug_sys_dump_level|0|TRUE|the oradebug system state level for hang manager hang analysis|
|_hm_analysis_output_disk|FALSE|TRUE|if TRUE the hang manager outputs hang analysis results to disk|
|_hm_verification_interval|30|TRUE|the hang manager verification interval|
|_hwm_sync_threshold|10|TRUE|HWM synchronization threshold in percentage|
|_idl_conventional_index_maintenance|TRUE|TRUE|enable conventional index maintenance for insert direct load|
|_idle_session_kill_enabled|TRUE|TRUE|enables or disables resource manager session idle limit checks|
|_idxrb_rowincr|100000000|TRUE|proportionality constant for dop vs. rows in index rebuild|
|_ignore_desc_in_index|FALSE|TRUE|ignore DESC in indexes, sort those columns ascending anyhow|
|_ignore_fg_deps|%NULL%|TRUE|ignore fine-grain dependencies during invalidation|
|_immediate_commit_propagation|TRUE|TRUE|if TRUE, propagate commit SCN immediately|
|_improved_outerjoin_card|TRUE|FALSE|improved outer-join cardinality calculation|
|_improved_row_length_enabled|TRUE|FALSE|enable the improvements for computing the average row length|
|_imr_active|TRUE|TRUE|Activate Instance Membership Recovery feature|
|_imr_avoid_double_voting|TRUE|TRUE|Avoid device voting for CSS reconfig during IMR|
|_imr_device_type|controlfile|TRUE|Type of device to be used by IMR|
|_imr_disk_voting_interval|3|TRUE|Maximum wait for IMR disk voting (seconds)|
|_imr_diskvote_implementation|auto|TRUE|IMR disk voting implementation method|
|_imr_evicted_member_kill|TRUE|TRUE|IMR issue evicted member kill after a wait|
|_imr_evicted_member_kill_wait|20|TRUE|IMR evicted member kill wait time in seconds|
|_imr_highload_threshold|%NULL%|TRUE|IMR system highload threshold|
|_imr_max_reconfig_delay|75|TRUE|Maximum Reconfiguration delay (seconds)|
|_imr_splitbrain_res_wait|0|TRUE|Maximum wait for split-brain resolution (seconds)|
|_imr_systemload_check|TRUE|TRUE|Perform the system load check during IMR|
|_imu_pools|3|TRUE|in memory undo pools|
|_in_memory_tbs_search|TRUE|TRUE|FALSE - disable fast path for alter tablespace read only|
|_in_memory_undo|TRUE|TRUE|Make in memory undo for top level transactions|
|_incremental_recovery_ckpt_min_batch|500|TRUE|minimum number of writes for incremental recovery ckpt every second|
|_index_join_enabled|TRUE|FALSE|enable the use of index joins|
|_index_prefetch_factor|100|TRUE|index prefetching factor|
|_index_scan_check_skip_corrupt|FALSE|TRUE|check and skip corrupt blocks during index scans|
|_init_granule_interval|10|TRUE|number of granules to process for deferred cache|
|_init_sql_file|?/rdbms/admin/sql.bsq|TRUE|File containing SQL statements to execute upon database creation|
|_inject_startup_fault|0|TRUE|inject fault in the startup code|
|_inline_sql_in_plsql|FALSE|TRUE|inline SQL in PL/SQL|
|_inquiry_retry_interval|3|TRUE|if greater than 0 enables inquiry retry after specified interval|
|_insert_enable_hwm_brokered|TRUE|TRUE|during parallel inserts high water marks are brokered|
|_inst_locking_period|5|TRUE|period an instance can retain a newly acquired level1 bitmap|
|_interconnect_checksum|TRUE|TRUE|if TRUE, checksum interconnect blocks|
|_intrapart_pdml_enabled|TRUE|TRUE|Enable intra-partition updates/deletes|
|_intrapart_pdml_randomlocal_enabled|TRUE|TRUE|Enable intra-partition updates/deletes with random local dist|
|_io_resource_manager_always_on|FALSE|TRUE|io resource manager always on|
|_io_shared_pool_size|4194304|TRUE|Size of I/O buffer pool from SGA|
|_io_slaves_disabled|FALSE|TRUE|Do not use I/O slaves|
|_io_statistics|TRUE|TRUE|if TRUE, ksfd I/O statistics are collected|
|_ioq_fanin_multiplier|2|TRUE|IOQ miss count before a miss exception|
|_ior_serialize_fault|0|TRUE|inject fault in the ior serialize code|
|_iorm_tout|1000|TRUE|IORM scheduler timeout value in msec|
|_ioslave_batch_count|1|TRUE|Per attempt IOs picked|
|_ioslave_issue_count|500|TRUE|IOs issued before completion check|
|_ipc_fail_network|0|TRUE|Simulate cluster network failer|
|_ipc_test_failover|0|TRUE|Test transparent cluster network failover|
|_ipc_test_mult_nets|0|TRUE|simulate multiple cluster networks|
|_ipddb_enable|FALSE|TRUE|Enable IPD/DB data collection|
|_job_queue_interval|5|TRUE|Wakeup interval in seconds for job queue co-ordinator|
|_k2q_latches|0|TRUE|number of k2q latches|
|_kcfis_caching_enabled|TRUE|TRUE|enable kcfis intra-scan session caching|
|_kcfis_cell_passthru_enabled|FALSE|TRUE|Do not perform smart IO filtering on the cell|
|_kcfis_control1|0|TRUE|Kcfis control1|
|_kcfis_control2|0|TRUE|Kcfis control2|
|_kcfis_control3|0|TRUE|Kcfis control3|
|_kcfis_control4|0|TRUE|Kcfis control4|
|_kcfis_control5|0|TRUE|Kcfis control5|
|_kcfis_control6|0|TRUE|Kcfis control6|
|_kcfis_fault_control|0|TRUE|trace level setting for KCFIS module|
|_kcfis_max_cached_sessions|10|TRUE|Sets the maximum number of kcfis sessions cached|
|_kcfis_metadata_io_write_enabled|TRUE|TRUE|enables writes that combine metadata and IO requests|
|_kcfis_oss_io_size|0|TRUE|KCFIS OSS I/O size|
|_kcfis_rdbms_blockio_enabled|FALSE|TRUE|Use block IO instead of smart IO in the smart IO module on RDBMS|
|_kcfis_read_buffer_limit|0|TRUE|KCFIS Read Buffer (per session) memory limit in bytes|
|_kcfis_reuse_any_cached_enabled|TRUE|TRUE|enable kcfis reuse of a kcfis caching entry with key mismatch|
|_kcfis_stats_level|0|TRUE|sets kcfis stats level|
|_kcfis_storageidx_diag_mode|FALSE|TRUE|Debug mode for storage index on the cell|
|_kcfis_storageidx_disabled|FALSE|TRUE|Don't use storage index optimization on the storage cell|
|_kcfis_trace_level|0|TRUE|sets kcfis tracing level|
|_kcfis_work_set_appliances|2|TRUE|Working Set of appliances in a KCFIS session|
|_kcl_commit|TRUE|TRUE|if TRUE, call kjbcommit|
|_kcl_conservative_log_flush|FALSE|TRUE|if TRUE, conservatively log flush before CR serving|
|_kcl_debug|TRUE|TRUE|if TRUE, record le history|
|_kcl_index_split|TRUE|TRUE|if TRUE, reject pings on blocks in middle of a split|
|_kd_symtab_chk|TRUE|TRUE|enable or disable symbol table integrity block check|
|_kdbl_enable_post_allocation|FALSE|TRUE|allocate dbas after populating data buffers|
|_kdi_avoid_block_checking|FALSE|TRUE|avoid index block checking on sensitive opcodes|
|_kdis_reject_level|24|TRUE|b+tree level to enable rejection limit|
|_kdis_reject_limit|5|TRUE|#block rejections in space reclamation before segment extension|
|_kdis_reject_ops|FALSE|TRUE|enable rejection heuristic for branch splits|
|_kdli_STOP_bsz|0|TRUE|undocumented parameter for internal use only|
|_kdli_STOP_dba|0|TRUE|undocumented parameter for internal use only|
|_kdli_STOP_fsz|0|TRUE|undocumented parameter for internal use only|
|_kdli_STOP_nio|0|TRUE|undocumented parameter for internal use only|
|_kdli_STOP_tsn|0|TRUE|undocumented parameter for internal use only|
|_kdli_allow_corrupt|TRUE|TRUE|allow corrupt filesystem_logging data blocks during read/write|
|_kdli_buffer_inject|TRUE|TRUE|use buffer injection for CACHE [NO]LOGGING lobs|
|_kdli_cache_inode|TRUE|TRUE|cache inode state across calls|
|_kdli_cache_read_threshold|0|TRUE|minimum lob size for cache->nocache read (0 disables heuristic)|
|_kdli_cache_size|8|TRUE|maximum #entries in inode cache|
|_kdli_cache_verify|FALSE|TRUE|verify cached inode via deserialization|
|_kdli_cache_write_threshold|0|TRUE|minimum lob size for cache->nocache write (0 disables heuristic)|
|_kdli_cacheable_length|0|TRUE|minimum lob length for inode cacheability|
|_kdli_checkpoint_flush|FALSE|TRUE|do not invalidate cache buffers after write|
|_kdli_dbc|none|TRUE|override db_block_checking setting for securefiles|
|_kdli_delay_flushes|TRUE|TRUE|delay flushing cache writes to direct-write lobs|
|_kdli_flush_cache_reads|TRUE|TRUE|flush cache-reads data blocks after load|
|_kdli_flush_injections|TRUE|TRUE|flush injected buffers of CACHE NOLOGGING lobs before commit|
|_kdli_force_cr|TRUE|TRUE|force CR when reading data blocks of direct-write lobs|
|_kdli_force_cr_meta|TRUE|TRUE|force CR when reading metadata blocks of direct-write lobs|
|_kdli_force_storage|none|TRUE|force storage settings for all lobs|
|_kdli_full_readahead_threshold|0|TRUE|maximum lob size for full readahead|
|_kdli_inject_assert|0|TRUE|inject asserts into the inode|
|_kdli_inject_batch|0|TRUE|buffer injection batch size [1, KCBNEWMAX]|
|_kdli_inject_crash|0|TRUE|inject crashes into the inode|
|_kdli_inline_xfm|TRUE|TRUE|allow inline transformed lobs|
|_kdli_inode_preference|data|TRUE|inline inode evolution preference (data, headless, lhb)|
|_kdli_itree_entries|0|TRUE|#entries in lhb/itree blocks (for testing only)|
|_kdli_memory_protect|FALSE|TRUE|trace accesses to inode memory outside kdli API functions|
|_kdli_rci_lobmap_entries|255|TRUE|#entries in RCI lobmap before migration to lhb|
|_kdli_readahead_limit|0|TRUE|shared/cached IO readahead limit|
|_kdli_readahead_strategy|contig|TRUE|shared/cached IO readahead strategy|
|_kdli_recent_scn|FALSE|TRUE|use recent (not dependent) scns for block format/allocation|
|_kdli_reshape|FALSE|TRUE|reshape an inode to inline or headless on length truncation|
|_kdli_safe_callbacks|TRUE|TRUE|invoke inode read/write callbacks safely|
|_kdli_sio_async|TRUE|TRUE|asynchronous shared IO|
|_kdli_sio_backoff|FALSE|TRUE|use exponential backoff when attempting SIOP allocations|
|_kdli_sio_bps|0|TRUE|maximum blocks per IO slot|
|_kdli_sio_dop|2|TRUE|degree-of-parallelism in the SIO keep pool|
|_kdli_sio_fgio|TRUE|TRUE|reap asynchronous IO in the foreground|
|_kdli_sio_fileopen|none|TRUE|shared IO fileopen mode: datasync vs nodatasync vs async|
|_kdli_sio_flush|FALSE|TRUE|enable shared IO pool operations|
|_kdli_sio_free|TRUE|TRUE|free IO buffers when not in active use|
|_kdli_sio_min_read|0|TRUE|shared IO pool read threshold|
|_kdli_sio_min_write|0|TRUE|shared IO pool write threshold|
|_kdli_sio_nbufs|8|TRUE|maximum #IO buffers to allocate per session|
|_kdli_sio_niods|8|TRUE|maximum #IO descriptors to allocate per session|
|_kdli_sio_on|TRUE|TRUE|enable shared IO pool operations|
|_kdli_sio_pga|FALSE|TRUE|use PGA allocations for direct IO|
|_kdli_sio_pga_top|FALSE|TRUE|PGA allocations come from toplevel PGA heap|
|_kdli_sio_strategy|extent|TRUE|shared IO strategy: block vs. extent|
|_kdli_small_cache_limit|32|TRUE|size limit of small inode cache|
|_kdli_sort_dbas|FALSE|TRUE|sort dbas during chunkification|
|_kdli_space_cache_limit|2048|TRUE|maximum size of the space cache in #blocks|
|_kdli_squeeze|TRUE|TRUE|compact lobmap extents with contiguous dbas|
|_kdli_timer_dmp|FALSE|TRUE|dump inode timers on session termination|
|_kdli_timer_trc|FALSE|TRUE|trace inode timers to uts/tracefile|
|_kdli_trace|0|TRUE|inode trace level|
|_kdlu_max_bucket_size|4194304|TRUE|UTS kdlu bucket size|
|_kdlu_max_bucket_size_mts|131072|TRUE|UTS kdlu bucket size for mts|
|_kdlu_trace_layer|0|TRUE|UTS kdlu per-layer trace level|
|_kdlu_trace_session|0|TRUE|UTS session dump|
|_kdlu_trace_system|0|TRUE|UTS system dump|
|_kdlw_enable_ksi_locking|FALSE|TRUE|enable ksi locking for lobs|
|_kdlw_enable_write_gathering|TRUE|TRUE|enable lob write gathering for sql txns|
|_kdlwp_flush_threshold|4194304|TRUE|WGC flush threshold in bytes|
|_kdlxp_cmp_subunit_size|262144|TRUE|size of compression sub-unit in bytes|
|_kdlxp_dedup_flush_threshold|8388608|TRUE|deduplication flush threshold in bytes|
|_kdlxp_dedup_hash_algo|SHA1|TRUE|secure hash algorithm for deduplication - only on SecureFiles|
|_kdlxp_dedup_inl_pctfree|5|TRUE|deduplication pct size increase by which inlining avoided|
|_kdlxp_dedup_prefix_threshold|1048576|TRUE|deduplication prefix hash threshold in bytes|
|_kdlxp_lobcmpadp|FALSE|TRUE|enable adaptive compression - only on SecureFiles|
|_kdlxp_lobcmplevel|2|TRUE|Default securefile compression|
|_kdlxp_lobcmprciver|1|TRUE|Default securefile compression map version|
|_kdlxp_lobcompress|FALSE|TRUE|enable lob compression - only on SecureFiles|
|_kdlxp_lobdeduplicate|FALSE|TRUE|enable lob deduplication - only on SecureFiles|
|_kdlxp_lobdedupvalidate|TRUE|TRUE|enable deduplicate validate - only on SecureFiles|
|_kdlxp_lobencrypt|FALSE|TRUE|enable lob encryption - only on SecureFiles|
|_kdlxp_mincmp|20|TRUE|minimum comp ratio in pct - only on SecureFiles|
|_kdlxp_minxfm_size|32768|TRUE|minimum transformation size in bytes|
|_kdlxp_spare1|0|TRUE|deduplication spare 1|
|_kdlxp_xfmcache|TRUE|TRUE|enable xfm cache - only on SecureFiles|
|_kdt_buffering|TRUE|TRUE|control kdt buffering for conventional inserts|
|_kdu_array_depth|16|TRUE|array update retry recursion depth limits|
|_kdz_hcc_flags|0|TRUE|Miscellaneous HCC flags|
|_kdz_hcc_track_upd_rids|TRUE|TRUE|Enable rowid tracking during updates|
|_kebm_nstrikes|3|TRUE|kebm # strikes to auto suspend an action|
|_kebm_suspension_time|82800|TRUE|kebm auto suspension time in seconds|
|_keep_remote_column_size|FALSE|TRUE|remote column size does not get modified|
|_kernel_message_network_driver|FALSE|TRUE|kernel message network driver|
|_kffmap_hash_size|1024|TRUE|size of kffmap_hash table|
|_kffmop_hash_size|2048|TRUE|size of kffmop_hash table|
|_kfm_disable_set_fence|FALSE|TRUE|disable set fence calls and revert to default (process fence)|
|_kghdsidx_count|1|TRUE|max kghdsidx count|
|_kgl_bucket_count|9|TRUE|Library cache hash table bucket count (2^_kgl_bucket_count * 256)|
|_kgl_debug|%NULL%|TRUE|Library cache debugging|
|_kgl_fixed_extents|TRUE|TRUE|fixed extent size for library cache memory allocations|
|_kgl_hash_collision|FALSE|TRUE|Library cache name hash collision possible|
|_kgl_heap_size|4096|TRUE|extent size for library cache heap 0|
|_kgl_kqr_cap_so_stacks|FALSE|TRUE|capture stacks for library and row cache state objects|
|_kgl_large_heap_warning_threshold|52428800|TRUE|maximum heap size before KGL writes warnings to the alert log|
|_kgl_latch_count|0|TRUE|number of library cache latches|
|_kgl_message_locks|64|TRUE|RAC message lock count|
|_kgl_min_cached_so_count|1|TRUE|Minimum cached SO count. If > 1 can help find SO corruptions|
|_kgl_multi_instance_invalidation|TRUE|TRUE|whether KGL to support multi-instance invalidations|
|_kgl_multi_instance_lock|TRUE|TRUE|whether KGL to support multi-instance locks|
|_kgl_multi_instance_pin|TRUE|TRUE|whether KGL to support multi-instance pins|
|_kgl_mutex_wait_time|0|TRUE|KGL mutex wait time|
|_kgl_time_to_wait_for_locks|15|TRUE|time to wait for locks and pins before timing out|
|_kglsim_maxmem_percent|5|TRUE|max percentage of shared pool size to be used for KGL advice|
|_kgsb_threshold_size|16777216|TRUE|threshold size for base allocator|
|_kgx_latches|1024|TRUE|# of mutex latches if CAS is not supported.|
|_kgx_spin_count|255|TRUE|Mutex spin count|
|_kill_controlfile_enqueue_blocker|TRUE|TRUE|enable killing controlfile enqueue blocker on timeout|
|_kill_diagnostics_timeout|60|TRUE|timeout delay in seconds before killing enqueue blocker|
|_kill_enqueue_blocker|2|TRUE|if greater than 0 enables killing enqueue blocker|
|_kill_java_threads_on_eoc|FALSE|TRUE|Kill Java threads and do sessionspace migration at end of call|
|_kill_session_dump|TRUE|TRUE|Process dump on kill session immediate|
|_kkfi_trace|FALSE|TRUE|trace expression substitution|
|_kks_free_cursor_stat_pct|10|TRUE|percentage of cursor stats buckets to scan on each load, in 1/10th of a percent|
|_kokli_cache_size|128|TRUE|Size limit of ADT Table Lookup Cache|
|_kolfuseslf|FALSE|TRUE|allow kolf to use slffopen|
|_kql_subheap_trace|0|TRUE|tracing level for library cache subheap level pins|
|_ksb_restart_clean_time|30000|TRUE|process uptime for restarts|
|_ksb_restart_policy_times|0, 60, 120, 240|FALSE|process restart policy times in seconds|
|_ksd_test_param|999|TRUE|KSD test parmeter|
|_ksdx_charset_ratio|0|TRUE|ratio between the system and oradebug character set|
|_ksdxdocmd_default_timeout_ms|30000|TRUE|default timeout for internal oradebug commands|
|_ksdxdocmd_enabled|TRUE|TRUE|if TRUE ksdxdocmd* invocations are enabled|
|_ksdxw_cini_flg|0|TRUE|ksdxw context initialization flag|
|_ksdxw_nbufs|1000|TRUE|ksdxw number of buffers in buffered mode|
|_ksdxw_num_pgw|10|TRUE|number of watchpoints on a per-process basis|
|_ksdxw_num_sgw|10|TRUE|number of watchpoints to be shared by all processes|
|_ksdxw_stack_depth|4|TRUE|number of PCs to collect in the stack when watchpoint is hit|
|_kse_die_timeout|60000|TRUE|amount of time a dying process is spared by PMON (in centi-secs)|
|_kse_pc_table_size|256|TRUE|kse pc table cache size|
|_kse_signature_entries|0|TRUE|number of entries in the kse stack signature cache|
|_kse_signature_limit|7|TRUE|number of stack frames to cache per kse signature|
|_kse_snap_ring_size|0|TRUE|ring buffer to debug internal error 17090|
|_kse_trace_int_msg_clear|FALSE|TRUE|enables soft assert of KGECLEAERERROR is cleares an interrupt message|
|_ksfd_verify_write|FALSE|TRUE|verify asynchronous writes issued through ksfd|
|_ksi_clientlocks_enabled|TRUE|TRUE|if TRUE, DLM-clients can provide the lock memory|
|_ksi_trace|%NULL%|TRUE|KSI trace string of lock type(s)|
|_ksi_trace_bucket|PRIVATE|TRUE|memory tracing: use ksi-private or rdbms-shared bucket|
|_ksi_trace_bucket_size|8192|TRUE|size of the KSI trace bucket|
|_ksmb_debug|0|TRUE|ksmb debug flags|
|_ksmd_protect_mode|off|TRUE|KSMD protect mode for catching stale access|
|_ksmg_granule_locking_status|1|TRUE|granule locking status|
|_ksmg_granule_size|4194304|TRUE|granule size in bytes|
|_ksmg_lock_check_interval|%NULL%|TRUE|timeout action interval in minutes|
|_ksmg_lock_reacquire_count|5|TRUE|repeat count for acquisition of locks|
|_kspol_tac_timeout|5|TRUE|timeouts for TAC registerd by kspol|
|_ksr_unit_test_processes|0|TRUE|number of ksr unit test processes|
|_kss_quiet|FALSE|TRUE|if TRUE access violations during kss dumps are not recorded|
|_ksu_diag_kill_time|5|TRUE|number of seconds ksuitm waits before killing diag|
|_ksuitm_addon_trccmd|%NULL%|TRUE|command to execute when dead processes don't go away|
|_ksuitm_dont_kill_dumper|FALSE|TRUE|delay inst. termination to allow processes to dump|
|_ksv_dynamic_flags1|0|TRUE|ksv dynamic flags 1 - override default behavior|
|_ksv_max_spawn_fail_limit|5|TRUE|bg slave spawn failure limit|
|_ksv_pool_hang_kill_to|0|TRUE|bg slave pool terminate timeout|
|_ksv_pool_wait_timeout|600|TRUE|bg slave pool wait limit|
|_ksv_spawn_control_all|FALSE|TRUE|control all spawning of background slaves|
|_ksv_static_flags1|0|TRUE|ksv static flags 1 - override default behavior|
|_ksvppktmode|0|TRUE|ksv internal pkt test|
|_ksxp_compat_flags|0|TRUE|ksxp compat flags|
|_ksxp_diagmode|OFF|TRUE|set to OFF to disable automatic slowsend diagnostics|
|_ksxp_disable_clss|0|TRUE|disable CLSS interconnects|
|_ksxp_disable_dynamic_loading|FALSE|TRUE|disable dynamic loadin of lib skgxp|
|_ksxp_disable_ipc_stats|FALSE|TRUE|disable ipc statistics|
|_ksxp_disable_rolling_migration|FALSE|TRUE|disable possibility of starting rolling migration|
|_ksxp_dump_timeout|20|TRUE|set timeout for kjzddmp request|
|_ksxp_dynamic_skgxp_param|%NULL%|TRUE|dynamic skgxp parameters|
|_ksxp_if_config|0|TRUE|ksxp if config flags|
|_ksxp_init_stats_bkts|0|TRUE|initial number arrays for ipc statistics|
|_ksxp_max_stats_bkts|0|TRUE|max. arrays for ipc statistics|
|_ksxp_ping_enable|TRUE|TRUE|disable dynamic loadin of lib skgxp|
|_ksxp_ping_polling_time|0|TRUE|max. arrays for ipc statistics|
|_ksxp_reaping|20|TRUE|tune ksxp layer reaping limit|
|_ksxp_reporting_process|LMD0|TRUE|reporting process for KSXP|
|_ksxp_send_timeout|300|TRUE|set timeout for sends queued with the inter-instance IPC|
|_ksxp_skgxp_compat_library_path|%NULL%|TRUE|over-ride default location of lib skgxp compat|
|_ksxp_skgxp_ctx_flags1|0|TRUE|IPC debug options flags (RAC)|
|_ksxp_skgxp_ctx_flags1mask|0|TRUE|IPC debug options flags mask (RAC)|
|_ksxp_skgxp_dynamic_protocol|4096|TRUE|IPC protocol override (RAC) (0/-1=*,2=UDP,3=RDS,!0x1000=ipc_X)|
|_ksxp_skgxp_library_path|%NULL%|TRUE|over-ride default location of lib skgxp|
|_ksxp_skgxp_rgn_ports|0|TRUE|region socket limits (0xFFFFNNXX): F=flags, N=min, X=max|
|_ksxp_skgxp_spare_param1|%NULL%|TRUE|ipc ksxp spare parameter 1|
|_ksxp_skgxp_spare_param2|%NULL%|TRUE|ipc ksxp spare parameter 2|
|_ksxp_skgxp_spare_param3|%NULL%|TRUE|ipc ksxp spare parameter 3|
|_ksxp_skgxp_spare_param4|%NULL%|TRUE|ipc ksxp spare parameter 4|
|_ksxp_skgxp_spare_param5|%NULL%|TRUE|ipc ksxp spare parameter 5|
|_ksxp_skgxpg_last_parameter|26|TRUE|last defined skgxpg parameter - ksxp|
|_ksxp_stats_mem_lmt|0|TRUE|limit ipc statistics memory. this parameter is a percentage value|
|_ksxp_testing|0|TRUE|KSXP test parameter|
|_ksxp_unit_test_byte_transformation|FALSE|TRUE|enable byte transformation unit test|
|_ktb_debug_flags|0|TRUE|ktb-layer debug flags|
|_ktc_debug|0|TRUE|for ktc debug|
|_ktc_latches|0|TRUE|number of ktc latches|
|_ktslj_segext_max_mb|0|TRUE|segment pre-extension max size in MB (0: unlimited)|
|_ktslj_segext_retry|5|TRUE|segment pre-extension retry|
|_ktslj_segext_warning|10|TRUE|segment pre-extension warning threshold in percentage|
|_ktslj_segext_warning_mb|0|TRUE|segment pre-extension warning threshold in MB|
|_ktspsrch_maxsc|1024|TRUE|maximum segments supported by space search cache|
|_kttext_warning|5|TRUE|tablespace pre-extension warning threshold in percentage|
|_ktu_latches|0|TRUE|number of KTU latches|
|_ku_trace|none|TRUE|datapump trace parameter|
|_large_pool_min_alloc|65536|TRUE|minimum allocation size in bytes for the large allocation pool|
|_last_allocation_period|5|TRUE|period over which an instance can retain an active level1 bitmap|
|_latch_class_0|%NULL%|TRUE|latch class 0|
|_latch_class_1|%NULL%|TRUE|latch class 1|
|_latch_class_2|%NULL%|TRUE|latch class 2|
|_latch_class_3|%NULL%|TRUE|latch class 3|
|_latch_class_4|%NULL%|TRUE|latch class 4|
|_latch_class_5|%NULL%|TRUE|latch class 5|
|_latch_class_6|%NULL%|TRUE|latch class 6|
|_latch_class_7|%NULL%|TRUE|latch class 7|
|_latch_classes|%NULL%|TRUE|latch classes override|
|_latch_miss_stat_sid|0|TRUE|Sid of process for which to collect latch stats|
|_latch_recovery_alignment|65534|TRUE|align latch recovery structures|
|_ldr_io_size|262144|TRUE|size of write IOs used during a load operation|
|_ldr_pga_lim|0|TRUE|pga limit, beyond which new partition loads are delayed|
|_left_nested_loops_random|TRUE|FALSE|enable random distribution method for left of nestedloops|
|_lgwr_async_broadcasts|TRUE|TRUE|LGWR Asynchronous Broadcasts enabling boolean flag|
|_lgwr_delay_write|FALSE|TRUE|LGWR write delay for debugging|
|_lgwr_io_slaves|0|TRUE|LGWR I/O slaves|
|_lgwr_max_ns_wt|1|TRUE|Maximum wait time for lgwr to allow NetServer to progress|
|_lgwr_ns_nl_max|1000|TRUE|Variable to simulate network latency or buffer threshold|
|_lgwr_ns_nl_min|500|TRUE|Variable to simulate network latency or buffer threshold|
|_lgwr_ns_sim_err|0|TRUE|Variable to simulate errors lgwrns|
|_lgwr_ta_sim_err|0|TRUE|Variable to simulate errors lgwr true async|
|_library_cache_advice|TRUE|TRUE|whether KGL advice should be turned on|
|_lightweight_hdrs|TRUE|TRUE|Lightweight headers for redo|
|_like_with_bind_as_equality|FALSE|TRUE|treat LIKE predicate with bind as an equality predicate|
|_limit_itls|20|TRUE|limit the number of ITLs in OLTP Compressed Tables|
|_lm_activate_lms_threshold|100|TRUE|threshold value to activate an additional lms|
|_lm_asm_enq_hashing|TRUE|TRUE|if TRUE makes ASM use enqueue master hashing for fusion locks|
|_lm_batch_compression_threshold|0|TRUE|GES threshold to start compression on batch messages|
|_lm_better_ddvictim|TRUE|TRUE|GES better deadlock victim|
|_lm_cache_lvl0_cleanup|0|TRUE|how often to cleanup level 0 cache res (in sec)|
|_lm_cache_res_cleanup|25|TRUE|percentage of cached resources should be cleanup|
|_lm_cache_res_type|TMHWDI|TRUE|cache resource: string of lock types(s)|
|_lm_checksum_batch_msg|0|TRUE|GES checksum batch messages|
|_lm_compression_scheme|zlib|TRUE|GES compression scheme|
|_lm_contiguous_res_count|128|TRUE|number of contiguous blocks that will hash to the same HV bucket|
|_lm_dd_ignore_nodd|FALSE|TRUE|if TRUE nodeadlockwait/nodeadlockblock options are ignored|
|_lm_dd_interval|10|TRUE|dd time interval in seconds|
|_lm_dd_max_search_time|180|TRUE|max dd search time per token|
|_lm_dd_maxdump|50|TRUE|max number of locks to be dumped during dd validation |
|_lm_dd_scan_interval|5|TRUE|dd scan interval in seconds|
|_lm_dd_search_cnt|3|TRUE|number of dd search per token get|
|_lm_deferred_msg_timeout|163|TRUE|deferred message timeout|
|_lm_drm_disable|0|TRUE|disable drm in different level|
|_lm_drm_max_requests|100|TRUE|dynamic remastering maximum affinity requests processed together|
|_lm_drm_object_scan|TRUE|TRUE|enable/disable object scan to force full table scan always|
|_lm_drm_window|0|TRUE|dynamic remastering bucket window size|
|_lm_drm_xlatch|0|TRUE|dynamic remastering forced exclusive latches|
|_lm_dynamic_lms|FALSE|TRUE|dynamic lms invocation|
|_lm_dynamic_load|TRUE|TRUE|dynamic load adjustment|
|_lm_enq_lock_freelist|%NULL%|TRUE|Number of ges enqueue element freelist|
|_lm_enq_rcfg|TRUE|TRUE|if TRUE enables enqueue reconfiguration|
|_lm_enqueue_blocker_dump_timeout|120|TRUE|enqueue blocker dump timeout|
|_lm_enqueue_blocker_kill_timeout|0|TRUE|enqueue blocker kill timeout|
|_lm_enqueue_freelist|3|TRUE|Number of enqueue freelist|
|_lm_file_affinity|%NULL%|TRUE|mapping between file id and master instance number|
|_lm_global_posts|TRUE|TRUE|if TRUE deliver global posts to remote nodes|
|_lm_idle_connection_check|TRUE|TRUE|GES idle connection check|
|_lm_idle_connection_check_interval|140|TRUE|GES idle connection check interval time|
|_lm_idle_connection_kill|TRUE|TRUE|GES idle connection kill|
|_lm_kill_fg_on_timeout|TRUE|TRUE|GES kill fg on IPC timeout|
|_lm_lmd_waittime|8|TRUE|default wait time for lmd in centiseconds|
|_lm_lmon_nowait_latch|TRUE|TRUE|if TRUE makes lmon get nowait latches with timeout loop|
|_lm_lms|0|TRUE|number of background gcs server processes to start|
|_lm_lms_waittime|8|TRUE|default wait time for lms in centiseconds|
|_lm_locks|12000|TRUE|number of enqueues configured for cluster database|
|_lm_master_weight|1|TRUE|master resource weight for this instance|
|_lm_max_lms|0|TRUE|max. number of background global cache server processes|
|_lm_msg_batch_size|0|TRUE|GES batch message size|
|_lm_msg_cleanup_interval|3000|TRUE|GES message buffer cleanup interval time|
|_lm_no_sync|TRUE|TRUE|skip reconfiguration/drm syncr/synca messaging|
|_lm_node_join_opt|FALSE|TRUE|cluster database node join optimization in reconfig|
|_lm_non_fault_tolerant|FALSE|TRUE|disable cluster database fault-tolerance mode|
|_lm_num_pt_buckets|8192|TRUE|number of buckets in the object affinity hash table|
|_lm_num_pt_latches|128|TRUE|number of latches in the object affinity hash table|
|_lm_postevent_buffer_size|256|TRUE|postevent buffer size|
|_lm_preregister_css_restype|CF|TRUE|enqueue type that requires pre-registration to css|
|_lm_proc_freeze_timeout|70|TRUE|reconfiguration: process freeze timeout|
|_lm_process_batching|TRUE|TRUE|GES implicit process batching for IPC messages|
|_lm_procs|320|TRUE|number of client processes configured for cluster database|
|_lm_psrcfg|TRUE|TRUE|enable pseudo reconfiguration|
|_lm_rcfg_timeout|489|TRUE|dlm reconfiguration timeout|
|_lm_rcvinst|TRUE|TRUE|designated instance to do instance recovery|
|_lm_rcvr_hang_allow_time|70|TRUE|receiver hang allow time in seconds|
|_lm_rcvr_hang_check_frequency|20|TRUE|receiver hang check frequency in seconds|
|_lm_rcvr_hang_check_system_load|TRUE|TRUE|examine system load when check receiver health|
|_lm_rcvr_hang_kill|TRUE|TRUE|to kill receiver hang|
|_lm_rcvr_hang_systemstate_dump_level|0|TRUE|systemstate dump level upon receiver hang|
|_lm_res_hash_bucket|0|TRUE|number of resource hash buckets|
|_lm_res_part|128|TRUE|number of resource partition configured for gcs|
|_lm_ress|6000|TRUE|number of resources configured for cluster database|
|_lm_send_mode|auto|TRUE|GES send mode|
|_lm_send_queue_batching|TRUE|TRUE|GES send queue message batching|
|_lm_send_queue_length|5000|TRUE|GES send queue maximum length|
|_lm_sendproxy_reserve|25|TRUE|GES percentage of send proxy reserve of send tickets|
|_lm_share_lock_opt|FALSE|TRUE|if TRUE enables share lock optimization|
|_lm_spare_threads|0|TRUE|number of spare threads to be created by the GPnP master|
|_lm_spare_undo|0|TRUE|number of spare undo tablespaces to be created by GPnP master|
|_lm_sq_batch_factor|2|TRUE|GES send queue minimum batching factor|
|_lm_sq_batch_type|auto|TRUE|GES send queue batching mechanism|
|_lm_sq_batch_waittick|3|TRUE|GES send queue batching waittime in tick|
|_lm_sync_timeout|163|TRUE|Synchronization timeout for DLM reconfiguration steps|
|_lm_ticket_active_sendback|%NULL%|TRUE|Flow control ticket active sendback threshold|
|_lm_tickets|1000|TRUE|GES messaging tickets|
|_lm_tx_delta|16|TRUE|TX lock localization delta|
|_lm_use_gcr|TRUE|TRUE|use GCR module if TRUE|
|_lm_validate_pbatch|FALSE|TRUE|GES process batch validation|
|_lm_validate_resource_type|FALSE|TRUE|if TRUE enables resource name validation|
|_lm_xids|352|TRUE|number of transaction IDs configured for cluster database|
|_lmn_compression|TRUE|TRUE|suppl logging for compression enabled|
|_load_without_compile|NONE|TRUE|Load PL/SQL or Database objects without compilation|
|_local_communication_costing_enabled|TRUE|FALSE|enable local communication costing when TRUE|
|_local_communication_ratio|50|TRUE|set the ratio between global and local communication (0..100)|
|_local_hang_analysis_interval_secs|3|TRUE|the interval at which local hang analysis is run|
|_lock_sga_areas|0|TRUE|Lock specified areas of the SGA in physical memory|
|_log_archive_avoid_memcpy|TRUE|TRUE|log archive avoid memcpy|
|_log_archive_buffers|10|TRUE|Number of buffers to allocate for archiving|
|_log_archive_callout|%NULL%|TRUE|archival callout|
|_log_archive_network_redo_size|10|TRUE|Log archive network redo buffer size used by ARCH|
|_log_archive_prot_auto_demote|FALSE|TRUE|log archive protection auto demotion|
|_log_archive_security_enabled|TRUE|TRUE|log archive security enabled|
|_log_archive_strong_auth|TRUE|TRUE|log archive security strong auth|
|_log_blocks_during_backup|TRUE|TRUE|log block images when changed during backup|
|_log_buffers_corrupt|FALSE|TRUE|corrupt redo buffers before write|
|_log_buffers_debug|FALSE|TRUE|debug redo buffers (slows things down)|
|_log_checkpoint_recovery_check|0|TRUE|# redo blocks to verify after checkpoint|
|_log_committime_block_cleanout|FALSE|TRUE|Log commit-time block cleanout|
|_log_deletion_policy|mandatory|TRUE|archivelog deletion policy for mandatory/all destination|
|_log_event_queues|0|TRUE|number of the log writer event queues|
|_log_file_sync_timeout|10|TRUE|Log file sync timeout (centiseconds)|
|_log_io_size|0|TRUE|automatically initiate log write if this many redo blocks in buffer|
|_log_max_optimize_threads|128|TRUE|maximum number of threads to which log scan optimization is applied|
|_log_parallelism_dynamic|TRUE|TRUE|Enable dynamic strands|
|_log_parallelism_max|1|TRUE|Maximum number of log buffer strands|
|_log_private_mul|5|TRUE|Private strand multiplier for log space preallocation|
|_log_private_parallelism_mul|10|TRUE|Active sessions multiplier to deduce number of private strands|
|_log_read_buffer_size|8|TRUE|buffer size for reading log files|
|_log_read_buffers|8|TRUE|Number of log read buffers for media recovery|
|_log_simultaneous_copies|2|TRUE|number of simultaneous copies into redo buffer(# of copy latches)|
|_log_space_errors|TRUE|TRUE|should we report space errors to alert log|
|_log_switch_timeout|0|TRUE|Maximum number of seconds redos in the current log could span|
|_logout_storm_rate|0|TRUE|number of processes that can logout in a second|
|_logout_storm_retrycnt|600|TRUE|maximum retry count for logouts|
|_logout_storm_timeout|5|TRUE|timeout in centi-seconds for time to wait between retries|
|_longops_enabled|TRUE|TRUE|longops stats enabled|
|_low_server_threshold|0|TRUE|low server thresholds|
|_ltc_trace|0|TRUE|tracing level for load table conventional|
|_master_direct_sends|31|TRUE|direct sends for messages from master (DFS)|
|_mav_refresh_consistent_read|TRUE|TRUE|refresh materialized views using consistent read snapshot|
|_mav_refresh_double_count_prevented|FALSE|TRUE|materialized view MAV refreshes avoid double counting|
|_mav_refresh_opt|0|TRUE|optimizations during refresh of materialized views|
|_mav_refresh_unionall_tables|3|TRUE|# tables for union all expansion during materialized view refresh|
|_max_aq_persistent_queue_memory|10|TRUE|max aq persistent queue memory|
|_max_cr_rollbacks|0|TRUE|Maximum number of CR	rollbacks per block (LMS)|
|_max_exponential_sleep|0|TRUE|max sleep during exponential backoff|
|_max_io_size|1048576|TRUE|Maximum I/O size in bytes for sequential file accesses|
|_max_large_io|0|TRUE|IORM:max number of large I/O's to issue|
|_max_lns_shutdown_archival_time|30|TRUE|Maximum time spent by LNS to archive last log during shutdown|
|_max_protocol_support|10000|TRUE|Max occurrence protocols supported in a process|
|_max_shrink_obj_stats|0|TRUE|number of segments for which shrink stats will be maintained|
|_max_sleep_holding_latch|4|TRUE|max time to sleep while holding a latch|
|_max_small_io|0|TRUE|IORM:max number of small I/O's to issue|
|_max_spacebg_slaves|10|TRUE|maximum space management background slaves|
|_max_spacebg_tasks|1000|TRUE|maximum space management background tasks|
|_max_temp_overhead|0|TRUE|max tempspc overhead|
|_media_recovery_read_batch|32|TRUE|media recovery block read batch|
|_mem_annotation_pr_lev|0|TRUE|private memory annotation collection level|
|_mem_annotation_scale|1|TRUE|memory annotation pre-allocation scaling|
|_mem_annotation_sh_lev|0|TRUE|shared memory annotation collection level|
|_mem_annotation_store|FALSE|TRUE|memory annotation in-memory store|
|_mem_std_extent_size|4096|TRUE|standard extent size for fixed-size-extent heaps|
|_memory_broker_log_stat_entries|5|TRUE|memory broker num stat entries|
|_memory_broker_marginal_utility_bc|12|TRUE|Marginal Utility threshold pct for bc|
|_memory_broker_marginal_utility_sp|7|TRUE|Marginal Utility threshold pct for sp|
|_memory_broker_shrink_heaps|15|TRUE|memory broker allow policy to shrink shared pool|
|_memory_broker_shrink_java_heaps|900|TRUE|memory broker allow policy to shrink java pool|
|_memory_broker_shrink_streams_pool|900|TRUE|memory broker allow policy to shrink streams pool|
|_memory_broker_shrink_timeout|60000000|TRUE|memory broker policy to timeout shrink shared/java pool|
|_memory_broker_stat_interval|30|TRUE|memory broker statistics gathering interval for auto sga|
|_memory_checkinuse_timeintv|30|TRUE|check inuse time interval|
|_memory_imm_mode_without_autosga|TRUE|TRUE|Allow immediate mode without sga/memory target|
|_memory_initial_sga_split_perc|60|TRUE|Initial default sga target percentage with memory target|
|_memory_management_tracing|0|TRUE|trace memory management activity|
|_memory_mgmt_fail_immreq|FALSE|TRUE|always fail immediate mode request|
|_memory_mgmt_immreq_timeout|150|TRUE|time in seconds to time out immediate mode request|
|_memory_nocancel_defsgareq|FALSE|TRUE|do not cancel deferred sga reqs with auto-memory|
|_memory_sanity_check|0|TRUE|partial granule sanity check|
|_messages|300|TRUE|message queue resources - dependent on # processes & # buffers|
|_mgd_rcv_handle_orphan_datafiles|FALSE|TRUE|Managed recovery handle orphan datafile situation|
|_midtier_affinity_clusterwait_threshold|100|TRUE|cluster wait threshold to enter affinity|
|_midtier_affinity_goodness_threshold|2000|TRUE|goodness gradient threshold to dissolve affinity|
|_minfree_plus|0|TRUE|max percentage of block space + minfree before we mark block full|
|_minimal_stats_aggregation|TRUE|FALSE|prohibit stats aggregation at compile/partition maintenance time|
|_minimum_blocks_to_shrink|0|TRUE|minimum number freeable blocks for shrink to be present|
|_minimum_db_flashback_retention|60|TRUE|Minimum flashback retention|
|_minimum_extents_to_shrink|1|TRUE|minimum number freeable extents for shrink to be present|
|_minimum_giga_scn|0|TRUE|Minimum SCN to start with in 2^30 units|
|_mirror_redo_buffers|FALSE|TRUE|Save buffers for debugging redo corruptions|
|_mmv_query_rewrite_enabled|TRUE|FALSE|allow rewrites with multiple MVs and/or base tables|
|_mpmt_enabled|FALSE|TRUE|MPMT mode enabled|
|_multi_instance_pmr|TRUE|TRUE|force multi instance or single instance parallel recovery|
|_multi_join_key_table_lookup|TRUE|TRUE|TRUE iff multi-join-key table lookup prefetch is enabled|
|_multiple_instance_recovery|FALSE|TRUE|use multiple instances for media recovery|
|_mv_generalized_oj_refresh_opt|TRUE|TRUE|enable/disable new algorithm for MJV with generalized outer joins|
|_mv_refresh_ana|0|TRUE|what percent to analyze after complete/PCT refresh|
|_mv_refresh_costing|rule|TRUE|refresh decision based on cost or on rules|
|_mv_refresh_delta_fraction|10|TRUE|delta mv as fractional percentage of size of mv|
|_mv_refresh_enhanced_dml_detection|ON_RC|TRUE|enable enhanced detection of DML types from MV log|
|_mv_refresh_eut|TRUE|TRUE|refresh materialized views using EUT(partition)-based algorithm|
|_mv_refresh_force_parallel_query|0|TRUE|force materialized view refreshes to use parallel query|
|_mv_refresh_new_setup_disabled|FALSE|TRUE|materialized view MV refresh new setup disabling|
|_mv_refresh_pkfk_data_units_opt|auto|TRUE|control MV refresh based on the assumption of PK-FK data units|
|_mv_refresh_pkfk_relationship_opt|TRUE|TRUE|control MV refresh based on the use of PK-FK relationships|
|_mv_refresh_rebuild_percentage|10|TRUE|minimum percentage change required in MV to force an indexrebuild|
|_mv_refresh_selections|TRUE|TRUE|create materialized views with selections and fast refresh|
|_mv_refresh_use_no_merge|TRUE|TRUE|use no_merge hint in queries|
|_mv_refresh_use_stats|FALSE|TRUE|pass cardinality hints to refresh queries|
|_mv_refsched_timeincr|300000|TRUE|proportionality constant for dop vs. time in MV refresh|
|_mv_rolling_inv|FALSE|TRUE|create/alter mv uses rolling cursor invalidation instead of immediate |
|_mwin_schedule|TRUE|TRUE|Enable/disable Maintenance Window Schedules|
|_nchar_imp_cnv|TRUE|TRUE|NLS allow Implicit Conversion between CHAR and NCHAR|
|_nchar_imp_conv|TRUE|TRUE|should implicit conversion bewteen clob and nclob be allowed|
|_ncmb_readahead_enabled|0|TRUE|enable multi-block readahead for an index scan|
|_ncmb_readahead_tracing|0|TRUE|turn on multi-block readahead tracing|
|_ncomp_shared_objects_dir|%NULL%|TRUE|native compilation shared objects dir|
|_nested_loop_fudge|100|TRUE|nested loop fudge|
|_nested_mav_fast_oncommit_enabled|TRUE|TRUE|nested MAV refresh fast on commit allowed|
|_new_initial_join_orders|TRUE|FALSE|enable initial join orders based on new ordering heuristics|
|_new_sort_cost_estimate|TRUE|FALSE|enables the use of new cost estimate for sort|
|_newsort_enabled|TRUE|TRUE|controls whether new sorts can be used as system sort|
|_newsort_ordered_pct|63|TRUE|controls when new sort avoids sorting ordered input|
|_newsort_type|0|TRUE|specifies options for the new sort algorithm|
|_nlj_batching_enabled|1|FALSE|enable batching of the RHS IO in NLJ|
|_nlj_batching_misses_enabled|1|TRUE|enable exceptions for buffer cache misses|
|_no_objects|FALSE|TRUE|no object features are used|
|_no_or_expansion|FALSE|TRUE|OR expansion during optimization disabled|
|_no_recovery_through_resetlogs|FALSE|TRUE|no recovery through this resetlogs operation|
|_noseg_for_unusable_index_enabled|TRUE|TRUE|no segments for unusable indexes if set to TRUE|
|_notify_crs|FALSE|TRUE|notify cluster ready services of startup and shutdown|
|_ns_max_flush_wt|1|TRUE|Flush wait time for NetServer to flush oustanding writes|
|_ns_max_send_delay|15|TRUE|Data Loss Time Bound for NetServer|
|_num_longop_child_latches|1|TRUE|number of child latches for long op array|
|_numa_trace_level|0|TRUE|numa trace event|
|_number_cached_attributes|10|TRUE|maximum number of cached attributes per instance|
|_obj_ckpt_tracing|0|TRUE|Enable object checkpoint tracing|
|_object_number_cache_size|5|TRUE|Object number cache size|
|_object_reuse_bast|2|TRUE|if 1 or higher, handle object reuse|
|_object_statistics|TRUE|TRUE|enable the object level statistics collection|
|_object_stats_max_entries|3072|TRUE|Maximum number of entries to be tracked per stat|
|_offline_rollback_segments|%NULL%|TRUE|offline undo segment list|
|_ogms_home|%NULL%|TRUE|GMS home directory|
|_olap_adv_comp_stats_cc_precomp|20|TRUE|do additional predicate stats analysis for AW rowsource|
|_olap_adv_comp_stats_max_rows|100000|TRUE|do additional predicate stats analysis for AW rowsource|
|_olap_aggregate_buffer_size|1048576|TRUE|OLAP Aggregate max buffer size|
|_olap_aggregate_flags|0|TRUE|OLAP Aggregate debug flags|
|_olap_aggregate_function_cache_enabled|TRUE|TRUE|OLAP Aggregate function cache enabler|
|_olap_aggregate_function_merge_threshold|529|TRUE|OLAP Aggregate function merge threshold|
|_olap_aggregate_max_thread_tuples|5000|TRUE|OLAP Aggregate max thread tuples creation|
|_olap_aggregate_min_buffer_size|1024|TRUE|OLAP Aggregate min buffer size|
|_olap_aggregate_min_thread_status|64|TRUE|OLAP Aggregate minimum cardinality of dimensions for thread|
|_olap_aggregate_multipath_hier|FALSE|TRUE|OLAP Aggregate Multi-path Hierarhies enabled|
|_olap_aggregate_statlen_thresh|1024|TRUE|OLAP Aggregate status array usage threshold|
|_olap_aggregate_store_probability|100|TRUE|OLAP Aggregate function storeback probability|
|_olap_aggregate_work_per_thread|1024|TRUE|OLAP Aggregate max work parents|
|_olap_aggregate_worklist_max|5000|TRUE|OLAP Aggregate max worklists generated at once|
|_olap_allocate_errorlog_format|%8p %8y %8z %e (%n)|TRUE|OLAP Allocate Errorlog Format|
|_olap_allocate_errorlog_header|Dim	 Source   Basis%BR%%-8d %-8s %-8b Description%BR%-------- -------- -------- -----------|TRUE|OLAP Allocate Errorlog Header format|
|_olap_analyze_max|10000|TRUE|OLAP DML ANALYZE command max cells to analyze|
|_olap_continuous_trace_file|false|TRUE|OLAP logging definition|
|_olap_dbgoutfile_echo_to_eventlog|FALSE|TRUE|OLAP DbgOutfile copy output to event log (tracefile)|
|_olap_dimension_corehash_large|50000|TRUE|OLAP Dimension In-Core Hash Table Large Threshold|
|_olap_dimension_corehash_pressure|90|TRUE|OLAP Dimension In-Core Hash Table Pressure Threshold|
|_olap_dimension_corehash_size|30|TRUE|OLAP Dimension In-Core Hash Table Maximum Memory Use|
|_olap_dimension_corehash_small|1000|TRUE|OLAP Dimension In-Core Hash Table Small Threshold|
|_olap_eif_export_lob_size|2147483647|TRUE|OLAP EIF Export BLOB size|
|_olap_lmgen_dim_size|100|TRUE|Limitmap generator dimension column size|
|_olap_lmgen_meas_size|1000|TRUE|Limitmap generator measure column size|
|_olap_object_hash_class|3|TRUE|OLAP Object Hash Table Class|
|_olap_page_pool_expand_rate|20|TRUE|OLAP Page Pool Expand Rate|
|_olap_page_pool_hi|50|TRUE|OLAP Page Pool High Watermark|
|_olap_page_pool_hit_target|100|TRUE|OLAP Page Pool Hit Target|
|_olap_page_pool_low|262144|TRUE|OLAP Page Pool Low Watermark|
|_olap_page_pool_pressure|90|TRUE|OLAP Page Pool Pressure Threshold|
|_olap_page_pool_shrink_rate|50|TRUE|OLAP Page Pool Shrink Rate|
|_olap_parallel_update_small_group|400|TRUE|OLAP parallel update pagespace by group|
|_olap_parallel_update_small_threshold|1000|TRUE|OLAP parallel update threshold for number of small pagespaces|
|_olap_parallel_update_threshold|1000|TRUE|OLAP parallel update threshold in pages|
|_olap_sesscache_enabled|TRUE|TRUE|OLAP Session Cache knob|
|_olap_sort_buffer_pct|10|TRUE|OLAP Sort Buffer Size Percentage|
|_olap_sort_buffer_size|262144|TRUE|OLAP Sort Buffer Size|
|_olap_statbool_corebits|20000000|TRUE|OLAP Status Boolean max incore bits|
|_olap_statbool_threshold|8100|TRUE|OLAP Status Boolean CBM threshold|
|_olap_table_function_statistics|FALSE|TRUE|Specify TRUE to output OLAP table function timed statistics trace|
|_olap_wrap_errors|FALSE|TRUE|Wrap error messages to OLAP outfile|
|_olapi_history_retention|FALSE|TRUE|enable olapi history retention|
|_olapi_iface_object_history|1000|TRUE|enable olapi interface object history collection|
|_olapi_iface_object_history_retention|FALSE|TRUE|enable olapi interface object history retention|
|_olapi_iface_operation_history_retention|FALSE|TRUE|enable olapi interface operation history retention|
|_olapi_interface_operation_history|1000|TRUE|enable olapi interface operation history collection|
|_olapi_memory_operation_history|1000|TRUE|enable olapi memory alloc/free history collection|
|_olapi_memory_operation_history_pause_at_seqno|0|TRUE|enable olapi memory alloc/free history collection pausing|
|_olapi_memory_operation_history_retention|FALSE|TRUE|enable olapi memory operation history retention|
|_olapi_session_history|300|TRUE|enable olapi session history collection|
|_olapi_session_history_retention|FALSE|TRUE|enable olapi session history retention|
|_old_connect_by_enabled|FALSE|TRUE|enable/disable old connect by|
|_ols_cleanup_task|TRUE|TRUE|Clean up unnecessary entries in OLS sessinfo table|
|_oltp_compression|TRUE|TRUE|oltp compression enabled|
|_oltp_compression_gain|10|TRUE|oltp compression gain|
|_omf|enabled|TRUE|enable/disable OMF|
|_oneside_colstat_for_equijoins|TRUE|TRUE|sanity check on default selectivity for like/range predicate|
|_online_patch_disable_stack_check|FALSE|TRUE|disable check for function on stack for online patches|
|_optim_adjust_for_part_skews|TRUE|FALSE|adjust stats for skews across partitions|
|_optim_dict_stats_at_db_cr_upg|TRUE|TRUE|enable/disable dictionary stats gathering at db create/upgrade|
|_optim_enhance_nnull_detection|TRUE|FALSE|TRUE to enable index [fast] full scan more often|
|_optim_new_default_join_sel|TRUE|FALSE|improves the way default equijoin selectivity are computed|
|_optim_peek_user_binds|TRUE|FALSE|enable peeking of user binds|
|_optimizer_adaptive_cursor_sharing|TRUE|FALSE|optimizer adaptive cursor sharing|
|_optimizer_adjust_for_nulls|TRUE|TRUE|adjust selectivity for null values|
|_optimizer_autostats_job|TRUE|TRUE|enable/disable auto stats collection job|
|_optimizer_aw_join_push_enabled|TRUE|TRUE|Enables AW Join Push optimization|
|_optimizer_aw_stats_enabled|TRUE|TRUE|Enables statistcs on AW olap_table table function|
|_optimizer_better_inlist_costing|ALL|FALSE|enable improved costing of index access using in-list(s)|
|_optimizer_block_size|8192|TRUE|standard block size used by optimizer|
|_optimizer_cache_stats|FALSE|TRUE|cost with cache statistics|
|_optimizer_cartesian_enabled|TRUE|TRUE|optimizer cartesian join enabled|
|_optimizer_cbqt_factor|50|TRUE|cost factor for cost-based query transformation|
|_optimizer_cbqt_no_size_restriction|TRUE|FALSE|disable cost based transformation query size restriction|
|_optimizer_ceil_cost|TRUE|TRUE|CEIL cost in CBO|
|_optimizer_coalesce_subqueries|TRUE|FALSE|consider coalescing of subqueries optimization|
|_optimizer_complex_pred_selectivity|TRUE|FALSE|enable selectivity estimation for builtin functions|
|_optimizer_compute_index_stats|TRUE|FALSE|force index stats collection on index creation/rebuild|
|_optimizer_connect_by_cb_whr_only|FALSE|TRUE|use cost-based transformation for whr clause in connect by|
|_optimizer_connect_by_combine_sw|TRUE|FALSE|combine no filtering connect by and start with|
|_optimizer_connect_by_cost_based|TRUE|FALSE|use cost-based transformation for connect by|
|_optimizer_connect_by_elim_dups|TRUE|FALSE|allow connect by to eliminate duplicates from input|
|_optimizer_correct_sq_selectivity|TRUE|FALSE|force correct computation of subquery selectivity|
|_optimizer_cost_based_transformation|LINEAR|FALSE|enables cost-based query transformation|
|_optimizer_cost_filter_pred|FALSE|TRUE|enables  costing of filter predicates in IO cost model|
|_optimizer_cost_hjsmj_multimatch|TRUE|FALSE|add cost of generating result set when #rows per key > 1|
|_optimizer_cost_model|CHOOSE|FALSE|optimizer cost model|
|_optimizer_degree|0|TRUE|force the optimizer to use the same degree of parallelism|
|_optimizer_dim_subq_join_sel|TRUE|FALSE|use join selectivity in choosing star transformation dimensions|
|_optimizer_disable_strans_sanity_checks|0|TRUE|disable star transformation sanity checks|
|_optimizer_distinct_agg_transform|TRUE|FALSE|Transforms Distinct Aggregates to non-distinct aggregates|
|_optimizer_distinct_elimination|TRUE|FALSE|Eliminates redundant SELECT DISTNCT's|
|_optimizer_distinct_placement|TRUE|FALSE|consider distinct placement optimization|
|_optimizer_dyn_smp_blks|32|TRUE|number of blocks for optimizer dynamic sampling|
|_optimizer_eliminate_filtering_join|TRUE|FALSE|optimizer filtering join elimination enabled|
|_optimizer_enable_density_improvements|TRUE|FALSE|use improved density computation for selectivity estimation|
|_optimizer_enable_extended_stats|TRUE|FALSE|use extended statistics for selectivity estimation|
|_optimizer_enhanced_filter_push|TRUE|FALSE|push filters before trying cost-based query transformation|
|_optimizer_extend_jppd_view_types|TRUE|FALSE|join pred pushdown on group-by, distinct, semi-/anti-joined view|
|_optimizer_extended_cursor_sharing|UDO|FALSE|optimizer extended cursor sharing|
|_optimizer_extended_cursor_sharing_rel|SIMPLE|FALSE|optimizer extended cursor sharing for relational operators|
|_optimizer_extended_stats_usage_control|224|FALSE|controls the optimizer usage of extended stats|
|_optimizer_fast_access_pred_analysis|TRUE|FALSE|use fast algorithm to traverse predicates for physical optimizer|
|_optimizer_fast_pred_transitivity|TRUE|FALSE|use fast algorithm to generate transitive predicates|
|_optimizer_feedback_control|%NULL%|TRUE|controls the optimizer feedback framework|
|_optimizer_filter_pred_pullup|TRUE|FALSE|use cost-based flter predicate pull up transformation|
|_optimizer_fkr_index_cost_bias|10|FALSE|Optimizer index bias over FTS/IFFS under first K rows mode|
|_optimizer_force_CBQT|%NULL%|TRUE|force CBQT transformation regardless of cost|
|_optimizer_free_transformation_heap|TRUE|TRUE|free transformation subheap after each transformation|
|_optimizer_group_by_placement|TRUE|FALSE|consider group-by placement optimization|
|_optimizer_ignore_hints|FALSE|TRUE|enables the embedded hints to be ignored|
|_optimizer_improve_selectivity|TRUE|FALSE|improve table and partial overlap join selectivity computation|
|_optimizer_instance_count|0|TRUE|force the optimizer to use the specified number of instances|
|_optimizer_invalidation_period|18000|TRUE|time window for invalidation of cursors of analyzed objects|
|_optimizer_join_elimination_enabled|TRUE|FALSE|optimizer join elimination enabled|
|_optimizer_join_factorization|TRUE|FALSE|use join factorization transformation|
|_optimizer_join_order_control|3|FALSE|controls the optimizer join order search algorithm|
|_optimizer_join_sel_sanity_check|TRUE|FALSE|enable/disable sanity check for multi-column join selectivity|
|_optimizer_max_permutations|2000|FALSE|optimizer maximum join permutations per query block|
|_optimizer_min_cache_blocks|10|TRUE|set minimum cached blocks|
|_optimizer_mjc_enabled|TRUE|TRUE|enable merge join cartesian|
|_optimizer_mode_force|TRUE|FALSE|force setting of optimizer mode for user recursive SQL also|
|_optimizer_multi_level_push_pred|TRUE|FALSE|consider join-predicate pushdown that requires multi-level pushdown to base table|
|_optimizer_multiple_cenv|%NULL%|TRUE|generate and run plans using several compilation environments|
|_optimizer_multiple_cenv_report|result|TRUE|control what to report in trace file when run in multi-plan mode|
|_optimizer_multiple_cenv_stmt|query|TRUE|control the types of statements that are run in multi-plan mode|
|_optimizer_native_full_outer_join|FORCE|FALSE|execute full outer join using native implementaion|
|_optimizer_nested_rollup_for_gset|100|TRUE|number of groups above which we use nested rollup exec for gset|
|_optimizer_new_join_card_computation|TRUE|FALSE|compute join cardinality using non-rounded input values|
|_optimizer_null_aware_antijoin|TRUE|FALSE|null-aware antijoin parameter|
|_optimizer_or_expansion|DEPTH|FALSE|control or expansion approach used|
|_optimizer_or_expansion_subheap|TRUE|TRUE|Use subheap for optimizer or-expansion|
|_optimizer_order_by_elimination_enabled|TRUE|FALSE|Eliminates order bys from views before query transformation|
|_optimizer_outer_to_anti_enabled|TRUE|FALSE|Enable transformation of outer-join to anti-join if possible|
|_optimizer_percent_parallel|101|TRUE|optimizer percent parallel|
|_optimizer_push_down_distinct|0|FALSE|push down distinct from query block to table|
|_optimizer_push_pred_cost_based|TRUE|FALSE|use cost-based query transformation for push pred optimization|
|_optimizer_random_plan|0|TRUE|optimizer seed value for random plans|
|_optimizer_reuse_cost_annotations|TRUE|TRUE|reuse cost annotations during cost-based query transformation|
|_optimizer_rownum_bind_default|10|FALSE|Default value to use for rownum bind|
|_optimizer_rownum_pred_based_fkr|TRUE|FALSE|enable the use of first K rows due to rownum predicate|
|_optimizer_save_stats|TRUE|TRUE|enable/disable saving old versions of optimizer stats|
|_optimizer_search_limit|5|TRUE|optimizer search limit|
|_optimizer_self_induced_cache_cost|FALSE|TRUE|account for self-induced caching|
|_optimizer_skip_scan_enabled|TRUE|FALSE|enable/disable index skip scan|
|_optimizer_skip_scan_guess|FALSE|TRUE|consider index skip scan for predicates with guessed selectivity|
|_optimizer_sortmerge_join_enabled|TRUE|TRUE|enable/disable sort-merge join method|
|_optimizer_sortmerge_join_inequality|TRUE|FALSE|enable/disable sort-merge join using inequality predicates|
|_optimizer_squ_bottomup|TRUE|FALSE|enables unnesting of subquery in a bottom-up manner|
|_optimizer_star_tran_in_with_clause|TRUE|FALSE|enable/disable star transformation in with clause queries|
|_optimizer_star_trans_min_cost|0|TRUE|optimizer star transformation minimum cost|
|_optimizer_star_trans_min_ratio|0|TRUE|optimizer star transformation minimum ratio|
|_optimizer_starplan_enabled|TRUE|TRUE|optimizer star plan enabled|
|_optimizer_system_stats_usage|TRUE|FALSE|system statistics usage|
|_optimizer_table_expansion|TRUE|FALSE|consider table expansion transformation|
|_optimizer_trace|none|TRUE|optimizer trace parameter|
|_optimizer_transitivity_retain|TRUE|FALSE|retain equi-join pred upon transitive equality pred generation|
|_optimizer_try_st_before_jppd|TRUE|FALSE|try Star Transformation before Join Predicate Push Down|
|_optimizer_undo_changes|FALSE|TRUE|undo changes to query optimizer|
|_optimizer_undo_cost_change|11.2.0.1|FALSE|optimizer undo cost change|
|_optimizer_unnest_all_subqueries|TRUE|TRUE|enables unnesting of every type of subquery|
|_optimizer_unnest_corr_set_subq|TRUE|FALSE|Unnesting of correlated set subqueries (TRUE/FALSE)|
|_optimizer_unnest_disjunctive_subq|TRUE|FALSE|Unnesting of disjunctive subqueries (TRUE/FALSE)|
|_optimizer_use_cbqt_star_transformation|TRUE|FALSE|use rewritten star transformation using cbqt framework|
|_optimizer_use_feedback|TRUE|FALSE|optimizer use feedback|
|_optimizer_use_subheap|TRUE|TRUE|Enables physical optimizer subheap|
|_or_expand_nvl_predicate|TRUE|FALSE|enable OR expanded plan for NVL/DECODE predicate|
|_oradbg_pathname|%NULL%|TRUE|path of oradbg script|
|_oradebug_cmds_at_startup|%NULL%|TRUE|oradebug commands to execute at instance startup|
|_oradebug_force|FALSE|TRUE|force target processes to execute oradebug commands?|
|_ordered_nested_loop|TRUE|FALSE|enable ordered nested loop costing|
|_ordered_semijoin|TRUE|TRUE|enable ordered semi-join subquery|
|_orph_cln_interval|1200|TRUE|qmon periodic interval for removed subscriber messages cleanup|
|_os_sched_high_priority|1|TRUE|OS high priority level|
|_oss_skgxp_udp_dynamic_credit_mgmt|1|TRUE|OSSLIB enable[!0]/disable[0] dynamic credit mgmt for SKGXP-UDP|
|_other_wait_event_exclusion|%NULL%|TRUE|exclude event names from _other_wait_threshold calculations|
|_other_wait_threshold|0|TRUE|threshold wait percentage for event wait class Other|
|_outline_bitmap_tree|TRUE|TRUE|BITMAP_TREE hint enabled in outline|
|_outstanding_broadcast_scn_waits|8|TRUE|maximum number of outstanding broadcast SCN waits|
|_parallel_adaptive_max_users|1|TRUE|maximum number of users running with default DOP|
|_parallel_broadcast_enabled|TRUE|FALSE|enable broadcasting of small inputs to hash and sort merge joins|
|_parallel_cluster_cache_pct|80|TRUE|max percentage of the global buffer cache to use for affinity|
|_parallel_cluster_cache_policy|ADAPTIVE|TRUE|policy used for parallel execution on cluster(ADAPTIVE/CACHED)|
|_parallel_conservative_queuing|FALSE|TRUE|conservative parallel statement queuing|
|_parallel_default_max_instances|1|TRUE|default maximum number of instances for parallel query|
|_parallel_execution_message_align|FALSE|TRUE|Alignment of PX buffers to OS page boundary|
|_parallel_fake_class_pct|0|TRUE|fake db-scheduler percent used for testing|
|_parallel_fixwrite_bucket|1000|TRUE|Number of buckets for each round of fix write|
|_parallel_load_bal_unit|0|TRUE|number of threads to allocate per instance|
|_parallel_load_balancing|TRUE|TRUE|parallel execution load balanced slave allocation|
|_parallel_min_message_pool|491520|TRUE|minimum size of shared pool memory to reserve for pq servers|
|_parallel_optimization_phase_for_local|FALSE|TRUE|parallel optimization phase when all slaves are local|
|_parallel_queuing_max_waitingtime|%NULL%|TRUE|parallel statement queuing: max waiting time in queue|
|_parallel_recovery_stopat|32767|TRUE|stop at -position- to step through SMON|
|_parallel_replay_msg_limit|4000|TRUE|Number of messages for each round of parallel replay|
|_parallel_scalability|50|TRUE|Parallel scalability criterion for parallel execution|
|_parallel_server_idle_time|30000|TRUE|idle time before parallel query server dies (in 1/100 sec)|
|_parallel_server_sleep_time|10|TRUE|sleep time between dequeue timeouts (in 1/100ths)|
|_parallel_slave_acquisition_wait|1|TRUE|time(in seconds) to wait before retrying slave acquisition|
|_parallel_statement_queuing|FALSE|TRUE|parallel statement queuing enabled|
|_parallel_syspls_obey_force|TRUE|TRUE|TRUE to obey force parallel query/dml/ddl under System PL/SQL|
|_parallel_time_unit|10|TRUE|unit of work used to derive the degree of parallelism (in seconds)|
|_parallel_txn_global|FALSE|TRUE|enable parallel_txn hint with updates and deletes|
|_parallelism_cost_fudge_factor|350|TRUE|set the parallelism cost fudge factor|
|_parameter_table_block_size|2048|TRUE|parameter table block size|
|_partial_pwise_join_enabled|TRUE|TRUE|enable partial partition-wise join when TRUE|
|_partition_view_enabled|TRUE|FALSE|enable/disable partitioned views|
|_passwordfile_enqueue_timeout|900|TRUE|password file enqueue timeout in seconds|
|_pct_refresh_double_count_prevented|TRUE|TRUE|materialized view PCT refreshes avoid double counting|
|_pdml_gim_sampling|5000|TRUE|control separation of global index maintenance for PDML|
|_pdml_gim_staggered|FALSE|TRUE|slaves start on different index when doing index maint|
|_pdml_slaves_diff_part|TRUE|TRUE|slaves start on different partition when doing index maint|
|_percent_flashback_buf_partial_full|50|TRUE|Percent of flashback buffer filled to be considered partial full|
|_pga_large_extent_size|1048576|TRUE|PGA large extent size|
|_pga_max_size|209715200|TRUE|Maximum size of the PGA memory for one process|
|_pgactx_cap_stacks|FALSE|TRUE|capture stacks for setting pgactx|
|_pivot_implementation_method|CHOOSE|FALSE|pivot implementation method|
|_pkt_enable|FALSE|TRUE|enable progressive kill test|
|_pkt_pmon_interval|50|TRUE|PMON process clean-up interval (cs)|
|_pkt_start|FALSE|TRUE|start progressive kill test instrumention|
|_plan_outline_data|TRUE|TRUE|explain plan outline data enabled|
|_plan_verify_improvement_margin|150|TRUE|Performance improvement criterion for evolving plan baselines|
|_plan_verify_local_time_limit|0|TRUE|Local time limit to use for an individual plan verification|
|_plsql_anon_block_code_type|INTERPRETED|TRUE|PL/SQL anonymous block code-type|
|_plsql_cache_enable|TRUE|TRUE|PL/SQL Function Cache Enabled|
|_plsql_dump_buffer_events|%NULL%|TRUE|conditions upon which the PL/SQL circular buffer is dumped|
|_plsql_max_stack_size|0|TRUE|PL/SQL maximum stack size|
|_plsql_minimum_cache_hit_percent|20|TRUE|plsql minimum cache hit percentage required to keep caching active|
|_plsql_native_frame_threshold|4294967294|TRUE|Allocate PL/SQL native frames on the heap if size exceeds this value|
|_plsql_nvl_optimize|FALSE|TRUE|PL/SQL NVL optimize|
|_pmon_dead_blkrs_alive_chk_rate_secs|10|TRUE|rate to check blockers are alive during cleanup (in seconds)|
|_pmon_dead_blkrs_max_cleanup_attempts|5|TRUE|max attempts per blocker while checking dead blockers|
|_pmon_dead_blkrs_scan_rate_secs|3|TRUE|rate to scan for dead blockers during cleanup (in seconds)|
|_pmon_enable_dead_blkrs|FALSE|TRUE|look for dead blockers during PMON cleanup|
|_pmon_load_constants|300,192,64,3,10,10,0,0|TRUE|server load balancing constants (S,P,D,I,L,C,M)|
|_pqq_debug_txn_act|FALSE|TRUE|pq queuing transaction active|
|_pre_rewrite_push_pred|TRUE|FALSE|push predicates into views before rewrite|
|_precompute_gid_values|TRUE|TRUE|precompute gid values and copy them before returning a row|
|_pred_move_around|TRUE|FALSE|enables predicate move-around|
|_predicate_elimination_enabled|TRUE|TRUE|allow predicate elimination if set to TRUE|
|_prescomm|FALSE|TRUE|presume commit of IMU transactions|
|_print_refresh_schedule|false|TRUE|enable dbms_output of materialized view refresh schedule|
|_private_memory_address|%NULL%|TRUE|Start address of large extent memory segment|
|_project_view_columns|TRUE|TRUE|enable projecting out unreferenced columns of a view|
|_projection_pushdown|TRUE|TRUE|projection pushdown|
|_projection_pushdown_debug|0|TRUE|level for projection pushdown debugging|
|_prop_old_enabled|FALSE|TRUE|Shift to pre 11g propagation behaviour|
|_protect_frame_heaps|FALSE|TRUE|Protect cursor frame heaps|
|_push_join_predicate|TRUE|FALSE|enable pushing join predicate inside a view|
|_push_join_union_view|TRUE|FALSE|enable pushing join predicate inside a union all view|
|_push_join_union_view2|TRUE|FALSE|enable pushing join predicate inside a union view|
|_px_async_getgranule|FALSE|TRUE|asynchronous get granule in the slave|
|_px_bind_peek_sharing|TRUE|TRUE|enables sharing of px cursors that were built using bind peeking|
|_px_broadcast_fudge_factor|100|TRUE|set the tq broadcasting fudge factor percentage|
|_px_buffer_ttl|30|TRUE|ttl for px mesg buffers in seconds|
|_px_compilation_debug|0|TRUE|debug level for parallel compilation|
|_px_compilation_trace|0|TRUE|tracing level for parallel compilation|
|_px_dump_12805_source|TRUE|TRUE|enables or disables tracing of 12805 signal source|
|_px_dynamic_opt|TRUE|TRUE|turn off/on restartable qerpx dynamic optimization|
|_px_dynamic_sample_size|50|TRUE|num of samples for restartable qerpx dynamic optimization|
|_px_execution_services_enabled|TRUE|TRUE|enable service-based constraint of px slave allocation|
|_px_freelist_latch_divisor|2|TRUE|Divide the computed number of freelists by this power of 2|
|_px_gim_factor|100|TRUE|weighted autodop global index maintenance factor|
|_px_granule_batch_size|10|TRUE|default size of a batch of granules|
|_px_granule_size|1000000|TRUE|default size of a rowid range granule (in KB)|
|_px_index_sampling|200|TRUE|parallel query sampling for index create (100000 = 100%)|
|_px_index_sampling_objsize|TRUE|TRUE|parallel query sampling for index create based on object size|
|_px_io_process_bandwidth|200|TRUE|IO process bandwidth in MB/sec for computing DOP|
|_px_io_system_bandwidth|0|TRUE|total IO system bandwidth in MB/sec for computing DOP|
|_px_kxib_tracing|0|TRUE|turn on kxib tracing|
|_px_load_factor|300|TRUE|weighted autodop load factor|
|_px_load_publish_interval|200|TRUE|interval at which LMON will check whether to publish PX load|
|_px_loc_msg_cost|1000|TRUE|CPU cost to send a PX message via shared memory|
|_px_max_granules_per_slave|100|TRUE|maximum number of rowid range granules to generate per slave|
|_px_max_map_val|32|TRUE|Maximum value of rehash mapping for PX|
|_px_min_granules_per_slave|13|TRUE|minimum number of rowid range granules to generate per slave|
|_px_minus_intersect|TRUE|FALSE|enables pq for minus/interect operators|
|_px_net_msg_cost|10000|TRUE|CPU cost to send a PX message over the internconnect|
|_px_no_granule_sort|FALSE|TRUE|prevent parallel partition granules to be sorted on size|
|_px_no_stealing|FALSE|TRUE|prevent parallel granule stealing in shared nothing environment|
|_px_nss_planb|TRUE|TRUE|enables or disables NSS Plan B reparse with outline|
|_px_proc_constrain|TRUE|TRUE|reduce parallel_max_servers if greater than (processes - fudge)|
|_px_pwg_enabled|TRUE|FALSE|parallel partition wise group by enabled|
|_px_rownum_pd|TRUE|TRUE|turn off/on parallel rownum pushdown optimization|
|_px_send_timeout|300|TRUE|IPC message	send timeout value in seconds|
|_px_slaves_share_cursors|0|TRUE|slaves share cursors with QC|
|_px_trace|none|TRUE|px trace parameter|
|_px_ual_serial_input|TRUE|FALSE|enables new pq for UNION operators|
|_px_xtgranule_size|10000|TRUE|default size of a external table granule (in KB)|
|_qa_control|0|TRUE|Oracle internal parameter to control QA|
|_qa_lrg_type|0|TRUE|Oracle internal parameter to specify QA lrg type|
|_query_cost_rewrite|TRUE|TRUE|perform the cost based rewrite with materialized views|
|_query_execution_cache_max_size|65536|TRUE|max size of query execution cache|
|_query_mmvrewrite_maxcmaps|20|TRUE|query mmv rewrite maximum number of cmaps per dmap in query disjunct|
|_query_mmvrewrite_maxdmaps|10|TRUE|query mmv rewrite maximum number of dmaps per query disjunct|
|_query_mmvrewrite_maxinlists|5|TRUE|query mmv rewrite maximum number of in-lists per disjunct|
|_query_mmvrewrite_maxintervals|5|TRUE|query mmv rewrite maximum number of intervals per disjunct|
|_query_mmvrewrite_maxmergedcmaps|50|TRUE|query mmv rewrite maximum number of merged cmaps|
|_query_mmvrewrite_maxpreds|10|TRUE|query mmv rewrite maximum number of predicates per disjunct|
|_query_mmvrewrite_maxqryinlistvals|500|TRUE|query mmv rewrite maximum number of query in-list values|
|_query_mmvrewrite_maxregperm|512|TRUE|query mmv rewrite maximum number of region permutations|
|_query_on_physical|TRUE|TRUE|query on physical|
|_query_rewrite_1|TRUE|TRUE|perform query rewrite before&after or only before view merging|
|_query_rewrite_2|TRUE|TRUE|perform query rewrite before&after or only after view merging|
|_query_rewrite_drj|TRUE|TRUE|mv rewrite and drop redundant joins|
|_query_rewrite_expression|TRUE|TRUE|rewrite with cannonical form for expressions|
|_query_rewrite_fpc|TRUE|TRUE|mv rewrite fresh partition containment|
|_query_rewrite_fudge|90|TRUE|cost based query rewrite with MVs fudge factor|
|_query_rewrite_jgmigrate|TRUE|TRUE|mv rewrite with jg migration|
|_query_rewrite_maxdisjunct|257|TRUE|query rewrite max disjuncts|
|_query_rewrite_or_error|FALSE|TRUE|allow query rewrite, if referenced tables are not dataless|
|_query_rewrite_setopgrw_enable|TRUE|FALSE|perform general rewrite using set operator summaries|
|_query_rewrite_vop_cleanup|TRUE|TRUE|prune frocol chain before rewrite after view-merging|
|_queue_buffer_max_dump_len|65536|TRUE|max number of bytes to dump to trace file for queue buffer dump|
|_rbr_ckpt_tracing|0|TRUE|Enable reuse block range checkpoint tracing|
|_rcfg_disable_verify|TRUE|TRUE|if TRUE disables verify at reconfiguration|
|_rcfg_parallel_fixwrite|TRUE|TRUE|if TRUE enables parallel fixwrite at reconfiguration|
|_rcfg_parallel_replay|TRUE|TRUE|if TRUE enables parallel replay and cleanup at reconfiguration|
|_rcfg_parallel_verify|TRUE|TRUE|if TRUE enables parallel verify at reconfiguration|
|_rdbms_compatibility|10.1|TRUE|default RDBMS compatibility level|
|_rdbms_internal_fplib_enabled|FALSE|TRUE|enable CELL FPLIB filtering within rdbms|
|_rdbms_internal_fplib_raise_errors|FALSE|TRUE|enable reraising of any exceptions in CELL FPLIB|
|_readable_standby_sync_timeout|10|TRUE|readable standby query scn sync timeout|
|_real_time_apply_sim|0|TRUE|Simulation value with real time apply|
|_realfree_heap_max_size|32768|TRUE|minimum max total heap size, in Kbytes|
|_realfree_heap_mode|0|TRUE|mode flags for real-free heap|
|_realfree_heap_pagesize_hint|65536|TRUE|hint for real-free page size in bytes|
|_recoverable_recovery_batch_percent|50|TRUE|Recoverable recovery batch size (percentage of buffer cache)|
|_recovery_asserts|FALSE|TRUE|if TRUE, enable expensive integrity checks|
|_recovery_percentage|50|TRUE|recovery buffer cache percentage|
|_recovery_skip_cfseq_check|FALSE|TRUE|allow media recovery even if controlfile seq check fails|
|_recovery_verify_writes|FALSE|TRUE|enable thread recovery write verify|
|_recursive_imu_transactions|FALSE|TRUE|recursive transactions may be IMU|
|_recursive_with_max_recursion_level|0|TRUE|check for maximum level of recursion instead of checking for cycles|
|_redo_compatibility_check|FALSE|TRUE|general and redo/undo compatibility sanity check|
|_redo_read_from_memory|TRUE|TRUE|Enable reading redo from in-memory log buffer|
|_redo_transport_compress_all|TRUE|TRUE|Is ASYNC LNS compression allowed?|
|_redo_transport_max_stall_time|240|TRUE|Maximum stall time before redo tranport process is terminated|
|_redo_transport_stream_test|TRUE|TRUE|test stream connection?|
|_redo_transport_stream_writes|TRUE|TRUE|Stream network writes?|
|_redo_transport_vio_size_req|4294967294|TRUE|VIO size requirement|
|_reduce_sby_log_scan|TRUE|TRUE|enable stabdby log scan optimization|
|_release_insert_threshold|5|TRUE|maximum number of unusable blocks to unlink from freelist|
|_reliable_block_sends|FALSE|TRUE|if TRUE, block sends across interconnect are reliable|
|_remove_aggr_subquery|TRUE|FALSE|enables removal of subsumed aggregated subquery|
|_rep_base_path|%NULL%|TRUE|base path for EM reports in database|
|_replace_virtual_columns|TRUE|FALSE|replace expressions with virtual columns|
|_resource_manager_always_off|FALSE|TRUE|disable the resource manager always|
|_resource_manager_always_on|TRUE|TRUE|enable the resource manager always|
|_restore_maxopenfiles|8|TRUE|restore assumption for maxopenfiles|
|_restore_spfile|%NULL%|TRUE|restore spfile to this location|
|_result_cache_auto_execution_threshold|1|TRUE|result cache auto execution threshold|
|_result_cache_auto_size_threshold|100|TRUE|result cache auto max size allowed|
|_result_cache_auto_time_distance|300|TRUE|result cache auto time distance|
|_result_cache_auto_time_threshold|1000|TRUE|result cache auto time threshold |
|_result_cache_block_size|1024|TRUE|result cache block size|
|_result_cache_global|TRUE|TRUE|Are results available globally across RAC?|
|_result_cache_timeout|10|TRUE|maximum time (sec) a session waits for a result|
|_reuse_index_loop|5|TRUE|number of blocks being examine for index block reuse|
|_right_outer_hash_enable|TRUE|FALSE|Right Outer/Semi/Anti Hash Enabled|
|_rm_cluster_interconnects|%NULL%|TRUE|interconnects for RAC use (RM)|
|_rm_numa_sched_enable|TRUE|TRUE|Is Resource Manager (RM) related NUMA scheduled policy enabled|
|_rm_numa_simulation_cpus|0|TRUE|number of cpus for each pg for numa simulation in resource manager|
|_rm_numa_simulation_pgs|0|TRUE|number of PGs for numa simulation in resource manager|
|_rman_io_priority|3|TRUE|priority at which rman backup i/o's are done|
|_rman_restore_through_link|FALSE|TRUE|RMAN restore through link|
|_rollback_segment_count|0|TRUE|number of undo segments|
|_rollback_segment_initial|1|TRUE|starting undo segment number|
|_rollback_stopat|0|TRUE|stop at -position to step rollback|
|_row_cache_cursors|20|TRUE|number of cached cursors for row cache management|
|_row_cr|TRUE|TRUE|enable row cr for all sql|
|_row_locking|always|TRUE|row-locking|
|_row_shipping_explain|FALSE|TRUE|enable row shipping explain plan support|
|_row_shipping_threshold|80|TRUE|row shipping column selection threshold|
|_rowsource_execution_statistics|FALSE|TRUE|if TRUE, Oracle will collect rowsource statistics|
|_rowsource_profiling_statistics|TRUE|TRUE|if TRUE, Oracle will capture active row sources in v$active_session_history|
|_rowsource_statistics_sampfreq|128|TRUE|frequency of rowsource statistic sampling (must be a power of 2)|
|_rowsrc_trace_level|0|TRUE|Row source tree tracing level|
|_rta_sync_wait_timeout|10|TRUE|RTA sync wait timeout|
|_rtc_infeasible_threshold|25|TRUE|Redo Transport Compression infeasible threshold|
|_sample_rows_per_block|4|TRUE|number of rows per block used for sampling IO optimization|
|_scatter_gcs_resources|FALSE|TRUE|if TRUE, gcs resources are scattered uniformly across sub pools|
|_scatter_gcs_shadows|FALSE|TRUE|if TRUE, gcs shadows are scattered uniformly across sub pools|
|_scn_wait_interface_max_backoff_time_secs|600|TRUE|max exponential backoff time for scn wait interface in kta|
|_scn_wait_interface_max_timeout_secs|2147483647|TRUE|max timeout for scn wait interface in kta|
|_sdiag_crash|NONE|TRUE|sql diag crash|
|_sec_enable_test_rpcs|FALSE|TRUE|Whether to enable the test RPCs|
|_second_spare_parameter|%NULL%|TRUE|second spare parameter - integer|
|_securefile_timers|FALSE|TRUE|collect kdlu timers and accumulate per layers|
|_securefiles_concurrency_estimate|12|TRUE|securefiles concurrency estimate|
|_select_any_dictionary_security_enabled|FALSE|TRUE|Exclude user$ from SELECT ANY DICTIONARY system privilege|
|_selectivity_for_srf_enabled|FALSE|TRUE|enable/disable selectivity for storage reduction factor|
|_selfjoin_mv_duplicates|TRUE|FALSE|control rewrite self-join algorithm|
|_selftune_checkpoint_write_pct|3|TRUE|Percentage of total physical i/os for self-tune ckpt|
|_selftune_checkpointing_lag|300|TRUE|Self-tune checkpointing lag the tail of the redo log|
|_send_ast_to_foreground|TRUE|TRUE|if TRUE, send ast message to foreground|
|_send_close_with_block|TRUE|TRUE|if TRUE, send close with block even with direct sends|
|_send_requests_to_pi|TRUE|TRUE|if TRUE, try to send CR requests to PI buffers|
|_serial_direct_read|FALSE|TRUE|enable direct read in serial|
|_serial_recovery|FALSE|TRUE|force serial recovery or parallel recovery|
|_serializable|FALSE|TRUE|serializable|
|_serialize_lgwr_sync_io|FALSE|TRUE|Serialize LGWR SYNC local and remote io|
|_session_allocation_latches|1|TRUE|one latch per group of sessions|
|_session_cached_instantiations|60|TRUE|Number of pl/sql instantiations to cache in a session.|
|_session_context_size|10000|TRUE|session app context size|
|_session_idle_bit_latches|0|TRUE|one latch per session or a latch per group of sessions|
|_session_wait_history|10|TRUE|enable session wait history collection|
|_seventh_spare_parameter|%NULL%|TRUE|seventh spare parameter - string list|
|_sga_clear_dump|FALSE|TRUE|Allow dumping encrypted blocks in clear for debugging|
|_sga_early_trace|0|TRUE|sga early trace event|
|_sga_locking|none|TRUE|sga granule locking state|
|_shared_io_pool_buf_size|1048576|TRUE|Shared IO pool buffer size|
|_shared_io_pool_debug_trc|0|TRUE|trace kcbi debug info to tracefile|
|_shared_io_pool_size|0|TRUE|Size of shared IO pool|
|_shared_io_set_value|FALSE|TRUE|shared io pool size set internal value - overwrite zero user size|
|_shared_iop_max_size|536870912|TRUE|maximum shared io pool size|
|_shared_pool_max_size|0|TRUE|shared pool maximum size when auto SGA enabled|
|_shared_pool_minsize_on|FALSE|TRUE|shared pool minimum size when auto SGA enabled|
|_shared_pool_reserved_min_alloc|4400|TRUE|minimum allocation size in bytes for reserved area of shared pool|
|_shared_pool_reserved_pct|5|TRUE|percentage memory of the shared pool allocated for the reserved area|
|_shared_server_load_balance|0|TRUE|shared server load balance|
|_shared_server_num_queues|1|TRUE|number of shared server common queues|
|_shmprotect|0|TRUE|allow mprotect use for shared memory|
|_short_stack_timeout_ms|30000|TRUE|short stack timeout in ms|
|_shrunk_aggs_disable_threshold|60|TRUE|percentage of exceptions at which to switch to full length aggs|
|_shrunk_aggs_enabled|TRUE|TRUE|enable use of variable sized buffers for non-distinct aggregates|
|_side_channel_batch_size|200|TRUE|number of messages to batch in a side channel message (DFS)|
|_side_channel_batch_timeout|6|TRUE|timeout before shipping out the batched side channelmessages in seconds|
|_side_channel_batch_timeout_ms|500|TRUE|timeout before shipping out the batched side channelmessages in milliseconds|
|_simple_view_merging|TRUE|TRUE|control simple view merging performed by the optimizer|
|_simulate_disk_sectorsize|0|TRUE|Enables skgfr to report simulated disk sector size|
|_simulate_mem_transfer|FALSE|TRUE|simulate auto memory sga/pga transfers|
|_simulator_bucket_mindelta|8192|TRUE|LRU bucket minimum delta|
|_simulator_internal_bound|10|TRUE|simulator internal bound percent|
|_simulator_lru_rebalance_sizthr|5|TRUE|LRU list rebalance threshold (size)|
|_simulator_lru_rebalance_thresh|10240|TRUE|LRU list rebalance threshold (count)|
|_simulator_lru_scan_count|8|TRUE|LRU scan count|
|_simulator_pin_inval_maxcnt|16|TRUE|maximum count of invalid chunks on pin list|
|_simulator_reserved_heap_count|4096|TRUE|simulator reserved heap count|
|_simulator_reserved_obj_count|1024|TRUE|simulator reserved object count|
|_simulator_sampling_factor|2|TRUE|sampling factor for the simulator|
|_simulator_upper_bound_multiple|2|TRUE|upper bound multiple of pool size|
|_single_process|FALSE|TRUE|run without detached processes|
|_siop_flashback_scandepth|20|TRUE|Shared IO pool flashback io completion scan depth|
|_sixth_spare_parameter|%NULL%|TRUE|sixth spare parameter - string list|
|_skgxp_ctx_flags1|0|TRUE|IPC debug options flags (oss)|
|_skgxp_ctx_flags1mask|0|TRUE|IPC debug options flags mask (oss)|
|_skgxp_dynamic_protocol|0|TRUE|IPC protocol override (!0/-1=*,2=UDP,3=RDS,0x1000=ipc_X)|
|_skgxp_gen_ant_off_rpc_timeout_in_sec|30|TRUE|VRPC request timeout when ANT disabled|
|_skgxp_gen_ant_ping_misscount|3|TRUE|ANT protocol ping miss count|
|_skgxp_gen_rpc_no_path_check_in_sec|5|TRUE|ANT ping protocol miss count|
|_skgxp_gen_rpc_timeout_in_sec|300|TRUE|VRPC request timeout when ANT enabled|
|_skgxp_inets|%NULL%|TRUE|limit SKGXP networks|
|_skgxp_min_rpc_rcv_zcpy_len|0|TRUE|IPC threshold for rpc rcv zcpy operation (default = 0 - disabled)|
|_skgxp_min_zcpy_len|0|TRUE|IPC threshold for zcpy operation (default = 0 - disabled)|
|_skgxp_reaping|1000|TRUE|tune skgxp OSD reaping limit|
|_skgxp_rgn_ports|0|TRUE|region socket limits (0xFFFFNNXX): F=flags, N=min, X=max|
|_skgxp_spare_param1|%NULL%|TRUE|ipc oss spare parameter 1|
|_skgxp_spare_param2|%NULL%|TRUE|ipc oss spare parameter 2|
|_skgxp_spare_param3|%NULL%|TRUE|ipc oss spare parameter 3|
|_skgxp_spare_param4|%NULL%|TRUE|ipc oss spare parameter 4|
|_skgxp_spare_param5|%NULL%|TRUE|ipc oss spare parameter 5|
|_skgxp_udp_ach_reaping_time|120|TRUE|time in minutes before idle ach's are reaped|
|_skgxp_udp_ack_delay|0|TRUE|Enables delayed acks|
|_skgxp_udp_enable_dynamic_credit_mgmt|0|TRUE|Enables dynamic credit management|
|_skgxp_udp_hiwat_warn|1000|TRUE|ach hiwat mark warning interval|
|_skgxp_udp_interface_detection_time_secs|60|TRUE|time in seconds between interface detection checks|
|_skgxp_udp_keep_alive_ping_timer_secs|300|TRUE|connection idle time in seconds before keep alive is initiated. min: 30 sec max: 1800 sec default: 300 sec|
|_skgxp_udp_lmp_mtusize|0|TRUE|MTU size for UDP LMP testing|
|_skgxp_udp_lmp_on|FALSE|TRUE|enable UDP long message protection|
|_skgxp_udp_timed_wait_buffering|1024|TRUE|diagnostic log buffering space (in bytes) for timed wait (0 means unbufferd|
|_skgxp_udp_timed_wait_seconds|5|TRUE|time in seconds before timed wait is invoked|
|_skgxp_udp_use_tcb|TRUE|TRUE|disable use of high speek timer|
|_skgxp_zcpy_flags|0|TRUE|IPC zcpy options flags|
|_skgxpg_last_parameter|26|TRUE|last defined skgxpg parameter - oss|
|_skip_assume_msg|TRUE|TRUE|if TRUE, skip assume message for consigns at the master|
|_slave_mapping_enabled|TRUE|TRUE|enable slave mapping when TRUE|
|_slave_mapping_group_size|0|TRUE|force the number of slave group in a slave mapper|
|_slave_mapping_skew_ratio|2|TRUE|maximum skew before slave mapping is disabled|
|_small_table_threshold|520|TRUE|lower threshold level of table size for direct reads|
|_smm_advice_enabled|TRUE|TRUE|if TRUE, enable v$pga_advice|
|_smm_advice_log_size|0|TRUE|overwrites default size of the PGA advice workarea history log|
|_smm_auto_cost_enabled|TRUE|TRUE|if TRUE, use the AUTO size policy cost functions|
|_smm_auto_max_io_size|248|TRUE|Maximum IO size (in KB) used by sort/hash-join in auto mode|
|_smm_auto_min_io_size|56|TRUE|Minimum IO size (in KB) used by sort/hash-join in auto mode|
|_smm_bound|0|TRUE|overwrites memory manager automatically computed bound|
|_smm_control|0|TRUE|provides controls on the memory manager|
|_smm_freeable_retain|5120|TRUE|value in KB of the instance freeable PGA memory to retain|
|_smm_isort_cap|102400|TRUE|maximum work area for insertion sort(v1)|
|_smm_max_size|13107|FALSE|maximum work area size in auto mode (serial)|
|_smm_min_size|128|FALSE|minimum work area size in auto mode|
|_smm_px_max_size|32768|FALSE|maximum work area size in auto mode (global)|
|_smm_retain_size|0|TRUE|work area retain size in SGA for shared server sessions (0 for AUTO)|
|_smm_trace|0|TRUE|Turn on/off tracing for SQL memory manager|
|_smon_internal_errlimit|100|TRUE|limit of SMON internal errors|
|_smon_undo_seg_rescan_limit|10|TRUE|limit of SMON continous undo segments re-scan|
|_smu_debug_mode|0|TRUE|<debug-flag> - set debug event for testing SMU operations|
|_smu_error_simulation_site|0|TRUE|site ID of error simulation in KTU code|
|_smu_error_simulation_type|0|TRUE|error type for error simulation in KTU code|
|_smu_timeouts|%NULL%|TRUE|comma-separated *AND double-quoted* list of AUM timeouts: mql, tur, sess_exprn, qry_exprn, slot_intvl|
|_sort_elimination_cost_ratio|0|TRUE|cost ratio for sort eimination under first_rows mode|
|_sort_multiblock_read_count|2|TRUE|multi-block read count for sort|
|_sort_spill_threshold|0|TRUE|force sort to spill to disk each time this many rows are received|
|_space_align_size|1048576|TRUE|space align size|
|_spare_test_parameter|0|TRUE|Spare test parameter|
|_spawn_diag_opts|0|TRUE|thread spawn diagnostic options|
|_spawn_diag_thresh_secs|30|TRUE|thread spawn diagnostic minimal threshold in seconds|
|_spin_count|1|TRUE|Amount to spin waiting for a latch|
|_spr_max_rules|10000|TRUE|maximum number of rules in sql spreadsheet|
|_spr_push_pred_refspr|TRUE|TRUE|push predicates through reference spreadsheet|
|_spr_use_AW_AS|TRUE|TRUE|enable AW for hash table in spreadsheet|
|_spr_use_hash_table|FALSE|TRUE|use hash table for spreadsheet|
|_sql_analyze_enable_auto_txn|FALSE|TRUE|SQL Analyze Autonomous Transaction control parameter|
|_sql_analyze_parse_model|2|TRUE|SQL Analyze Parse Model control parameter|
|_sql_compatibility|0|TRUE|sql compatability bit vector|
|_sql_connect_capability_override|0|TRUE|SQL Connect Capability Table Override|
|_sql_connect_capability_table|%NULL%|TRUE|SQL Connect Capability Table (testing only)|
|_sql_hash_debug|0|TRUE|Hash value of the SQL statement to debug|
|_sql_model_unfold_forloops|RUN_TIME|FALSE|specifies compile-time unfolding of sql model forloops|
|_sql_ncg_mode|OFF|TRUE|Optimization mode for SQL NCG|
|_sql_plan_management_control|0|TRUE|controls various internal SQL Plan Management algorithms|
|_sqlexec_progression_cost|1000|TRUE|sql execution progression monitoring cost threshold|
|_sqlmon_binds_xml_format|default|TRUE|format of column binds_xml in [G]V$SQL_MONITOR|
|_sqlmon_max_plan|20|TRUE|Maximum number of plans entry that can be monitored. Defaults to 20 per CPU|
|_sqlmon_max_planlines|300|TRUE|Number of plan lines beyond which a plan cannot be monitored|
|_sqlmon_recycle_time|60|TRUE|Minimum time (in s) to wait before a plan entry can be recycled|
|_sqlmon_threshold|5|TRUE|CPU/IO time threshold before a statement is monitored. 0 is disabled|
|_sqltune_category_parsed|DEFAULT|FALSE|Parsed category qualifier for applying hintsets|
|_srvntfn_job_deq_timeout|60|TRUE|srvntfn job deq timeout|
|_srvntfn_jobsubmit_interval|3|TRUE|srvntfn job submit interval|
|_srvntfn_max_concurrent_jobs|20|TRUE|srvntfn max concurrent jobs|
|_srvntfn_q_msgcount|50|TRUE|srvntfn q msg count for job exit|
|_srvntfn_q_msgcount_inc|100|TRUE|srvntfn q msg count increase for job submit|
|_sscr_dir|%NULL%|TRUE|Session State Capture and Restore DIRectory object|
|_sscr_osdir|%NULL%|TRUE|Session State Capture and Restore OS DIRectory|
|_sta_control|0|TRUE|SQL Tuning Advisory control parameter|
|_stack_guard_level|0|TRUE|stack guard level|
|_standby_causal_heartbeat_timeout|2|TRUE|readable standby causal heartbeat timeout|
|_standby_flush_mode|SLFLUSH|TRUE|standby flush mode|
|_standby_switchover_timeout|120|TRUE|Number of secords for standby switchover enqueue timeout|
|_static_backgrounds|%NULL%|TRUE|static backgrounds|
|_statistics_based_srf_enabled|TRUE|TRUE|enable/disable the use of statistics for storage reduction factor|
|_step_down_limit_in_pct|1|TRUE|step down limit in percentage|
|_streams_pool_max_size|0|TRUE|streams pool maximum size when auto SGA enabled|
|_subquery_pruning_cost_factor|20|TRUE|subquery pruning cost factor|
|_subquery_pruning_enabled|TRUE|FALSE|enable the use of subquery predicates to perform pruning|
|_subquery_pruning_mv_enabled|FALSE|FALSE|enable the use of subquery predicates with MVs to perform pruning|
|_subquery_pruning_reduction|50|TRUE|subquery pruning reduction factor|
|_swrf_metric_frequent_mode|FALSE|TRUE|Enable/disable SWRF Metric Frequent Mode Collection|
|_swrf_mmon_dbfus|TRUE|TRUE|Enable/disable SWRF MMON DB Feature Usage|
|_swrf_mmon_flush|TRUE|TRUE|Enable/disable SWRF MMON FLushing|
|_swrf_mmon_metrics|TRUE|TRUE|Enable/disable SWRF MMON Metrics Collection|
|_swrf_on_disk_enabled|TRUE|TRUE|Parameter to enable/disable SWRF|
|_swrf_test_action|0|TRUE|test action parameter for SWRF|
|_swrf_test_dbfus|FALSE|TRUE|Enable/disable DB Feature Usage Testing|
|_synonym_repoint_tracing|FALSE|TRUE|whether to trace metadata comparisons for synonym repointing|
|_sysaux_test_param|1|TRUE|test parameter for SYSAUX|
|_system_api_interception_debug|FALSE|TRUE|enable debug tracing for system api interception|
|_system_index_caching|0|TRUE|optimizer percent system index caching|
|_system_trig_enabled|TRUE|TRUE|are system triggers enabled|
|_ta_lns_wait_for_arch_log|20|TRUE|LNS Wait time for arhcived version of online log|
|_table_lookup_prefetch_size|40|TRUE|table lookup prefetch vector size|
|_table_lookup_prefetch_thresh|2|TRUE|table lookup prefetch threshold|
|_table_scan_cost_plus_one|TRUE|FALSE|bump estimated full table scan and index ffs cost by one|
|_tablespaces_per_transaction|10|TRUE|estimated number of tablespaces manipulated by each transaction|
|_target_rba_max_lag_percentage|81|TRUE|target rba max log lag percentage|
|_tdb_debug_mode|16|TRUE|set debug mode for testing transportable database|
|_temp_tran_block_threshold|100|TRUE|number of blocks for a dimension before we temp transform|
|_temp_tran_cache|TRUE|TRUE|determines if temp table is created with cache option|
|_test_ksusigskip|5|TRUE|test the function ksusigskip|
|_test_param_1|25|TRUE|test parmeter 1 - integer|
|_test_param_2|%NULL%|TRUE|test parameter 2 - string|
|_test_param_3|%NULL%|TRUE|test parameter 3 - string|
|_test_param_4|%NULL%|TRUE|test parameter 4 - string list|
|_test_param_5|25|TRUE|test parmeter 5 - deprecated integer|
|_test_param_6|0|TRUE|test parmeter 6 - size (ub8)|
|_third_spare_parameter|%NULL%|TRUE|third spare parameter - integer|
|_threshold_alerts_enable|1|TRUE|if 1, issue threshold-based alerts|
|_timemodel_collection|TRUE|TRUE|enable timemodel collection|
|_timeout_actions_enabled|TRUE|TRUE|enables or disables KSU timeout actions|
|_timer_precision|10|TRUE|VKTM sleep time in milli-sec|
|_total_large_extent_memory|0|TRUE|Total memory for allocating large extents|
|_tq_dump_period|0|TRUE|time period for duping of TQ statistics (s)|
|_trace_buffer_wait_timeouts|0|TRUE|trace buffer busy wait timeouts|
|_trace_buffers|ALL:256|TRUE|trace buffer sizes per process|
|_trace_dump_all_procs|FALSE|TRUE|if TRUE on error buckets of all processes will be dumped to the current trace file|
|_trace_dump_client_buckets|TRUE|TRUE|if TRUE dump client (ie. non-kst) buckets|
|_trace_dump_cur_proc_only|FALSE|TRUE|if TRUE on error just dump our process bucket|
|_trace_dump_static_only|FALSE|TRUE|if TRUE filter trace dumps to always loaded dlls|
|_trace_events|%NULL%|TRUE|trace events enabled at startup|
|_trace_files_public|FALSE|TRUE|Create publicly accessible trace files|
|_trace_kqlidp|FALSE|TRUE|trace kqlidp0 operation|
|_trace_navigation_scope|global|TRUE|enabling trace navigation linking|
|_trace_pin_time|0|TRUE|trace how long a current pin is held|
|_trace_pool_size|%NULL%|TRUE|trace pool size in bytes|
|_trace_processes|ALL|TRUE|enable KST tracing in process|
|_trace_virtual_columns|FALSE|TRUE|trace virtual columns exprs|
|_transaction_auditing|TRUE|TRUE|transaction auditing records generated in the redo log|
|_transaction_recovery_servers|0|TRUE|max number of parallel recovery slaves that may be used|
|_transient_logical_clear_hold_mrp_bit|FALSE|TRUE|clear KCCDI2HMRP flag during standby recovery|
|_truncate_optimization_enabled|TRUE|TRUE|do truncate optimization if set to TRUE|
|_tsenc_tracing|0|TRUE|Enable TS encryption tracing|
|_tsm_connect_string|%NULL%|TRUE|TSM test connect string|
|_tsm_disable_auto_cleanup|1|TRUE|Disable TSM auto cleanup actions|
|_tstz_localtime_bypass|FALSE|TRUE|Should TTC not convert to LocalTime to preserve Timestamp with Timezone values|
|_tts_allow_charset_mismatch|FALSE|TRUE|allow plugging in a tablespace with an incompatible character set|
|_two_pass|TRUE|TRUE|enable two-pass thread recovery|
|_two_pass_reverse_polish_enabled|TRUE|TRUE|uses two-pass reverse polish alg. to generate canonical forms|
|_uga_cga_large_extent_size|262144|TRUE|UGA/CGA large extent size|
|_ultrafast_latch_statistics|TRUE|TRUE|maintain fast-path statistics for ultrafast latches|
|_undo_autotune|TRUE|TRUE|enable auto tuning of undo_retention|
|_undo_block_compression|TRUE|TRUE|enable undo block compression|
|_undo_debug_mode|0|TRUE|debug flag for undo related operations|
|_undo_debug_usage|0|TRUE|invoke undo usage functions for testing|
|_union_rewrite_for_gs|YES_GSET_MVS|FALSE|expand queries with GSets into UNIONs for rewrite|
|_unnest_subquery|TRUE|FALSE|enables unnesting of complex subqueries|
|_unused_block_compression|TRUE|TRUE|enable unused block compression|
|_update_datafile_headers_with_space_information|FALSE|TRUE|user requested update of datafile headers of locally managed datafiles with space information|
|_use_adaptive_log_file_sync|FALSE|TRUE|Adaptively switch between post/wait and polling|
|_use_best_fit|FALSE|TRUE|use best fit to allocate space|
|_use_column_stats_for_function|TRUE|FALSE|enable the use of column statistics for DDP functions|
|_use_ism|TRUE|TRUE|Enable Shared Page Tables - ISM|
|_use_ism_for_pga|TRUE|TRUE|Use ISM for allocating large extents|
|_use_nosegment_indexes|FALSE|TRUE|use nosegment indexes in explain plan|
|_use_platform_compression_lib|FALSE|TRUE|Enable platform optimized compression implementation|
|_use_realfree_heap|TRUE|TRUE|use real-free based allocator for PGA memory|
|_use_seq_process_cache|TRUE|TRUE|whether to use process local seq cache|
|_use_vector_post|TRUE|TRUE|use vector post|
|_use_zero_copy_io|TRUE|TRUE|Should network vector IO interface be used for data transfer|
|_validate_flashback_database|FALSE|TRUE|Scan database to validate result of flashback database|
|_validate_readmem_redo|OFF|TRUE|validate redo blocks read from in-memory log buffer|
|_vendor_lib_loc|%NULL%|TRUE|Vendor library search root directory|
|_verify_flashback_redo|TRUE|TRUE|Verify that the redo logs needed for flashback are available|
|_verify_undo_quota|FALSE|TRUE|TRUE - verify consistency of undo quota statistics|
|_very_large_object_threshold|500|TRUE|upper threshold level of object size for direct reads|
|_very_large_partitioned_table|1024|TRUE|very_large_partitioned_table|
|_virtual_column_overload_allowed|TRUE|TRUE|overload virtual columns expression|
|_vkrm_schedule_interval|10|TRUE|VKRM scheduling interval|
|_wait_breakup_threshold_csecs|600|TRUE|Wait breakup threshold (in centiseconds)|
|_wait_breakup_time_csecs|300|TRUE|Wait breakup time (in centiseconds)|
|_wait_for_sync|TRUE|TRUE|wait for sync on commit MUST BE ALWAYS TRUE|
|_wait_samples_max_sections|40|TRUE|Wait Samples maximum sections|
|_wait_samples_max_time_secs|120|TRUE|Wait Samples maximum time in seconds|
|_wait_tracker_interval_secs|10|TRUE|Wait Tracker number of seconds per interval|
|_wait_tracker_num_intervals|0|TRUE|Wait Tracker number of intervals|
|_walk_insert_threshold|0|TRUE|maximum number of unusable blocks to walk across freelist|
|_watchpoint_on|FALSE|TRUE|is the watchpointing feature turned on?|
|_wcr_control|0|TRUE|Oracle internal test WCR parameter used ONLY for testing!|
|_windowfunc_optimization_settings|0|TRUE|settings for window function optimizations|
|_with_subquery|OPTIMIZER|TRUE|WITH subquery transformation|
|_write_clones|3|TRUE|write clones flag|
|_xpl_peeked_binds_log_size|8192|TRUE|maximum bytes for logging peeked bind values for V$SQL_PLAN (0 = OFF)|
|_xpl_trace|0|TRUE|Explain Plan tracing parameter|
|_xsolapi_auto_materialization_bound|20|TRUE|OLAP API lower bound for auto materialization.|
|_xsolapi_auto_materialization_type|PRED_AND_RC|TRUE|OLAP API behavior for auto materialization|
|_xsolapi_build_trace|FALSE|TRUE|OLAP API output build info to trace file |
|_xsolapi_debug_output|SUPPRESS|TRUE|OLAP API debug output disposition|
|_xsolapi_densify_cubes|TABULAR|TRUE|OLAP API cube densification|
|_xsolapi_dimension_group_creation|OVERFETCH|TRUE|OLAP API symmetric overfetch|
|_xsolapi_dml_trace|%NULL%|TRUE|OLAP API output dml commands and expressions to trace file |
|_xsolapi_fetch_type|PARTIAL|TRUE|OLAP API fetch type|
|_xsolapi_fix_vptrs|TRUE|TRUE|OLAP API Enable vptr fixing logic in shared server mode|
|_xsolapi_generate_with_clause|FALSE|TRUE|OLAP API generates WITH clause?|
|_xsolapi_hierarchy_value_type|unique|TRUE|OLAP API hierarchy value type|
|_xsolapi_load_at_process_start|NEVER|TRUE|When to load OLAP API library at server process start|
|_xsolapi_materialization_rowcache_min_rows_for_use|1|TRUE|OLAP API min number of rows required to use rowcache in query materialization|
|_xsolapi_materialize_sources|TRUE|TRUE|OLAP API Enable source materialization|
|_xsolapi_metadata_reader_mode|DEFAULT|TRUE|OLAP API metadata reader mode|
|_xsolapi_odbo_mode|FALSE|TRUE|OLAP API uses ODBO mode?|
|_xsolapi_opt_aw_position|TRUE|TRUE|OLAP API enables AW position and count optimization?|
|_xsolapi_optimize_suppression|TRUE|TRUE|OLAP API optimizes suppressions?|
|_xsolapi_precompute_subquery|TRUE|TRUE|OLAP API precomputes subqueries?|
|_xsolapi_remove_columns_for_materialization|TRUE|TRUE|OLAP API removes columns for materialization?|
|_xsolapi_set_nls|TRUE|TRUE|OLAP API sets NLS?|
|_xsolapi_share_executors|TRUE|TRUE|OLAP API share executors?|
|_xsolapi_source_trace|FALSE|TRUE|OLAP API output Source definitions to trace file |
|_xsolapi_sql_all_multi_join_non_base_hints|%NULL%|TRUE|OLAP API multi-join non-base hints|
|_xsolapi_sql_all_non_base_hints|%NULL%|TRUE|OLAP API non-base hints|
|_xsolapi_sql_auto_dimension_hints|FALSE|TRUE|OLAP API enable automatic dimension hints|
|_xsolapi_sql_auto_measure_hints|TRUE|TRUE|OLAP API enable automatic measure hints|
|_xsolapi_sql_dimension_hints|%NULL%|TRUE|OLAP API dimension hints|
|_xsolapi_sql_enable_aw_join|TRUE|TRUE|OLAP API enables AW join?|
|_xsolapi_sql_enable_aw_qdr_merge|TRUE|TRUE|OLAP API enables AW QDR merge?|
|_xsolapi_sql_hints|%NULL%|TRUE|OLAP API generic hints|
|_xsolapi_sql_measure_hints|%NULL%|TRUE|OLAP API measure hints|
|_xsolapi_sql_minus_threshold|1000|TRUE|OLAP API SQL MINUS threshold|
|_xsolapi_sql_optimize|TRUE|TRUE|OLAP API enable optimization|
|_xsolapi_sql_prepare_stmt_cache_size|16|TRUE|OLAP API prepare statement cache size|
|_xsolapi_sql_remove_columns|TRUE|TRUE|OLAP API enable remove unused columns optimizations|
|_xsolapi_sql_result_set_cache_size|32|TRUE|OLAP API result set cache size|
|_xsolapi_sql_symmetric_predicate|TRUE|TRUE|OLAP API enable symmetric predicate for dimension groups|
|_xsolapi_sql_top_dimension_hints|%NULL%|TRUE|OLAP API top dimension hints|
|_xsolapi_sql_top_measure_hints|%NULL%|TRUE|OLAP API top measure hints|
|_xsolapi_sql_use_bind_variables|TRUE|TRUE|OLAP API enable bind variables optimization|
|_xsolapi_stringify_order_levels|FALSE|TRUE|OLAP API stringifies order levels?|
|_xsolapi_support_mtm|FALSE|TRUE|OLAP API MTM mapping classes supported?|
|_xsolapi_suppression_aw_mask_threshold|1000|TRUE|OLAP API suppression AW mask threshold|
|_xsolapi_suppression_chunk_size|4000|TRUE|OLAP API suppression chunk size|
|_xsolapi_use_models|TRUE|TRUE|OLAP API uses models?|
|_xsolapi_use_olap_dml|TRUE|TRUE|OLAP API uses OLAP DML?|
|_xsolapi_use_olap_dml_for_rank|TRUE|TRUE|OLAP API uses OLAP DML for rank?|
|_xt_coverage|none|TRUE|external tables code coverage parameter|
|_xt_trace|none|TRUE|external tables trace parameter|
|_xtbuffer_size|0|TRUE|buffer size in KB needed for populate/query operation|
|_xtts_allow_pre10|FALSE|TRUE|allow cross platform for pre10 compatible tablespace|
|_xtts_set_platform_info|FALSE|TRUE|set cross platform info during file header read|
