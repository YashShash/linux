# SPDX-License-Identifier: GPL-2.0
#
# linux/arch/s390/boot/compressed/Makefile
#
# create a compressed vmlinux image from the original vmlinux
#

KCOV_INSTRUMENT := n

targets	:= vmlinux.lds vmlinux vmlinux.bin vmlinux.bin.gz vmlinux.bin.bz2
targets += vmlinux.bin.xz vmlinux.bin.lzma vmlinux.bin.lzo vmlinux.bin.lz4
targets += misc.o piggy.o sizes.h head.o

KBUILD_CFLAGS := -m64 -D__KERNEL__ -O2
KBUILD_CFLAGS += -DDISABLE_BRANCH_PROFILING -D__NO_FORTIFY
KBUILD_CFLAGS += $(cflags-y) -fno-delete-null-pointer-checks -msoft-float
KBUILD_CFLAGS += -fno-asynchronous-unwind-tables
KBUILD_CFLAGS += $(call cc-option,-mpacked-stack)
KBUILD_CFLAGS += $(call cc-option,-ffreestanding)

GCOV_PROFILE := n
UBSAN_SANITIZE := n

OBJECTS := $(addprefix $(objtree)/arch/s390/kernel/, head.o ebcdic.o als.o)
OBJECTS += $(objtree)/drivers/s390/char/sclp_early_core.o
OBJECTS += $(obj)/head.o $(obj)/misc.o $(obj)/piggy.o

LDFLAGS_vmlinux := --oformat $(LD_BFD) -e startup -T
$(obj)/vmlinux: $(obj)/vmlinux.lds $(OBJECTS)
	$(call if_changed,ld)

TRIM_HEAD_SIZE := 0x11000

sed-sizes := -e 's/^\([0-9a-fA-F]*\) . \(__bss_start\|_end\)$$/\#define SZ\2 (0x\1 - $(TRIM_HEAD_SIZE))/p'

quiet_cmd_sizes = GEN     $@
      cmd_sizes = $(NM) $< | sed -n $(sed-sizes) > $@

quiet_cmd_trim_head = TRIM    $@
      cmd_trim_head = tail -c +$$(($(TRIM_HEAD_SIZE) + 1)) $< > $@

$(obj)/sizes.h: vmlinux
	$(call if_changed,sizes)

AFLAGS_head.o += -I$(objtree)/$(obj)
$(obj)/head.o: $(obj)/sizes.h

CFLAGS_misc.o += -I$(objtree)/$(obj)
$(obj)/misc.o: $(obj)/sizes.h

OBJCOPYFLAGS_vmlinux.bin.full :=  -R .comment -S
$(obj)/vmlinux.bin.full: vmlinux
	$(call if_changed,objcopy)

$(obj)/vmlinux.bin: $(obj)/vmlinux.bin.full
	$(call if_changed,trim_head)

vmlinux.bin.all-y := $(obj)/vmlinux.bin

suffix-$(CONFIG_KERNEL_GZIP)  := gz
suffix-$(CONFIG_KERNEL_BZIP2) := bz2
suffix-$(CONFIG_KERNEL_LZ4)  := lz4
suffix-$(CONFIG_KERNEL_LZMA)  := lzma
suffix-$(CONFIG_KERNEL_LZO)  := lzo
suffix-$(CONFIG_KERNEL_XZ)  := xz

$(obj)/vmlinux.bin.gz: $(vmlinux.bin.all-y)
	$(call if_changed,gzip)
$(obj)/vmlinux.bin.bz2: $(vmlinux.bin.all-y)
	$(call if_changed,bzip2)
$(obj)/vmlinux.bin.lz4: $(vmlinux.bin.all-y)
	$(call if_changed,lz4)
$(obj)/vmlinux.bin.lzma: $(vmlinux.bin.all-y)
	$(call if_changed,lzma)
$(obj)/vmlinux.bin.lzo: $(vmlinux.bin.all-y)
	$(call if_changed,lzo)
$(obj)/vmlinux.bin.xz: $(vmlinux.bin.all-y)
	$(call if_changed,xzkern)

LDFLAGS_piggy.o := -r --format binary --oformat $(LD_BFD) -T
$(obj)/piggy.o: $(obj)/vmlinux.scr $(obj)/vmlinux.bin.$(suffix-y)
	$(call if_changed,ld)
