SHELL = /bin/bash

# The top-level makefile defines required variables and flags.
ifneq ($(shell [[ $(MAKELEVEL) -ge 1 ]] && echo 1),1)
  $(error Please use the top-level Makefile to build this driver)
endif

ifneq ($(wildcard /etc/lsb-release),)
  ifneq ($(shell $(grep) "Ubuntu" /etc/lsb-release),)
    FLAGS += -DUBUNTU_VERSION_CODE
  endif
endif

include $(srcdir)/make/common_flags.mk

$(info srcdir = $(srcdir).)
$(info KSRC = $(KSRC).)
$(info VF = $(VF).)

MOD_NAME := qdma$(PFVF_TYPE)

EXTRA_CFLAGS += -DLINUX -D__KERNEL__ -DMODULE -O2 -pipe -Wall -Werror
EXTRA_CFLAGS += $(FLAGS) $(CPPFLAGS) $(EXTRA_FLAGS)
EXTRA_CFLAGS += -I$(srcdir)/include
EXTRA_CFLAGS += -I$(KSRC)/include
EXTRA_CFLAGS += -I.

# linux >= 3.13 genl_ops become part of the genl_family. And although
# genl_register_family_with_ops() is still retained until kernel 4.10,
# its prototype changed from a extern function to a define.
#
ifneq ($(shell $(grep) -c 'int genl_register_family_with_ops' $(KINC)/net/genetlink.h),0) 
  ccflags-y += -D__GENL_REG_FAMILY_OPS_FUNC__
endif

# linux <= 3.13 pci_msix_vec_count is not defined.
#
ifeq ($(shell $(grep) -c 'int pci_msix_vec_count' $(KINC)/linux/pci.h),0) 
  ccflags-y += -D__PCI_MSI_VEC_COUNT__
endif

# linux < 3.13 list_next_entry is not defined.
#
ifeq ($(shell $(grep) -c 'list_next_entry' $(KINC)/linux/list.h),0) 
  ccflags-y += -D__LIST_NEXT_ENTRY__
endif

# linux < 3.18.13 READ_ONCE is not defined.
#
ifneq ($(shell $(grep) -c 'READ_ONCE' $(KINC)/linux/compiler.h),0) 
  ccflags-y += -D__READ_ONCE_DEFINED__
endif

$(info EXTRA_FLAGS = $(EXTRA_FLAGS).)
$(info ccflags-y = $(ccflags-y).)
#EXTRA_CFLAGS += -DDEBUG

ifneq ($(modulesymfile),)
  override symverfile = symverfile="$(topdir)/$(modulesymfile) \
					-o $(drvdir)/$(modulesymfile)"
else
  override symverfile =
endif

ifneq ($(SUBDIRS),)
  BASE_OBJS := $(patsubst $(SUBDIRS)/%.c,%.o,$(wildcard $(SUBDIRS)/*.c $(SUBDIRS)/*/*.c))
endif
obj-m := $(MOD_NAME).o
$(MOD_NAME)-objs := $(BASE_OBJS)

.PHONY: build
build:
	@mkdir -p -m 755 $(build_dir)
	@rm -f $(MOD_NAME).ko $(build_dir)/$(MOD_NAME).ko
	@$(MAKE) symverfile=$(symverfile) KBUILD_EXTRA_SYMBOLS=$(extra_symb) -C $(KOBJ) SUBDIRS=$(shell pwd) modules
	@cp $(MOD_NAME).ko $(build_dir)

.PHONY: clean
clean:
	@-/bin/rm -rf *.ko* ?odule* .tmp_versions *.mod.* *.o *.o.* .*.o.* .*.cmd
	@-/bin/rm -rf */*.o */*.o.* */.*.o.* */.*.cmd
	@-/bin/rm -rf $(build_dir)/*.ko
	@-/bin/rm -f $(srcdir)/drv/libqdma
