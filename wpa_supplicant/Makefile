#------------------------------------------------------------------------------#
# All Right Reserved.
# Author       :
# Version      : V1.0.0  202x.01.01
# Description  :
# Note         : gaoyang3513@163.com        Modified        202x.01.01
#------------------------------------------------------------------------------#

#------------------------------------------------------------------------------#
# Path information
#------------------------------------------------------------------------------#
LOCAL_DIR    := $(realpath $(dir $(lastword $(MAKEFILE_LIST))))

# SDK directories
SDK_TOP       ?= $(HOME)/Source/15-SG200x/01-MilkDuo/02-Project/SDK_SG200x_V2.0.0
KERNEL_DIR    ?= $(SDK_TOP)/linux_5.10/build/sg2000_milkv_duos_glibc_arm64_sd
TOOLCHAIN_DIR ?= $(SDK_TOP)/host-tools/gcc/gcc-linaro-7.3.1-2018.05-x86_64_aarch64-linux-gnu/bin

# Subdirectory

OUTPUT_DIR   := $(LOCAL_DIR)/output
INSTALL_DIR  ?= $(LOCAL_DIR)/__install
#$(info Output directoty : $(OUTPUT_DIR))
#$(info Install directoty: $(INSTALL_DIR))

export KERNEL_DIR

#------------------------------------------------------------------------------#
# Variables
#------------------------------------------------------------------------------#
MULTI_CORES ?= $(shell grep -c ^processor /proc/cpuinfo)

SUB_DIRS := \
	04-Debug \
	03-Bus/01-Platform/01-osdev \

#------------------------------------------------------------------------------#
# Compile configure
#------------------------------------------------------------------------------#
#ARCH          ?= riscv
#CROSS_COMPILE ?= riscv64-unknown-linux-musl-
ARCH          ?= arm64
CROSS_COMPILE ?= aarch64-linux-gnu-

CC            := $(CROSS_COMPILE)gcc
LD            := $(CROSS_COMPILE)ld
AR            := $(CROSS_COMPILE)ar
STRIP         := $(CROSS_COMPILE)strip

PATH := $(TOOLCHAIN_DIR):$(PATH)

export ARCH CROSS_COMPILE PATH

$(warning $(PATH))
#------------------------------------------------------------------------------#
# Targets
#------------------------------------------------------------------------------#
.PHONY: init all clean distclean install menuconfig

all: init libnl libopenssl wpa

clean:
#       Directory
	@if [ -d $(LOCAL_DIR)/output        ]; then rm -rf $(LOCAL_DIR)/output;        fi;
	@if [ -d $(LOCAL_DIR)/__install     ]; then rm -rf $(LOCAL_DIR)/__install;     fi;

install:
	@mkdir -p $(INSTALL_DIR)
	@cp -arf $(OUTPUT_DIR)/. $(INSTALL_DIR)

init:
	@mkdir -p $(OUTPUT_DIR);

libopenssl_tar=openssl-3.1.6.tar.gz
libopenssl_dir=$(patsubst %.tar.gz,%,$(libopenssl_tar))

libopenssl: init
	if [ ! -d $(libopenssl_dir) ]; then \
		tar -xvf $(libopenssl_tar) > /dev/null 2>&1; \
		cd $(libopenssl_dir); \
		./config no-asm shared --prefix=$(OUTPUT_DIR) --cross-compile-prefix=$(CROSS_COMPILE); \
		sed -i 's/-m64//g' Makefile; \
		$(MAKE) CC=$(CC) -j$(MULTI_CORES); \
	fi;
	$(MAKE) -C $(libopenssl_dir) install  -j$(MULTI_CORES);

libopenssl_clean:
	rm -rf $(libopenssl_dir)


libnl_tar=libnl-3.9.0.tar.gz
libnl_dir=$(patsubst %.tar.gz,%,$(libnl_tar))

libnl: init
	if [ ! -d $(libnl_dir) ]; then \
		tar -xvf $(libnl_tar) > /dev/null 2>&1; \
		cd $(libnl_dir); \
		./configure --prefix=$(OUTPUT_DIR) --host=$(CROSS_COMPILE:%-=%); \
		$(MAKE) -j$(MULTI_CORES); \
	fi;
	$(MAKE) -C $(libnl_dir) install -j$(MULTI_CORES);

libnl_clean:
	rm -rf $(libnl_dir);

wpa_tar=wpa_supplicant-2.10.tar.gz
wpa_dir=$(patsubst %.tar.gz,%,$(wpa_tar))

wpa: init
	if [ ! -d $(wpa_dir) ]; then \
		tar -xvf $(wpa_tar) > /dev/null 2>&1; \
		cd $(wpa_dir)/wpa_supplicant; \
		cp -arf defconfig .config; \
		sed -i 's/^CONFIG_CTRL_IFACE_DBUS_NEW=y/#CONFIG_CTRL_IFACE_DBUS_NEW=y/g'      .config; \
		sed -i 's/^CONFIG_CTRL_IFACE_DBUS_INTRO=y/#CONFIG_CTRL_IFACE_DBUS_INTRO=y/g'  .config; \
		sed -i '$$a CFLAGS += -I$(OUTPUT_DIR)/include -I$(OUTPUT_DIR)/include/libnl3' .config; \
		sed -i '$$a LIBS   += -L$(OUTPUT_DIR)/lib -L$(OUTPUT_DIR)/lib64'              .config; \
		$(MAKE) -C wpa_supplicant CC=$(CC) -j$(MULTI_CORES); \
	fi;
	cp -arf $(wpa_dir)/wpa_supplicant/wpa_supplicant $(OUTPUT_DIR)/bin;
	cp -arf $(wpa_dir)/wpa_supplicant/wpa_cli        $(OUTPUT_DIR)/bin;

wpa_clean:
	rm -rf $(wpa_dir);
