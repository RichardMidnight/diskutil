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
| 17 | `do_beep_up` | 567 | PASS | ascending chirp via audited do_beep; names/behavior match |
| 18 | `do_beep_down` | 573 | PASS | descending chirp, ditto |
| 19 | `get_elapsed_time` | 579 | PASS | DEAD (no callers); arithmetic sound if ever used; left per policy |
| 20 | `do_countdown` | 588 | FIX | added `local INPUT`; deleted dead `MSG=$(echo "$MSG.$i")` (undefined $i, unused). y/other/timeout rc 0/2/0 verified; lint -2 (SC2116, SC2154) |
| 21 | `get_ver_to_int` | 614 | FIX | parts/val were global → local (leak proven+fixed); behavior verified 1.2.3→1002003, 1.2→1002000, abc→0 |
| 22 | `is_number` | 624 | PASS | audited in earlier quoting/bugfix passes |
| 23 | `info_validate_num` | 646 | PASS | DEAD (no callers; only user of the ERRORS counter, which is otherwise unused). If ever enabled: MIN/MAX-violation paths log but don't return 1 (inconsistent with not-a-number path) — note for later |
| 24 | `bytes` | 678 | FIX | TB→bytes multiplier was `*2014` (typo) → `*1024`: 1tb was 2162516033536 (1.9667×), now 1099511627776; kb/mb/gb untouched. (reconfirmed earlier PASS verdict — this line had been missed) |
| 25 | `ui_countdown` | 769 | FIX | `local SECONDS` rebased shell $SECONDS (special var) → renamed local to CNT (no in-script consumer, but source-environment hazard). whiptail_countdown undefined = flagged, left (non-cli branch) |
| 26 | `ui_echo` | 801 | PASS | MSG/COLOR global scratch = established sibling idiom (ui_msg/ui_msg_error/ui_yesno all set-then-read); LOGIT local; colors defined @50-53; stderr routing is by design (protects captured stdout) |
| 27 | `ui_log` | 822 | PASS | `\n`→` - ` flatten fine; dir-mkdir runs even when LOG=off (harmless, pre-existing) |
| 28 | `ui_msg` | 839 | PASS | cli path clean; whiptail branch dormant (whiptail_countdown class, flagged); "" 4th-arg only touches dormant whiptail path |
| 29 | `ui_msg_error` | 861 | PASS | standard linewrap; do_beep_down already local-fixed (#16); logs with LINENO |
| 30 | `ui_msg_warning` | 871 | PASS | double beep + nolog = by design |
| 31 | `ui_yesno` | 868 | PASS | getargs/-y and TTY semantics verified |
| 32 | `media_device_fullname` | 941 | PASS | empty→rc1 ✓; /dev/ prefix test ✓ |
| 33 | `media_device_partname` | 959 | PASS | DEAD (0 callers); quirk noted: whole-disk ARM name would yield `p0` — moot while unused |
| 34 | `media_device_partnum` | 971 | FIX | guard `! [[ $(is_number ...) ]]` could never fire (non-empty-string test always true) → `! is_number "$PARTNUM" >/dev/null` (rc-based, no output execution, no new lint). sda/abc now rc1; sda10→10, mmcblk0p12→12 exact |
| 35 | `media_device_size` | 988 | PASS | lsblk -b -n -d -o size clean; empty input fails safely |
| 36 | `media_device_type` | 997 | PASS | 0.5s retry on empty = udev-race handling; empty-output contract consistent |
| 37 | `media_device_name` | 1012 | FIX | grep-10 branch stripped ALL digits (mmcblk0p10→/dev/mmcblkp, nvme0n1p10→mangled) → anchored `s/[1-9][0-9]*$//` (+p variant); 11-case matrix now exact, embedded zeros preserved; lint -2 (SC2001/SC2143, ref→final17) |
| 38 | `media_disk_friendlyname` | 1044 | PASS | composes two audited helpers |
| 39 | `media_disk_pt_type` | 1054 | PASS | parted -m f6 = PTTYPE ✓ |
| 40 | `media_device_checktarget` | 1070 | PASS | initDisk pass (live part guard / dead disk guard analyzed) |
| 41 | `media_disk_model` | 1127 | PASS | lsblk + parted-Model fallback |
| 42 | `media_disk_serial` | 1143 | FIX | was emitting vendor+model+serial line → `-o serial` only (both call sites are display lines; verified: mmcblk0/sda1 now return bare serial) |
| 43 | `media_disk_sectorsize` | 1151 | PASS | DEAD (no callers); `blockdev --getss` correct — left per policy |
| 44 | `media_disk_blocksize` | 1158 | PASS | `--getbsz` bytes as all 6 consumers expect (comment "4095" = cosmetic typo for 4096) |
| 45 | `media_disk_listdisks` | 1167 | PASS | TYPE=disk filter correct |
| 46 | `media_disk_align` | 1190 | PASS | `$(is_number …) = true` string idiom correct; 100% passthrough + clamp by design; roundup idiom correct |
| 47 | `media_disk_first_available_byte` | 1227 | FIX | fresh-disk bogus `religned from  to 1048576` debug line (empty→0 arithmetic) → `-n` guard. Alignment math verified exact across boundaries |
| 48 | `media_filesystem_ismounted` | 1281 | FIX | contract `1`/empty works for 16/18 sites; **2 dead `= true` safety guards fixed with it**: 3524 (repair-on-mounted-fs guard, #81) + 4614 (resize dispatch mounted guard, #95+). root-branch rc quirk harmless (no caller reads rc) |
| 49 | `media_filesystem_mountpoint` | 1304 | PASS | mount/detect/unmount cycle correct; `WAS_MOUNTED` arithmetic clean |
| 49b | `media_filesystem_record` | 1323 | PASS | (queue-gap, like #10b). Global `$PARTITION` set by its single caller (4300) — works, sloppy idiom; mount/detect/unmount cycle correct; `local FS_RECORD=$(lsblk …)` baseline SC2155 |
| 50 | `media_filesystem_min` | 1333 | PASS | per-fs min logic correct (fat floor, ext2fs -P, ntfsresize/ntfs, btrfs min-dev-size, no-shrink=size). Notes: `1025*1024*10` padding (10,240B over "10M", left), fat-guard `exit` aggressive, 1399 is_number tests always-true (degrades safely) |
| 51 | `media_filesystem_name` | 1454 | PASS | label via lsblk with 2×0.5s retries; empty = no label (callers at 2935/3685/3716 handle) |
| 52 | `media_filesystem_size_alt` | 1481 | PASS | mount→lsblk fssize→restore state; re-mount in the already-mounted branch is a benign no-op (end state correct) |
| 53 | `media_filesystem_size` | 1496 | FIX | xfs mount-restore was INVERTED (unmounted user's mounted vol, left unmounted vols mounted; 9 call sites) → `[[ -z $WAS_MOUNTED ]]`. Also `$currentsize`→`$blockcount` (ext guard typo, inert). lint -2 (SC2004/SC2154, ref→final18) |
| 54 | `media_filesystem_used` | 1608 | PASS | fat-only; `bytes/cluster`(f2) × `N/M` used-clusters parse self-consistent; sandbox blocks live test; only caller is #50 fat branch (hard-exits if non-numeric) |
| 55 | `media_disk_partition_list` | 1635 | PASS | parted `-m print` in on-disk order ✓; `NAME` unused (baseline SC2034); `PARTITIONS` leaks global (minor) |
| 56 | `media_disk_partcount` | 1656 | PASS | `${#PARTITIONS[@]}` count, 0 when empty ✓ |
| 57 | `media_partition_record` | 1684 | PASS | DEAD — 0 live callers (3198 commented); `$DEVICE` is properly defined via media_device_name |
| 58 | `media_partition_start` | 1693 | FIX | per-part `cut -f2` ✓; whole-disk branch returned disk SIZE as start → now `1` |
| 59 | `media_partition_end` | 1720 | FIX | per-part `cut -f3` ✓; whole-disk branch returned `1` as end → now disk size (last byte); #61's 0-partition path consistent again |
| 60 | `media_partition_size` | 1750 | FIX | per-part `cut -f4` ✓; whole-disk branch returned `1` as size → now disk size |
| 61 | `media_partition_max` | 1784 | PASS | with #59 fix, 0-partition path returns disk end as commented; free-space extension logic ✓; `PART_START` unused (baseline) |
| 61b | `media_disk_test` | 1838 | FIX | (queue gap, like #10b/#49b). prompt used `$PARTCOUNT` before assignment (was empty at 1858; "partitons" typo) → compute early + typo; loop built `mmcblk02`/`nvme0n12` → `p` prefix now. notes: `(( $? ))` after `echo $(friendlyname…)` dead (echo's rc); checktarget error keeps going (no return) |
| 62 | `media_disk_info` | 1888 | PASS | display-only (parted print free, blocksize, lsblk -t/-f); verbose gating ✓ |
| 63 | `media_disk_verify` | 1924 | FIX | 1970 `(( ! $(ui_yesno …)))` was INVERTED (YES skipped, NO ran the verify loop) → un-negated (every other `!` site is a cancel-guard — correct); 1972 loop now mmcblk/nvme-safe; pt_type case + rc checks ✓; "initialied" typo noted |
| 64 | `media_disk_repair` | 1978 | PASS | delegates to verify; `$YESNO` passed as arg re-parses harmlessly via getargs ("no changes made" matches its comment) |
| 65 | `media_disk_initialize` | 1996 | PASS | initDisk review (410db81) |
| 66 | `media_disk_addpartition` | 2068 | FIX | fat16 `max` cap never applied (`END=100%` parsed as `100 % -BEG`→100, always < 4G) → `max` capped at BEG+4090M, numeric path is_number-guarded; dead `(( $VERBOSE ))` @2221/2262 (always false — `(( -v ))`=0) → `[[ $VERBOSE = -v ]]`. notes: 2120 BLOCK_SIZE lookup dead (1MiB GPT hardcode, intentional); `$NAME` arg unused (mkpart name=`$PART_TYPE`); 2123-26 LAST_PARTITION/PART_NUM/PARTITION vestigial (num re-derived from `parted -m print` ✓); `grep "$BEG"` could match end==new start (edge); 2258 + consumer 4455 DISK+num naming (4455 queued) |
| 67 | `media_disk_delpartition` | 2274 | FIX | 2285 `PARTITION=$DEVICE$PARTNUM` built `mmcblk02`/`nvme0n11` (invalid) → checktarget@2296 rejected → delPartition failed on ALL SD/NVMe disks; now mmcblk/nvme-safe name (parted `rm` keeps bare number ✓). 2321 `$PART_NO` undef → `$PARTNUM` (SC2153 −1). note: final `parted rm` has no explicit rc check (implicit) |
| 68 | `media_disk_mount` | 2327 | FIX | 2356 `"$DISK""$PART"` → `mmcblk02` in live mount loop → `p$PART` prefix; count=0 → whole-disk-volume branch correct ✓ |
| 69 | `media_disk_unmount` | 2361 | FIX | 2377 `exit 1` → `return 1` (killed session; siblings use return). live path sound (df-list + multi-device umount). notes: 2390-2401 DEAD block (after `return`, has invalid `umount -q`) — left per dead-code rule; unanchored `grep "$DISK"` safe (disk name = partition prefix) |
| 70 | `media_disk_eject` | 2406 | PASS | yesno gate ✓ (action-body, no negation); unmount + rc ✓; `udisksctl power-off` last (implicit rc propagates). note: usage-guard 2409 `exit` where siblings `return` |
| 71 | `media_disk_format` | 2432 | FIX | 2509 `PARTITION=$DEVICE$PART_NUM` → `/dev/mmcblk01` invalid → Step 3 format + mount failed on SD/NVMe → mmcblk/nvme-safe name (`/dev/`-prefixed form); Step 3 had NO rc check (failed format still said "Step3: DONE" + mounted) → `\|\| return 1`. notes: ROOT 2459 + TYPE 2460 dead (root refusal via nested initDisk ✓); SIZE double-declared (2447/2483); flow unmount→init→add(max)→format→mount ✓ |
| 71b | `media_device_info` | 2526 | PASS | queue gap (3rd: #10b/#49b/#61b → this). checktarget no-type (either ok) ✓; part branch → filesystem_info + partition_os ✓. note: 2529 fullname evaluated BEFORE the `-z $1` guard (2533) — harmless (fullname has own empty-check); live from `info*` @5168 |
| 72 | `media_device_wipe` | 2571 | FIX | FLAG (from #9) confirmed: 2635 `[[ $DISK = $env_root_device ]]` read an UNSET VARIABLE (`env_root_device` is the function) → root-disk guard DEAD (would wipe the boot disk) → `$(env_root_disk)`, same idiom as 2037/2164. SC2154 −1, SC2053 −1 |
| 73 | `retire_maybe_media_partition_pt_type` | 2699 | PASS–DEAD | would work if called (`-m` disk-header line; **f6 = PTTYPE verified empirically** `f6=msdos`; `grep "$DISK"` matches only the disk line — partition lines start with digits) but ZERO live callers (def only; name already says retire) |
| 74 | `media_partition_fs_type` | 2710 | PASS | parted f5=FS ✓ (`^$PARTNUM:` anchor safe at 10+; verified on image) → lsblk×3 fallback; `echo_debug`→stderr (capture-safe); rc 1 if empty ✓; 15 live callers. note: triple identical lsblk retry redundant (harmless) |
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
| 2026-9 | #20 do_countdown | FIX | `local INPUT`; deleted dead `MSG=$(echo "$MSG.$i")` |
| 2026-9 | #21 get_ver_to_int | FIX | parts/val made local (were leaking global) |
| 2026-9 | #23 info_validate_num | note | DEAD; if enabled, MIN/MAX path logs but lacks `return 1` |
| 2026-9 | #24 bytes | FIX | TB→bytes `*2014`→`*1024` (1tb was 1.9667× too big) |
| 2026-9 | #25 ui_countdown | FIX | `local SECONDS`→`CNT` (special-var rebase hazard) |
| 2026-9 | #34 media_device_partnum | FIX | dead is_number guard → rc-based `is_number … >/dev/null` |
| 2026-9 | #37 media_device_name | FIX | 10+ partition mangle (mmcblk0p10) → anchored trailing-digit strip |
| 2026-9 | #42 media_disk_serial | FIX | `-o vendor,model,serial` → `-o serial` (bare serial) |
| 2026-9 | #47 media_disk_first_available_byte | FIX | spurious fresh-disk `religned` debug line → `-n` guard |
| 2026-9 | #48 ismounted call-sites | FIX | two dead `= true` mount-guards (3524 repair / 4614 resize) → `-n $(…)` |
| 2026-9 | #53 media_filesystem_size | FIX | xfs restore inverted → `[[ -z $WAS_MOUNTED ]]`; `$currentsize`→`$blockcount` |
| 2026-9 | #58/#59/#60 whole-disk branches | FIX | start was disk-size → `1`; end/size were `1` → disk size; #61 0-partition path now returns disk end. lint −1 (SC2005), ref→final19 |
| 2026-9 | #61b/#63 test+verify loops | FIX | 1970 yesno gate un-inverted (YES=verify); mmcblk/nvme `p`-partition names in 1881/1972; #61b PARTCOUNT before prompt + "partitions" typo |
| 2026-9 | #66 addpartition | FIX | fat16+max now capped at BEG+4090M (was unbounded → oversized fat16); 2221/2262 verbose prints were dead (`(( $VERBOSE ))` always false) → `[[ $VERBOSE = -v ]]`; SC2004 −1, ref→final20 |
| 2026-9 | #67/#68 delpart+mount names | FIX | mmcblk/nvme p-names: delpartition 2285 (was failing checktarget on SD/NVMe disks) + mount loop 2356; #67 `$PART_NO`→`$PARTNUM`; #69 `exit`→`return`. SC2153 −1, ref→final21 |
| 2026-9 | #71/#72 erase+format | FIX | eraseDisk p-name on mmcblk/nvme (/dev/ form); Step 3 rc check (`\|\| return 1`, added 0 lint); wipe root-disk guard was DEAD (unset var vs function) → `$(env_root_disk)`. SC2154 −1, SC2053 −1, ref→final22 |
