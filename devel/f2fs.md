#F2FS Porting

## Kernel Part

kernel for f2fs 3.10 

Merge f2fs code from: http://git.kernel.org/cgit/linux/kernel/git/jaegeuk/f2fs-tools.git

    1.1 fs/f2fs
    1.2 include/fs/f2fs_fs.h
    1.3 include/trace/events/f2fs.h


## Build F2FS userdata image


```diff
diff --git a/BoardConfig.mk b/BoardConfig.mk
index 6f9f373..a82a4a3 100755
--- a/BoardConfig.mk
+++ b/BoardConfig.mk
@@ -10,8 +10,12 @@ include device/meizu/$(MTK_TARGET_PROJECT)/yunos/BoardConfig.mk
 #Config partition size
 include device/meizu/meizu6795_lwt_cu/partition_size.mk
 BOARD_CACHEIMAGE_FILE_SYSTEM_TYPE := ext4
+BOARD_USERDATAIMAGE_FILE_SYSTEM_TYPE := f2fs
 BOARD_FLASH_BLOCK_SIZE := 4096
 
+ifeq ($(HOST_OS),linux)
+TARGET_USERIMAGES_USE_F2FS := true
+endif
 
 #flyme edit by cerberspace@MultiMedia.Framworks for DIRAC HDSOUND
 BOARD_USE_AUDIO_DIRAC_HD := true
```

## Build tools

```
diff --git a/device.mk b/device.mk
index b6d2d93..43bf2d1 100755
--- a/device.mk
+++ b/device.mk
@@ -391,6 +391,10 @@ PRODUCT_PACKAGES += libfile_op
 PRODUCT_PACKAGES += nvram_agent_binder
 PRODUCT_PACKAGES += nvram_daemon
 PRODUCT_PACKAGES += make_ext4fs
+PRODUCT_PACKAGES += make_f2fs
+PRODUCT_PACKAGES += f2fs_sparseblock
+PRODUCT_PACKAGES += fsck.f2fs
+PRODUCT_PACKAGES += mkfs.f2fs
 PRODUCT_PACKAGES += sdcard
 PRODUCT_PACKAGES += libext
 PRODUCT_PACKAGES += libext4
```

## Set fstab


```
diff --git a/fstab.mt6795.ma01 b/fstab.mt6795.ma01
index cc33b2c..7ff3f83 100644
--- a/fstab.mt6795.ma01
+++ b/fstab.mt6795.ma01
@@ -5,7 +5,7 @@
 
 /dev/block/platform/mtk-msdc.0/by-name/system      /system             ext4      ro                                                          wait
 /dev/block/platform/mtk-msdc.0/by-name/custom      /custom             ext4      ro                                                          wait
-/dev/block/platform/mtk-msdc.0/by-name/userdata    /data               ext4      noatime,nosuid,nodev,noauto_da_alloc,discard                wait,check,resize,encryptable=footer
+/dev/block/platform/mtk-msdc.0/by-name/userdata    /data               f2fs      noatime,nosuid,nodev,discard,nodiratime                    wait,nonremovable
 /dev/block/platform/mtk-msdc.0/by-name/cache       /cache              ext4      noatime,nosuid,nodev,noauto_da_alloc,discard                wait,check
 /dev/block/platform/mtk-msdc.0/by-name/protect1    /protect_f          ext4      noatime,nosuid,nodev,noauto_da_alloc,commit=1,nodelalloc    wait,check,autoformat
 /dev/block/platform/mtk-msdc.0/by-name/protect2    /protect_s          ext4      noatime,nosuid,nodev,noauto_da_alloc,commit=1,nodelalloc    wait,check,autoformat

```

## Disable SELINUX

```
diff --git a/platform/mt6795/rules.mk b/platform/mt6795/rules.mk
index 5e709ec..af471be 100755
--- a/platform/mt6795/rules.mk
+++ b/platform/mt6795/rules.mk
@@ -11,7 +11,7 @@ CPU     := generic
 MMC_SLOT         := 1
 
 # choose one of following value -> 1: disabled/ 2: permissive /3: enforcing
-SELINUX_STATUS := 3
+SELINUX_STATUS := 1
```