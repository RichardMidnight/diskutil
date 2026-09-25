# diskutil — function audit ledger

Process: leaf functions reviewed **one at a time, in file order**, each accepted (or
rejected) individually. Status: `PASS` = reviewed, no change · `FIX` = reviewed,
fix accepted+committed · `REJ` = reviewed, change rejected · `—` = not yet reviewed.

## Audit queue (file order)

| # | Function | Line | Status | Notes |
|---|----------|------|--------|-------|
| 1 | `echo_white` / `echo_red` / `echo_green` / `echo_blue` | 49–52 | — | one 4-line pattern |
| 2 | `echo_debug` | 55 | FIX | added `DEBUG=off` setting + guard (on = blue stderr as before) |
| 3 | `run_command` | 60 | PASS | audited in resize pass (rc + stdout semantics) |
| 4 | `file_path` | 89 | PASS | one-liner `dirname "$*"`; only caller passes $LOG_FILE (single arg) |
| 5 | `info` | 94 | PASS | trivially correct but DEAD (no callers; baseline SC2317 — left per policy) |
| 6 | `env_installer` | 99 | FIX | unknown-distro empty-result → clear error + return 1 |
| 7 | `env_removal` | 120 | FIX | same unknown-distro guard as #6 |
| 8 | `env_package_in_use` | 139 | FIX | apt chain-detection was dead (1-space indent test vs 2-space real; flowed list lines) → tokenize + self-exclude. Verified: libc-bin IN-USE (cloud-init,locales), bash IN-USE (essential), exfatprogs not-in-use. Non-apt distros still unguarded (known limitation, no change) |
| 9 | `env_root_device` | 162 | PASS | audited in initDisk pass (410db81) |
| 10 | `env_root_disk` | 168 | PASS | audited in initDisk pass (410db81) |
| 10b | `env_which` | 183 | FIX | triple-sequential apt/pacman/dnf install (no detection; broken on zypper/xbps, double-fail noise on Debian) → single per-distro install via audited `env_installer`; stdout kept quiet |
| 11 | `env_install_tools` | 203 | FIX | removed stray trailing `-y` after package name in `udisksctl`) branch (broke pacman; flag already in installer string) |
| 12 | `env_list_filesystems` | 236 | PASS | all 16 $FILESYSTEMS tokens mapped (vfat×3, mkswap×2, hfs+ literal, `*` rest); live run clean. `hfs`→`mkfs.hfs` is a $FILESYSTEMS data quirk the function honestly reports as Not installed |
| 13 | `env_install_smarttools` | 265 | PASS | DEAD (no callers). Landmine noted: if ever enabled it runs all 5 distro sudo installs unconditionally (no -y flags) — fix with `env_installer` pattern when/if it's wired up |
| 14 | `media_filesystem_install` | 282 | PASS | FS support pass (6155a6e c631283) |
| 15 | `media_filesystem_uninstall` | 460 | PASS | FS pass + in-use gate (c631283) |
| 16 | `do_beep` | 556 | FIX | FREQ/TIME were global; made local (no lint cost) |
| 17 | `do_beep_up` | 567 | — | |
| 18 | `do_beep_down` | 573 | — | |
| 19 | `get_elapsed_time` | 579 | — | |
| 20 | `do_countdown` | 588 | — | |
| 21 | `get_ver_to_int` | 614 | — | |
| 22 | `is_number` | 624 | PASS | audited in earlier quoting/bugfix passes |
| 23 | `info_validate_num` | 646 | — | |
| 24 | `bytes` | 678 | PASS | read in resize pass (base/suffix logic) |
| 25 | `ui_countdown` | 769 | — | |
| 26 | `ui_echo` | 787 | — | |
| 27 | `ui_log` | 808 | — | |
| 28 | `ui_msg` | 825 | — | |
| 29 | `ui_msg_error` | 847 | — | |
| 30 | `ui_msg_warning` | 857 | — | |
| 31 | `ui_yesno` | 868 | PASS | getargs/-y and TTY semantics verified |
| 32 | `media_device_fullname` | 941 | — | |
| 33 | `media_device_partname` | 959 | — | |
| 34 | `media_device_partnum` | 971 | — | |
| 35 | `media_device_size` | 988 | — | |
| 36 | `media_device_type` | 997 | — | |
| 37 | `media_device_name` | 1012 | — | |
| 38 | `media_disk_friendlyname` | 1044 | — | |
| 39 | `media_disk_pt_type` | 1054 | — | |
| 40 | `media_device_checktarget` | 1070 | PASS | initDisk pass (live part guard / dead disk guard analyzed) |
| 41 | `media_disk_model` | 1127 | — | |
| 42 | `media_disk_serial` | 1143 | — | |
| 43 | `media_disk_sectorsize` | 1151 | — | |
| 44 | `media_disk_blocksize` | 1158 | — | |
| 45 | `media_disk_listdisks` | 1167 | — | |
| 46 | `media_disk_align` | 1190 | — | |
| 47 | `media_disk_first_available_byte` | 1227 | — | |
| 48 | `media_filesystem_ismounted` | 1281 | — | |
| 49 | `media_filesystem_mountpoint` | 1304 | — | |
| 50 | `media_filesystem_min` | 1333 | — | mostly read (fat/exfat/ntfs); reconfirm rest |
| 51 | `media_filesystem_name` | 1447 | — | |
| 52 | `media_filesystem_size_alt` | 1474 | — | |
| 53 | `media_filesystem_size` | 1496 | — | mostly read (resize pass); reconfirm edges |
| 54 | `media_filesystem_used` | 1605 | — | |
| 55 | `media_disk_partition_list` | 1628 | — | |
| 56 | `media_disk_partcount` | 1649 | — | |
| 57 | `media_partition_record` | 1677 | — | |
| 58 | `media_partition_start` | 1686 | — | |
| 59 | `media_partition_end` | 1713 | — | |
| 60 | `media_partition_size` | 1743 | — | |
| 61 | `media_partition_max` | 1777 | — | |
| 62 | `media_disk_info` | 1881 | — | |
| 63 | `media_disk_verify` | 1917 | — | |
| 64 | `media_disk_repair` | 1971 | — | |
| 65 | `media_disk_initialize` | 1996 | PASS | initDisk review (410db81) |
| 66 | `media_disk_addpartition` | 2059 | — | mostly (guard 410db81, geometry); reconfirm rest |
| 67 | `media_disk_delpartition` | 2261 | — | |
| 68 | `media_disk_mount` | 2314 | — | |
| 69 | `media_disk_unmount` | 2348 | — | |
| 70 | `media_disk_eject` | 2391 | — | |
| 71 | `media_disk_format` | 2417 | — | |
| 72 | `media_device_wipe` | 2555 | — | FLAGGED (from #9 call-site walk): line 2628 `[[ $DISK = $env_root_device ]]` references a VARIABLE (empty) → root-disk guard is dead; `wipe mmcblk0 disk` would pass the guard. Fix belongs to this item: `$(media_device_name "$(env_root_device)")` |
| 73 | `retire_maybe_media_partition_pt_type` | 2683 | — | |
| 74 | `media_partition_fs_type` | 2694 | — | read (parted→lsblk fallback); formal PASS |
| 75 | `media_partition_format` | 2758 | — | |
| 76 | `media_partition_reformat` | 2908 | — | |
| 77 | `media_partition_resize` | 2941 | PASS | resize pass (ac8f5e1) |
| 78 | `media_partition_alignment` | 3070 | — | |
| 79 | `media_partition_check` | 3098 | — | |
| 80 | `media_filesystem_check` | 3214 | — | |
| 81 | `media_filesystem_repair` | 3479 | — | |
| 82 | `media_filesystem_rename` | 3660 | — | |
| 83 | `media_filesystem_resize` | 3714 | PASS | resize pass (ac8f5e1) |
| 84 | `media_partition_os` | 3987 | — | |
| 85 | `media_disk_os` | 4097 | — | |
| 86 | `media_volume_mount` | 4124 | — | |
| 87 | `media_volume_unmount` | 4186 | — | |
| 88 | `media_filesystem_info` | 4247 | — | |
| 89 | `media_volume_badblocks` | 4375 | — | |
| 90 | `media_volume_add` | 4410 | — | mostly (ENOSPC 1ffd7d5 + echo gates); reconfirm rest |
| 91 | `media_volume_resize` | 4490 | PASS | audited in resize pass |
| 92 | `getargs` | 4671 | PASS | YESNO/-y/TTY semantics verified |
| 93 | `media_volume_verify` | 4696 | — | |
| 94 | `media_volume_repair` | 4773 | — | |
| 95 | `media_clone` | 4813 | — | |
| 96 | `whiptail_calc_wt_size` | 4870 | — | |
| 97 | `menu` | 4899 | — | FLAGGED: "Change settings" (4928) calls `menu_settings` which is NOT defined |
| 98 | `diskutil_install` | 4947 | — | |
| 99 | `diskutil_update` | 4966 | — | dead-local-YES fixed (ac8f5e1); rest reconfirm |
| 100 | `diskutil_terminology` | 5015 | — | |
| 101 | `diskutil_help` | 5035 | — | |

## Log

| Date | Item | Action |
|------|------|--------|
| 2026-09 | setup | ledger created; PASS = prior passes (6155a6e, c631283, 1ffd7d5, 410db81, ac8f5e1) |
| 2026-09 | #1 color echoes (49–52) | PASS, no change |
| 2026-09 | #2 echo_debug (55) | FIX: DEBUG=on/off toggle (default off, rc-0 no-op), overridable via env |
| 2026-9 | #6 env_installer | FIX: empty-result guard for unknown distro (f7b6add) |
| 2026-9 | #7 env_removal | FIX: same empty-result guard (b9590ac) |
| 2026-9 | #8 env_package_in_use | FIX: apt chain-detection awk (indent + flowed lines), tokenized self-exclusion |
| 2026-9 | #10b env_which | FIX: per-distro `which` install via env_installer (was unconditional apt+pacman+dnf) |
| 2026-9 | #11 env_install_tools | FIX: stray trailing `-y` after package (pacman-breaking) removed |
| 2026-9 | #12 env_list_filesystems | PASS | all mappings verified; live run clean |
| 2026-9 | #13 env_install_smarttools | PASS | DEAD (no callers); landmine noted for future |
| 2026-9 | #16 do_beep | FIX | FREQ/TIME made local |
