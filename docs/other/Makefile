# Copyright (c) Quectel Wireless Solution, Co., Ltd.All Rights Reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

#env
TOP := ../../..
ROOT := $(TOP)/../..

KCONFIG_CONFIG_DIR := ../../../../..
include $(KCONFIG_CONFIG_DIR)/config/$(KCONFIG_CONFIG)

ifeq ("$(origin V)", "command line")
BUILD_VERBOSE=$(V)
endif
ifndef BUILD_VERBOSE
BUILD_VERBOSE = 0
endif
# ifeq ($(BUILD_VERBOSE),0)
# Q = @
# else
# Q =
# endif

BUILD ?= build
FROZEN_MPY_DIR ?= .

RM = rm
ECHO = echo
CP = cp
MKDIR = mkdir
SED = sed
CAT = cat
TOUCH = touch

MAKE_MANIFEST = $(PYTHON) $(TOP)/tools/makemanifest.py
MAKE_FROZEN = $(PYTHON) $(TOP)/tools/make-frozen.py
#because E-safeNet, mpy-cross could not read the py file, rename as notepad++
ifeq ($(strip $(HOST)),linux)
MPY_CROSS = $(TOP)/ports/quectel/tools/mpy-cross
else
MPY_CROSS = $(TOP)/ports/quectel/tools/notepad++.exe
endif
MPY_TOOL = $(PYTHON) $(TOP)/tools/mpy-tool.py

QSTR_DEFS = source/qstrdefsport.h
#py
PY_SRC ?= $(TOP)/py
HEADER_BUILD = $(TOP)/genhdr
PY_QSTR_DEFS = $(PY_SRC)/qstrdefs.h
ifneq ($(QSTR_AUTOGEN_DISABLE),1)
QSTR_DEFS_COLLECTED = $(HEADER_BUILD)/qstrdefs.collected.h
endif
QSTR_GLOBAL_DEPENDENCIES += $(PY_SRC)/mpconfig.h source/mpconfigport.h
CSUPEROPT = -O3
ifeq ($(strip $(PLAT)),ECR6600)
MPY_CROSS_FLAGS += -march=xtensa
else
MPY_CROSS_FLAGS += -march=armv7m
endif
# Enable building 32-bit code on 64-bit host.
ifeq ($(MICROPY_FORCE_32BIT),1)
CC += -m32
CXX += -m32
LD += -m32
endif

# External modules written in C.
ifneq ($(USER_C_MODULES),)
# pre-define USERMOD variables as expanded so that variables are immediate
# expanded as they're added to them
SRC_USERMOD :=
C_FLAGS_USERMOD :=
LD_FLAGS_USERMOD :=
$(foreach module, $(wildcard $(USER_C_MODULES)/*/micropython.mk), \
    $(eval USERMOD_DIR = $(patsubst %/,%,$(dir $(module))))\
    $(info Including User C Module from $(USERMOD_DIR))\
	$(eval include $(module))\
)

SRC_MOD += $(patsubst $(USER_C_MODULES)/%.c,%.c,$(SRC_USERMOD))
C_FLAGS_MOD += $(C_FLAGS_USERMOD)
LD_FLAGS_MOD += $(LD_FLAGS_USERMOD)
endif

# py object files
PY_CORE_O_BASENAME = $(addprefix py/,\
	mpstate.o \
	nlr.o \
	nlrx86.o \
	nlrx64.o \
	nlrthumb.o \
	nlrpowerpc.o \
	nlrxtensa.o \
	nlrsetjmp.o \
	malloc.o \
	gc.o \
	pystack.o \
	qstr.o \
	vstr.o \
	mpprint.o \
	unicode.o \
	mpz.o \
	reader.o \
	lexer.o \
	parse.o \
	scope.o \
	compile.o \
	emitcommon.o \
	emitbc.o \
	asmbase.o \
	asmx64.o \
	emitnx64.o \
	asmx86.o \
	emitnx86.o \
	asmthumb.o \
	emitnthumb.o \
	emitinlinethumb.o \
	asmarm.o \
	emitnarm.o \
	asmxtensa.o \
	emitnxtensa.o \
	emitinlinextensa.o \
	emitnxtensawin.o \
	formatfloat.o \
	parsenumbase.o \
	parsenum.o \
	emitglue.o \
	persistentcode.o \
	runtime.o \
	runtime_utils.o \
	scheduler.o \
	nativeglue.o \
	pairheap.o \
	ringbuf.o \
	stackctrl.o \
	argcheck.o \
	warning.o \
	profile.o \
	map.o \
	obj.o \
	objarray.o \
	objattrtuple.o \
	objbool.o \
	objboundmeth.o \
	objcell.o \
	objclosure.o \
	objcomplex.o \
	objdeque.o \
	objdict.o \
	objenumerate.o \
	objexcept.o \
	objfilter.o \
	objfloat.o \
	objfun.o \
	objgenerator.o \
	objgetitemiter.o \
	objint.o \
	objint_longlong.o \
	objint_mpz.o \
	objlist.o \
	objmap.o \
	objmodule.o \
	objobject.o \
	objpolyiter.o \
	objproperty.o \
	objnone.o \
	objnamedtuple.o \
	objrange.o \
	objreversed.o \
	objset.o \
	objsingleton.o \
	objslice.o \
	objstr.o \
	objstrunicode.o \
	objstringio.o \
	objtuple.o \
	objtype.o \
	objzip.o \
	opmethods.o \
	sequence.o \
	stream.o \
	binary.o \
	builtinimport.o \
	builtinevex.o \
	builtinhelp.o \
	modarray.o \
	modbuiltins.o \
	modcollections.o \
	modgc.o \
	modio.o \
	modmath.o \
	modcmath.o \
	modmicropython.o \
	modstruct.o \
	modsys.o \
	moduerrno.o \
	modthread.o \
	vm.o \
	bc.o \
	showbc.o \
	repl.o \
	smallint.o \
	frozenmod.o \
	)

PY_EXTMOD_O_BASENAME = \
	extmod/moductypes.o \
	extmod/modujson.o \
	extmod/modure.o \
	extmod/moduzlib.o \
	extmod/moduheapq.o \
	extmod/modutimeq.o \
	extmod/modubinascii.o \
	extmod/virtpin.o \
	extmod/machine_mem.o \
	extmod/machine_pinbase.o \
	extmod/machine_signal.o \
	extmod/machine_pulse.o \
	extmod/machine_i2c.o \
	extmod/machine_spi.o \
	extmod/modbluetooth.o \
	extmod/modurandom.o \
	extmod/moduselect.o \
	extmod/moduasyncio.o \
	extmod/moduwebsocket.o \
	extmod/modwebrepl.o \
	extmod/modframebuf.o \
	extmod/vfs.o \
	extmod/vfs_blockdev.o \
	extmod/vfs_reader.o \
	extmod/vfs_posix.o \
	extmod/vfs_posix_file.o \
	extmod/vfs_fat.o \
	extmod/vfs_fat_file.o \
	extmod/vfs_lfs.o \
	extmod/utime_mphal.o \
	extmod/uos_dupterm.o \
	extmod/modussl_mbedtls.o \
	extmod/moduhashlib.o \
	extmod/moducryptolib.o \
	lib/embed/abort_.o \
	lib/utils/printf.o \
	lib/utils/sys_stdio_mphal.o

ifeq ($(strip $(PLAT)),Qualcomm)
PY_EXTMOD_O_BASENAME += \
	extmod/vfs_efs.o \
	extmod/vfs_efs_file.o
endif

ifeq ($(strip $(PLAT)), $(filter $(PLAT),Unisoc Unisoc_8910_R05 Unisoc_8850_R02))
PY_EXTMOD_O_BASENAME += \
	extmod/vfs_temp.o \
	extmod/vfs_temp_file.o
endif

# Sources that may contain qstrings
SRC_QSTR_IGNORE = py/nlr%
SRC_QSTR += $(SRC_MOD) $(filter-out $(SRC_QSTR_IGNORE),$(PY_CORE_O_BASENAME:.o=.c)) $(PY_EXTMOD_O_BASENAME:.o=.c)
SRC_QSTR += \
			source/modalipay.c \
			source/modalipay_collection.c \
			source/modalipay_se.c \
			source/audio_audio.c \
			source/audio_handle.c \
			source/audio_queue.c \
			source/audio_record.c \
			source/audio_tts.c \
			source/audiocodec_rt5616.c \
			source/callbackdeal.c \
			source/camera_capture.c \
			source/camera_preview.c \
			source/camera_scandecode.c \
			source/devreport.c \
			source/machine_extint.c \
			source/machine_hw_spi.c \
			source/machine_iic.c \
			source/machine_iic_simulation.c \
			source/machine_keypad.c \
			source/machine_lcd.c \
			source/machine_nandflash.c \
			source/machine_pin.c \
			source/machine_rtc.c \
			source/machine_timer.c \
			source/machine_uart.c \
			source/machine_wdt.c \
			source/misc_adc.c \
			source/misc_power.c \
			source/misc_powerkey.c \
			source/misc_pwm.c \
			source/misc_temperature.c \
			source/misc_usb.c \
			source/misc_usbnet.c \
			source/modatcmd.c \
			source/modaudio.c \
			source/modaudiocodec.c \
			source/modble.c \
			source/modbt.c \
			source/modcamera.c \
			source/modcelllocator.c \
			source/modcelllocator_qpy.c \
			source/locator.c \
			source/modcsd.c \
			source/moddatacall.c \
			source/moddev.c \
			source/modethernet.c \
			source/modethernet_dm9051.c \
			source/modethernet_w5500.c \
			source/modethernet_ch395.c \
			source/modexample.c \
			source/modfota.c \
			source/modgnss.c \
			source/modlpm.c \
			source/modlvgl.c \
			source/modlwm2m.c \
			source/modmachine.c \
			source/modmisc.c \
			source/modnb.c \
			source/modnet.c \
			source/modostimer.c \
			source/modpoc.c \
			source/modqrcode.c \
			source/modrtmp.c \
			source/modsecuredata.c \
			source/modsensor.c \
			source/modsha1.c \
			source/modsim.c \
			source/modslipif.c \
			source/modsms.c \
			source/modussd.c \
			source/modsocket.c \
			source/modwifi.c \
			source/modtp.c \
			source/moduos.c \
			source/modutils.c \
			source/modutime.c \
			source/modvoicecall.c \
			source/modwifilocator.c \
			source/modwifiscan.c \
			source/nb_aep.c \
			source/nb_oc.c \
			source/nb_onenet.c \
			source/quecpython.c \
			source/sensor_sn95500.c \
			source/sensor_icckey.c \
			source/sensor_gnss.c \
			source/tp_gt9xx.c \
			source/tp_xpt2046.c \
			source/tp_cst816.c \
			source/tp_ft6x36.c \
			source/tp_st1615.c \
			source/modrsa.c \
			source/utils_crc32.c \
			source/http_parser.c \
			source/modwebserver.c \
			source/modethernet_yt8512h.c \
			source/sim_esim.c \
			source/modG711.c \
			source/modG729.c \
			source/modcan_mcp2515.c \
			source/modextuart_wk2114.c \
			source/modgpio_encoder.c \
			source/modextgpio_aw9523.c \
			source/machine_key.c \
			source/lpm_wakeup.c \
			source/sensor_handmic_key.c	\
			source/modnetwork.c	\
			source/modwlan_ASR5803.c \
			source/machine_soft_spi.c \
			source/modwlan.c \
			source/machine_one_wire.c \
			source/modsif.c\
			source/jrtc.c \
			source/modnfc.c \
			source/nfc_sl6320.c \
			source/modstk.c  \
			source/modvoip.c



ifeq ($(strip $(PLAT)),RDA)
SRC_QSTR += $(ROOT)/components/quecsdk/cloud/python/modquecIot-BC25old.c
else
SRC_QSTR += $(ROOT)/components/quecsdk/cloud/python/modquecIot.c
endif

ifeq ($(CONFIG_USQLITE), y)
SRC_QSTR += lib/usqlite/usqlite_connection.c \
			lib/usqlite/usqlite_cursor.c \
			lib/usqlite/usqlite_row.c \
			lib/usqlite/usqlite_file.c \
			lib/usqlite/usqlite_mem.c \
			lib/usqlite/usqlite_vfs.c \
			lib/usqlite/usqlite_utils.c \
			lib/usqlite/usqlite.c \
			lib/usqlite/usqlite_module.c

INC += -Ilib/usqlite
endif

$(BUILD)/%.pp: %.c
	@$(ECHO) "PreProcess $<"
	$(Q)$(CPP) $(C_FLAGS) -Wp,-C,-dD,-dI -o $@ $<

# Anything that depends on FORCE will be considered out-of-date
FORCE:
.PHONY: FORCE

$(HEADER_BUILD):
	$(MKDIR) -p $@

#$(HEADER_BUILD)/mpversion.h: FORCE | $(HEADER_BUILD)
#	$(Q)$(PYTHON) $(PY_SRC)/makeversionhdr.py $@
# mpconfigport.mk is optional, but changes to it may drastically change
# overall config, so they need to be caught
MPCONFIGPORT_MK = $(wildcard mpconfigport.mk)

# qstr data
# Adding an order only dependency on $(HEADER_BUILD) causes $(HEADER_BUILD) to get
# created before we run the script to generate the .h
# Note: we need to protect the qstr names from the preprocessor, so we wrap
# the lines in "" and then unwrap after the preprocessor is finished.
# See more information about this process in docs/develop/qstr.rst.
$(HEADER_BUILD)/qstrdefs.generated.h: $(PY_QSTR_DEFS) $(QSTR_DEFS) $(QSTR_DEFS_COLLECTED) $(PY_SRC)/makeqstrdata.py source/mpconfigport.h $(MPCONFIGPORT_MK) $(PY_SRC)/mpconfig.h | $(HEADER_BUILD)
	@$(ECHO) "GEN $@"
	$(Q)$(CAT) $(PY_QSTR_DEFS) $(QSTR_DEFS) $(QSTR_DEFS_COLLECTED) | $(SED) 's/^Q(.*)/"&"/' | $(CPP) $(C_FLAGS) - | $(SED) 's/^\"\(Q(.*)\)\"/\1/' > $(HEADER_BUILD)/qstrdefs.preprocessed.h
	$(Q)$(PYTHON) $(PY_SRC)/makeqstrdata.py $(HEADER_BUILD)/qstrdefs.preprocessed.h > $@

# Compressed error strings.
$(HEADER_BUILD)/compressed.split: $(HEADER_BUILD)/qstr.i.last
	@$(ECHO) "GEN $@"
	$(Q)$(PYTHON) $(PY_SRC)/makeqstrdefs.py split compress $< $(HEADER_BUILD)/compress _
	$(Q)$(TOUCH) $@
	
$(HEADER_BUILD)/compressed.collected: $(HEADER_BUILD)/compressed.split
	@$(ECHO) "GEN $@"
	$(Q)$(PYTHON) $(PY_SRC)/makeqstrdefs.py $(CAT) compress _ $(HEADER_BUILD)/compress $@
	
$(HEADER_BUILD)/compressed.data.h: $(HEADER_BUILD)/compressed.collected
	@$(ECHO) "GEN $@"
	$(Q)$(PYTHON) $(PY_SRC)/makecompresseddata.py $< > $@
	
# build a list of registered modules for py/objmodule.c.
$(HEADER_BUILD)/moduledefs.h: $(SRC_QSTR) $(QSTR_GLOBAL_DEPENDENCIES) | $(HEADER_BUILD)
	@$(ECHO) "GEN $@"
	$(Q)$(PYTHON) $(PY_SRC)/makemoduledefs.py --vpath="., $(TOP), $(USER_C_MODULES)" $(SRC_QSTR) > $@

SRC_QSTR += $(HEADER_BUILD)/moduledefs.h

INC += -I$(ROOT)/system/platform/$(strip $(PLAT))/boards/$(strip $(BOARD))/config
INC += -I$(ROOT)/system/platform/$(strip $(PLAT))/include

ifeq ($(strip $(PLAT)),Qualcomm)
INC += -I$(COMPILER_PATH)/armv7m-none-eabi/libc/include
INC += -I$(ROOT)/system/platform/$(strip $(PLAT))/boards/$(strip $(BOARD))/include/azure_api
INC += -I$(ROOT)/system/platform/$(strip $(PLAT))/boards/$(strip $(BOARD))/include/azure_api/c-utility
INC += -I$(ROOT)/system/platform/$(strip $(PLAT))/boards/$(strip $(BOARD))/include/azure_api/c-utility/inc
INC += -I$(ROOT)/system/platform/$(strip $(PLAT))/boards/$(strip $(BOARD))/include/azure_api/c-utility/inc/azure_c_shared_utility
INC += -I$(ROOT)/system/platform/$(strip $(PLAT))/boards/$(strip $(BOARD))/include/azure_api/c-utility/pal
INC += -I$(ROOT)/system/platform/$(strip $(PLAT))/boards/$(strip $(BOARD))/include/azure_api/c-utility/pal/generic
INC += -I$(ROOT)/system/platform/$(strip $(PLAT))/boards/$(strip $(BOARD))/include/azure_api/iothub_client
INC += -I$(ROOT)/system/platform/$(strip $(PLAT))/boards/$(strip $(BOARD))/include/azure_api/iothub_client/inc
INC += -I$(ROOT)/system/platform/$(strip $(PLAT))/boards/$(strip $(BOARD))/include/azure_api/iothub_client/inc/internal
INC += -I$(ROOT)/system/platform/$(strip $(PLAT))/boards/$(strip $(BOARD))/include/azure_api/serializer
INC += -I$(ROOT)/system/platform/$(strip $(PLAT))/boards/$(strip $(BOARD))/include/azure_api/serializer/inc
INC += -I$(ROOT)/system/platform/$(strip $(PLAT))/boards/$(strip $(BOARD))/include/azure_api/umqtt
INC += -I$(ROOT)/system/platform/$(strip $(PLAT))/boards/$(strip $(BOARD))/include/azure_api/umqtt/inc
INC += -I$(ROOT)/system/platform/$(strip $(PLAT))/boards/$(strip $(BOARD))/include/azure_api/umqtt/inc/azure_umqtt_c
INC += -I$(ROOT)/system/platform/$(strip $(PLAT))/boards/$(strip $(BOARD))/include/qapi
INC += -I$(ROOT)/system/platform/$(strip $(PLAT))/boards/$(strip $(BOARD))/include/threadx_api
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/include
INC += -I$(ROOT)/system/at
INC += -I$(ROOT)/system/esim
endif

INC += -I.
INC += -I$(HEADER_BUILD)
INC += -I$(TOP)
INC += -I$(TOP)/py
INC += -I$(TOP)/extmod
INC += -I$(TOP)/lib/utils
INC += -I$(TOP)/lib/mp-readline
INC += -I$(TOP)/lib/littlefs
INC += -I$(TOP)/lib/netutils
INC += -I$(TOP)/lib/timeutils
INC += -I$(TOP)/ports/quectel/core
INC += -I$(TOP)/ports/quectel/core/source
INC += -I$(TOP)/ports/quectel/core/py
INC += -I$(ROOT)/peripheral
INC += -I$(ROOT)/system/include
INC += -I$(ROOT)/system/platform/$(strip $(PLAT))/include
INC += -I$(ROOT)/system/platform/$(strip $(PLAT))/boards/$(strip $(BOARD))/include
INC += -I$(ROOT)/system/debug
INC += -I$(ROOT)/system/dev
INC += -I$(ROOT)/system/fs
INC += -I$(ROOT)/system/hal
INC += -I$(ROOT)/system/gnss
INC += -I$(ROOT)/system/network
INC += -I$(ROOT)/system/at
INC += -I$(ROOT)/system/aliiot

ifeq ($(strip $(PLAT)),ASR_1803s)
INC += -I$(ROOT)/system/csd
INC += -I$(ROOT)/system/esim
endif
ifeq ($(strip $(PLAT)),ASR_1606)
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/include
endif
ifeq ($(strip $(PLAT)),ASR_1602)
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/include
endif
ifeq ($(strip $(PLAT)),ASR_1609)
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/include
endif

ifeq ($(strip $(PLAT)),Unisoc)
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/include
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include/lwip
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include/lwip/apps
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include/lwip/priv
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include/lwip/prot
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include/netif
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include/posix
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include/posix/arpa
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include/posix/net
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include/posix/netinet
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include/posix/sys
INC += -I$(ROOT)/system/at
INC += -I$(ROOT)/system/esim
endif

ifeq ($(strip $(PLAT)),Unisoc_8910_R05)
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/include
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include/lwip
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include/lwip/apps
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include/lwip/priv
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include/lwip/prot
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include/netif
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include/netif/ppp
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include/netif/ppp/polarssl
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include/compat/posix
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include/compat/posix/arpa
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include/compat/posix/net
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include/compat/posix/netinet
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include/compat/posix/sys
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include/compat/stdc
endif

ifeq ($(strip $(PLAT)),$(filter $(PLAT),EIGEN EIGEN_718))
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/include
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/include/lwip
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/include/lwip/apps
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/include/lwip/priv
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/include/lwip/prot
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/include/arch
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/include/netif
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/include/posix
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/include/posix/sys
endif

ifeq ($(strip $(PLAT)),$(filter $(PLAT),Unisoc_8850 Unisoc_8850_R02))
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/include
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include/lwip
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include/lwip/apps
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include/lwip/priv
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include/lwip/prot
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include/netif
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include/netif/ppp
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include/netif/ppp/polarssl
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include/compat/posix
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include/compat/posix/arpa
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include/compat/posix/net
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include/compat/posix/netinet
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include/compat/posix/sys
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include/compat/stdc
endif

ifeq ($(strip $(PLAT)),RDA)
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/include
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/include/arch
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include/lwip
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include/lwip/apps
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include/lwip/priv
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include/lwip/prot
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include/netif
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include/posix
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include/posix/arpa
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include/posix/net
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include/posix/netinet
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/src/include/posix/sys
endif

ifeq ($(strip $(PLAT)), $(filter $(PLAT),ASR ASR_1803s))
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/include
INC += -I$(ROOT)/system/at
endif

ifeq ($(strip $(PLAT)),ECR6600)
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/include
endif

ifeq ($(strip $(PLAT)),aic8800m40)
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/lwip-STABLE-2_0_2_RELEASE_VER/ports/rtos/include
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/lwip-STABLE-2_0_2_RELEASE_VER/ports/rtos/include/arch
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/lwip-STABLE-2_0_2_RELEASE_VER/src/include
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/lwip-STABLE-2_0_2_RELEASE_VER/src/include/lwip
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/lwip-STABLE-2_0_2_RELEASE_VER/src/include/lwip/priv
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/lwip-STABLE-2_0_2_RELEASE_VER/src/include/lwip/prot
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/lwip-STABLE-2_0_2_RELEASE_VER/src/include/netif
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/lwip-STABLE-2_0_2_RELEASE_VER/src/include/posix
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/lwip-STABLE-2_0_2_RELEASE_VER/src/include/posix/sys
INC += -I$(ROOT)/system/lwip/$(strip $(PLAT))/net_al
endif

ifeq (y, $(CONFIG_MBEDTLS))
INC += -I$(ROOT)/system/mbedtls
INC += -I$(ROOT)/system/mbedtls/include
INC += -I$(ROOT)/system/mbedtls/include/mbedtls
INC += -I$(ROOT)/system/mbedtls/library
INC += -I$(ROOT)/system/mbedtls/port/helios/inc
endif
INC += -I$(ROOT)/system/network
INC += -I$(ROOT)/system/os
INC += -I$(ROOT)/system/startup
INC += -I$(ROOT)/system/fota
INC += -I$(ROOT)/system/bt
INC += -I$(ROOT)/system/at
ifeq (y, $(CONFIG_LVGL))
INC += -I$(TOP)/lib/lvgl
INC += -I$(TOP)/lib/lvgl/src
INC += -I$(TOP)/lib/lvgl/src/core
INC += -I$(TOP)/lib/lvgl/src/draw
INC += -I$(TOP)/lib/lvgl/src/extra
INC += -I$(TOP)/lib/lvgl/src/font
INC += -I$(TOP)/lib/lvgl/src/gpu
INC += -I$(TOP)/lib/lvgl/src/hal
INC += -I$(TOP)/lib/lvgl/src/misc
INC += -I$(TOP)/lib/lvgl/src/widgts
endif
INC += -I$(ROOT)/driver
INC += -I$(ROOT)/driver/lcd
ifeq (y, $(CONFIG_QRCODE))
INC += -I$(ROOT)/components/qrcode
endif
ifeq ($(CONFIG_JRTC), y)
INC += -I$(ROOT)/components/jrtc/inc
endif
ifeq (y, $(CONFIG_QUECTHING))
INC += -I$(ROOT)/components/quecsdk/cloud
INC += -I$(ROOT)/components/quecsdk/driverLayer
INC += -I$(ROOT)/components/quecsdk/thirdLib/mqtt
INC += -I$(ROOT)/components/quecsdk/thirdLib/cJSON
INC += -I$(ROOT)/components/quecsdk/kernel
endif
ifeq ($(CONFIG_SPINAND), y)
INC += -I$(ROOT)/components/fs/include
INC += -I$(ROOT)/components/fs/yaffs/src/direct
INC += -I$(ROOT)/components/fs/yaffs/src/port
INC += -I$(ROOT)/components/fs/yaffs/src
INC += -I$(ROOT)/components/fs/yaffs
endif
ifeq ($(CONFIG_RTMP), y)
INC += -I$(ROOT)/components/rtmpdump/librtmp/inc
INC += -I$(ROOT)/components/rtmpdump/libz/inc
endif
INC += -I$(ROOT)/utilities
ifeq ($(CONFIG_JPEG), y)
INC += -I$(ROOT)/components/jpeg/jpeg-9c
INC += -I$(ROOT)/components/jpeg
endif
ifeq ($(CONFIG_POC_BND), y)
INC += -I$(ROOT)/components/poc
INC += -I$(ROOT)/components/poc/bnd/inc
endif
ifeq ($(CONFIG_POC_BND_XIN), y)
INC += -I$(ROOT)/components/poc
INC += -I$(ROOT)/components/poc/bnd_xin/inc
endif
ifeq ($(CONFIG_POC_SL), y)
INC += -I$(ROOT)/components/poc
INC += -I$(ROOT)/components/poc/sl/inc
endif
ifeq ($(CONFIG_POC_QS), y)
INC += -I$(ROOT)/components/poc
INC += -I$(ROOT)/components/poc/qs/inc
endif
#Carlos.Meng add 2024/02/22
ifeq ($(CONFIG_POC_QS_R07), y)
INC += -I$(ROOT)/components/poc
INC += -I$(ROOT)/components/poc/qs/inc
endif
#Stephen.Gao add 2022/11/23
ifeq ($(CONFIG_POC_ZZD), y)
INC += -I$(ROOT)/components/poc
INC += -I$(ROOT)/components/poc/zzd/inc
endif
#Stephen.Gao add 2023/01/06
ifeq ($(CONFIG_POC_YDWL), y)
INC += -I$(ROOT)/components/poc
INC += -I$(ROOT)/components/poc/ydwl/inc
endif

ifeq ($(CONFIG_POC_SLPOC), y)
INC += -I$(ROOT)/components/poc  -I$(ROOT)/components/poc/slpoc/src
INC += -I$(ROOT)/components/poc/slpoc/inc
SRC_QSTR += $(ROOT)/components/poc/slpoc/src/modpoc_slpoc.c
endif

ifeq ($(CONFIG_POC_RDA_BND), y)
INC += -I$(ROOT)/components/poc  -I$(ROOT)/components/poc/rda_bnd/src
INC += -I$(ROOT)/components/poc/rda_bnd/inc
SRC_QSTR += $(ROOT)/components/poc/rda_bnd/src/modpocrda.c
endif

ifeq ($(CONFIG_POC_RDA_BND_XIN), y)
INC += -I$(ROOT)/components/poc  -I$(ROOT)/components/poc/rda_bnd_xin/src
INC += -I$(ROOT)/components/poc/rda_bnd_xin/inc
SRC_QSTR += $(ROOT)/components/poc/rda_bnd_xin/src/modpocrda.c
endif

ifeq ($(CONFIG_POC_TID), y)
INC += -I$(ROOT)/components/poc  -I$(ROOT)/components/poc/tid/src
INC += -I$(ROOT)/components/poc/tid/inc
SRC_QSTR += $(ROOT)/components/poc/tid/src/modpoc_tid.c
endif

ifeq (y,$(CONFIG_DECODE_ZBAR))
INC += -I$(ROOT)/components/decode/ZBar/zbar
INC += -I$(ROOT)/components/decode/ZBar/include
INC += -I$(ROOT)/components/decode/ZBar/zbar/decoder
INC += -I$(ROOT)/components/decode/ZBar/zbar/qrcode
endif
ifeq (y,$(CONFIG_DECODE_QUECTEL))
INC += -I$(ROOT)/components/decode/quectel/inc
endif

ifeq (y,$(CONFIG_DECODE_QINGYA))
ifneq ($(strip $(PLAT)),ASR)
ifneq ($(strip $(PLAT)),ASR_1606)
INC += -I$(ROOT)/components/decode/qingya
endif
endif
endif

ifeq ($(CONFIG_SPI_SDCARD), y)
INC += -I$(ROOT)/components/fs/fatfs/include
endif

ifeq ($(CONFIG_AUDIO_G711_COMPRESS), y)
INC += -I$(ROOT)/components/audio/G711
endif
ifeq ($(strip $(PLAT)),EIGEN)
INC += -I$(ROOT)/components/audio
endif

ifeq ($(CONFIG_GMSSL), y)
INC += -I$(ROOT)/components/GMSSL/include
INC += -I$(ROOT)/components/GMSSL/include/gmssl
endif

ifeq ($(CONFIG_GB2312), y)
INC += -I$(ROOT)/components/decode/gb2312
endif

ifeq ($(CONFIG_AUDIO_G729_COMPRESS), y)
INC += -I$(ROOT)/components/audio/G729
INC += -I$(ROOT)/components/audio/G729/src
INC += -I$(ROOT)/components/audio/G729/include
endif

ifeq ($(CONFIG_NFC_SL6320), y)
INC += -I$(ROOT)/components/NFC/SL6320
endif

ifeq ($(CONFIG_VOIP), y)
INC += -I$(ROOT)/components/voip/libexosip2/include
INC += -I$(ROOT)/components/voip/libosip2/include
INC += -I$(ROOT)/components/voip/rtp
endif

ifeq ($(CONFIG_ETHERNET_W5500), y)
INC += -I$(ROOT)/components/ethernet/wiznet
endif

ifeq ($(CONFIG_SLIP), y)
INC += -I$(ROOT)/components/slip
endif

# compiler settings
CWARN = -Wall -Werror
CWARN += -Wpointer-arith -Wuninitialized
C_FLAGS = $(INC) $(CWARN) -std=gnu99 $(C_FLAGS_MOD) $(COPT) $(C_FLAGS_EXTRA)
C_FLAGS += -fdata-sections -ffunction-sections -fno-asynchronous-unwind-tables --include=../../../../../../config/autoconf.h

LD_FLAGS_ARCH = -Wl,-Map=$@.map,--cref -Wl,--gc-sections
LD_FLAGS = $(LD_FLAGS_MOD) $(LD_FLAGS_ARCH) -lm $(LD_FLAGS_EXTRA)

D_FLAGS += $(foreach dflag, $(DFLAGS), -D$(dflag))
D_FLAGS += -DMP_CONFIGFILE="\"ports/quectel/core/source/mpconfigport.h\"" \
		  -DMICROPY_MPTHREADPORT_H="\"ports/quectel/core/source/mpthreadport.h\"" \
		  -DMICROPY_MPHALPORT_H="\"ports/quectel/core/source/mphalport.h\""

vpath %.S . $(TOP) $(USER_C_MODULES)
$(BUILD)/%.o: %.S
	@$(ECHO) "CC $<"
	$(Q)$(CC) $(C_FLAGS) $(D_FLAGS) -c -o $@ $<

vpath %.s . $(TOP) $(USER_C_MODULES)
$(BUILD)/%.o: %.s
	@$(ECHO) "AS $<"
	$(Q)$(AS) -o $@ $<
QSTR_GEN_EXTRA_C_FLAGS += -DNO_QSTR
vpath %.c . $(TOP) $(USER_C_MODULES)
$(BUILD)/%.pp: %.c
	@$(ECHO) "PreProcess $<"
	$(Q)$(CPP) $(C_FLAGS) $(D_FLAGS) -Wp,-C,-dD,-dI -o $@ $<

#$(OBJ): | $(HEADER_BUILD)/qstrdefs.generated.h $(HEADER_BUILD)/mpversion.h


$(HEADER_BUILD)/qstr.i.last: $(SRC_QSTR) $(QSTR_GLOBAL_DEPENDENCIES) | $(QSTR_GLOBAL_REQUIREMENTS)
	@$(ECHO) "GEN $@"
	@rm -f $@ $@.dep
	@for src in $(if $(filter $?,$(QSTR_GLOBAL_DEPENDENCIES)),$^,$(if $?,$?,$^)); \
	do \
		echo $$src >> $@.dep; \
	done
	@while read -r src; \
		do \
			echo processing on $$src ...; \
			$(CPP) $(QSTR_GEN_EXTRA_C_FLAGS) $(C_FLAGS) $(D_FLAGS) $$src >> $@; \
	done < $@.dep

$(HEADER_BUILD)/qstr.split: $(HEADER_BUILD)/qstr.i.last
	@$(ECHO) "GEN $@"
	$(Q)$(PYTHON) $(PY_SRC)/makeqstrdefs.py split qstr $(HEADER_BUILD)/qstr.i.last $(HEADER_BUILD)/qstr $(QSTR_DEFS_COLLECTED)
	$(Q)$(TOUCH) $@

$(QSTR_DEFS_COLLECTED): $(HEADER_BUILD)/qstr.split
	@$(ECHO) "GEN $@"
	$(Q)$(PYTHON) $(PY_SRC)/makeqstrdefs.py cat qstr $(HEADER_BUILD)/qstr.i.last $(HEADER_BUILD)/qstr $(QSTR_DEFS_COLLECTED)


$(BUILD)/%.mpy: $(FROZEN_MPY_DIR)/%.py
	@$(ECHO) "MPY $<"
	$(Q)$(MKDIR) -p $(dir $@)
	$(Q)$(MPY_CROSS) -o $@ -s $(<:$(FROZEN_MPY_DIR)/%=%) $(MPY_CROSS_FLAGS) $<


CODE_MPY_DIR ?= ./code
$(BUILD)/code/%.mpy: $(CODE_MPY_DIR)/%.py
	@$(ECHO) "MPY $<"
	$(Q)$(MKDIR) -p $(dir $@)
	$(Q)$(MPY_CROSS) -o $@ -s $(<:$(CODE_MPY_DIR)/%=%) $(MPY_CROSS_FLAGS) $<


CODE_TOOLS_MPY_DIR ?= ./code/tools
$(BUILD)/code/tools/%.mpy: $(CODE_TOOLS_MPY_DIR)/%.py
	@$(ECHO) "MPY $<"
	$(Q)$(MKDIR) -p $(dir $@)
	$(Q)$(MPY_CROSS) -o $@ -s $(<:$(CODE_TOOLS_MPY_DIR)/%=%) $(MPY_CROSS_FLAGS) $<


CODE_OCPP_MPY_DIR ?= ./code/ocpp
$(BUILD)/code/ocpp/%.mpy: $(CODE_OCPP_MPY_DIR)/%.py
	@$(ECHO) "MPY $<"
	$(Q)$(MKDIR) -p $(dir $@)
	$(Q)$(MPY_CROSS) -o $@ -s $(<:$(CODE_OCPP_MPY_DIR)/%=%) $(MPY_CROSS_FLAGS) $<


CODE_OCPP_V16_MPY_DIR ?= ./code/ocpp/v16
$(BUILD)/code/ocpp/v16/%.mpy: $(CODE_OCPP_V16_MPY_DIR)/%.py
	@$(ECHO) "MPY $<"
	$(Q)$(MKDIR) -p $(dir $@)
	$(Q)$(MPY_CROSS) -o $@ -s $(<:$(CODE_OCPP_V16_MPY_DIR)/%=%) $(MPY_CROSS_FLAGS) $<


ifeq ($(strip $(PLAT)),RDA)
frozen_mpy_list = 	$(BUILD)/_boot_RDA.mpy \
					$(BUILD)/system.mpy \
					$(BUILD)/dataCall.mpy \
					$(BUILD)/umqtt.mpy \
					$(BUILD)/sys_bus.mpy \
					$(BUILD)/queue.mpy \
					$(BUILD)/log.mpy \
					$(BUILD)/request.mpy \
					$(BUILD)/ql_fs.mpy \
					$(BUILD)/app_fota_mount.mpy \
					$(BUILD)/app_fota_download.mpy \
					$(BUILD)/checksum.mpy \
					$(BUILD)/file_crc32.mpy \
					$(BUILD)/app_fota_updater.mpy \
					$(BUILD)/app_fota.mpy \
					$(BUILD)/backup_restore.mpy \
					$(BUILD)/event_message.mpy \
					$(BUILD)/ntptime.mpy \
					$(BUILD)/checkNet.mpy \
					$(BUILD)/uping.mpy \


ifeq ($(strip $(BOARD)),$(filter $(strip $(BOARD)),BC92RB))
	frozen_mpy_list += $(BUILD)/ftplib.mpy \
					   $(BUILD)/ftplibtls.mpy 
endif					

ifeq ($(strip $(BOARD)),$(filter $(strip $(BOARD)),BC25B8PA_LBS))
	frozen_mpy_list += 	$(BUILD)/cellLocator.mpy 
endif

ifneq (,$(findstring BC25,$(BOARD)))
	frozen_mpy_list += 	$(BUILD)/ymodem.mpy
endif

ifeq ($(strip $(BOARD)),$(filter $(strip $(BOARD)),BC32RA))
	frozen_mpy_list += $(BUILD)/bak_util.mpy
endif

else ifeq ($(strip $(PLAT)),ASR_1803s)
frozen_mpy_list = 	$(BUILD)/_boot.mpy \
					$(BUILD)/system.mpy \
					$(BUILD)/checkNet.mpy \
					$(BUILD)/ntptime.mpy \
					$(BUILD)/dataCall.mpy \
					$(BUILD)/log.mpy \
					$(BUILD)/request.mpy \
					$(BUILD)/umqtt.mpy \
					$(BUILD)/ql_fs.mpy \
					$(BUILD)/app_fota_mount.mpy \
					$(BUILD)/app_fota_download.mpy \
					$(BUILD)/checksum.mpy \
					$(BUILD)/file_crc32.mpy \
					$(BUILD)/app_fota_updater.mpy \
					$(BUILD)/app_fota.mpy \
					$(BUILD)/backup_restore.mpy \
					$(BUILD)/queue.mpy \
					$(BUILD)/sys_bus.mpy \
					$(BUILD)/event_message.mpy \
					$(BUILD)/bak_util.mpy \
					$(BUILD)/cellLocator.mpy \
					$(BUILD)/ftplib.mpy \
					$(BUILD)/ftplibtls.mpy \
					$(BUILD)/uwebsocket.mpy \
					$(BUILD)/uping.mpy \
					$(BUILD)/ymodem.mpy \

else ifeq ($(strip $(PLAT)),Qualcomm)
frozen_mpy_list = 	$(BUILD)/_boot_Qualcomm.mpy \
					$(BUILD)/system.mpy \
					$(BUILD)/checkNet.mpy \
					$(BUILD)/ntptime.mpy \
					$(BUILD)/dataCall.mpy \
					$(BUILD)/umqtt.mpy \
					$(BUILD)/log.mpy \
					$(BUILD)/request.mpy \
					$(BUILD)/ql_fs.mpy \
					$(BUILD)/app_fota_download.mpy \
					$(BUILD)/checksum.mpy \
					$(BUILD)/file_crc32.mpy \
					$(BUILD)/app_fota_mount.mpy \
					$(BUILD)/app_fota_updater.mpy \
					$(BUILD)/app_fota.mpy \
					$(BUILD)/backup_restore.mpy \
					$(BUILD)/queue.mpy \
					$(BUILD)/sys_bus.mpy \
					$(BUILD)/event_message.mpy \
					$(BUILD)/bak_util.mpy \
					$(BUILD)/ftplib.mpy \
					$(BUILD)/ftplibtls.mpy \
					$(BUILD)/uwebsocket.mpy \
					$(BUILD)/uping.mpy \
					$(BUILD)/ymodem.mpy \

ifeq ($(strip $(BOARD)),$(filter $(strip $(BOARD)),BG600LM3))
frozen_mpy_list += 	$(BUILD)/gnss.mpy
endif					

else ifeq ($(strip $(PLAT)),ASR_1606)
ifeq ($(strip $(BOARD)),$(filter $(strip $(BOARD)),EC800KCN_CC EC600MCN_CC EC600MCN_CC_EXT EC600MCN_LF_SLPOC EC600MCN_CC_EXT_TTS EC800MCN_CC_TTS))
frozen_mpy_list = 	$(BUILD)/_boot.mpy \
					$(BUILD)/system.mpy \
					$(BUILD)/dataCall.mpy \
					$(BUILD)/umqtt.mpy \
					$(BUILD)/queue.mpy \
					$(BUILD)/log.mpy \
					$(BUILD)/request.mpy \
					$(BUILD)/ql_fs.mpy \
					$(BUILD)/app_fota_mount.mpy \
					$(BUILD)/app_fota_download.mpy \
					$(BUILD)/checksum.mpy \
					$(BUILD)/file_crc32.mpy \
					$(BUILD)/app_fota_updater.mpy \
					$(BUILD)/app_fota.mpy \
					$(BUILD)/backup_restore.mpy \
					$(BUILD)/ntptime.mpy \
					$(BUILD)/checkNet.mpy \
					$(BUILD)/ymodem.mpy

else ifeq ($(strip $(BOARD)),$(filter $(strip $(BOARD)),EC800MCN_CC))
frozen_mpy_list = 	$(BUILD)/_boot.mpy \
					$(BUILD)/system.mpy \
					$(BUILD)/dataCall.mpy \
					$(BUILD)/queue.mpy \
					$(BUILD)/log.mpy \
					$(BUILD)/request.mpy \
					$(BUILD)/ntptime.mpy \
					$(BUILD)/checkNet.mpy \
					$(BUILD)/ftplib.mpy \
					$(BUILD)/cellLocator.mpy

else
frozen_mpy_list = 	$(BUILD)/_boot.mpy \
					$(BUILD)/umqtt.mpy \
					$(BUILD)/ntptime.mpy \
					$(BUILD)/request.mpy \
					$(BUILD)/log.mpy \
					$(BUILD)/aLiYun.mpy \
					$(BUILD)/hmac.mpy \
					$(BUILD)/hashlib.mpy \
					$(BUILD)/base64.mpy \
					$(BUILD)/TenCentYun.mpy \
					$(BUILD)/file_sha256.mpy \
					$(BUILD)/checksum.mpy \
					$(BUILD)/checkNet.mpy \
					$(BUILD)/dataCall.mpy \
					$(BUILD)/ql_fs.mpy \
					$(BUILD)/app_fota_mount.mpy \
					$(BUILD)/app_fota_download.mpy\
					$(BUILD)/app_fota_updater.mpy \
					$(BUILD)/app_fota.mpy \
					$(BUILD)/backup_restore.mpy \
					$(BUILD)/file_crc32.mpy \
					$(BUILD)/system.mpy \
					$(BUILD)/gnss.mpy \
					$(BUILD)/unzip.mpy \
					$(BUILD)/queue.mpy \
					$(BUILD)/sys_bus.mpy \
					$(BUILD)/event_message.mpy \
					$(BUILD)/hls.mpy \
					$(BUILD)/wifilocator.mpy \
					$(BUILD)/cellLocator.mpy \
					$(BUILD)/bak_util.mpy \
					$(BUILD)/uwebsocket.mpy \
					$(BUILD)/uping.mpy \
					$(BUILD)/ymodem.mpy \
					$(BUILD)/copy.mpy \
					$(BUILD)/types.mpy 

ifeq ($(strip $(BOARD)),$(filter $(strip $(BOARD)),EC800MCN_LE_VOLVGL EC800MCN_LE_CPE))
frozen_mpy_list += 	$(BUILD)/CMCC_DM.mpy
endif

endif

else ifeq ($(strip $(PLAT)),ASR_1602)
ifeq ($(strip $(BOARD)),$(filter $(strip $(BOARD)),EC800KCN_CC EC800KCN_CC_TTS EC600KCN_CC_TTS))
frozen_mpy_list = 	$(BUILD)/_boot.mpy \
					$(BUILD)/system.mpy \
					$(BUILD)/dataCall.mpy \
					$(BUILD)/umqtt.mpy \
					$(BUILD)/queue.mpy \
					$(BUILD)/log.mpy \
					$(BUILD)/request.mpy \
					$(BUILD)/ql_fs.mpy \
					$(BUILD)/app_fota_mount.mpy \
					$(BUILD)/app_fota_download.mpy \
					$(BUILD)/checksum.mpy \
					$(BUILD)/file_crc32.mpy \
					$(BUILD)/app_fota_updater.mpy \
					$(BUILD)/app_fota.mpy \
					$(BUILD)/backup_restore.mpy \
					$(BUILD)/ntptime.mpy \
					$(BUILD)/checkNet.mpy \
					$(BUILD)/ymodem.mpy

else
frozen_mpy_list = 	$(BUILD)/_boot.mpy \
					$(BUILD)/umqtt.mpy \
					$(BUILD)/ntptime.mpy \
					$(BUILD)/request.mpy \
					$(BUILD)/log.mpy \
					$(BUILD)/aLiYun.mpy \
					$(BUILD)/hmac.mpy \
					$(BUILD)/hashlib.mpy \
					$(BUILD)/base64.mpy \
					$(BUILD)/TenCentYun.mpy \
					$(BUILD)/file_sha256.mpy \
					$(BUILD)/checksum.mpy \
					$(BUILD)/checkNet.mpy \
					$(BUILD)/dataCall.mpy \
					$(BUILD)/ql_fs.mpy \
					$(BUILD)/app_fota_mount.mpy \
					$(BUILD)/app_fota_download.mpy\
					$(BUILD)/app_fota_updater.mpy \
					$(BUILD)/app_fota.mpy \
					$(BUILD)/backup_restore.mpy \
					$(BUILD)/file_crc32.mpy \
					$(BUILD)/system.mpy \
					$(BUILD)/gnss.mpy \
					$(BUILD)/unzip.mpy \
					$(BUILD)/queue.mpy \
					$(BUILD)/sys_bus.mpy \
					$(BUILD)/event_message.mpy \
					$(BUILD)/hls.mpy \
					$(BUILD)/wifilocator.mpy \
					$(BUILD)/cellLocator.mpy \
					$(BUILD)/bak_util.mpy \
					$(BUILD)/uwebsocket.mpy \
					$(BUILD)/uping.mpy \
					$(BUILD)/ymodem.mpy \

endif

else ifeq ($(strip $(PLAT)),ASR_1609)
frozen_mpy_list = 	$(BUILD)/_boot.mpy \
					$(BUILD)/umqtt.mpy \
					$(BUILD)/ntptime.mpy \
					$(BUILD)/request.mpy \
					$(BUILD)/log.mpy \
					$(BUILD)/aLiYun.mpy \
					$(BUILD)/hmac.mpy \
					$(BUILD)/hashlib.mpy \
					$(BUILD)/base64.mpy \
					$(BUILD)/TenCentYun.mpy \
					$(BUILD)/file_sha256.mpy \
					$(BUILD)/checksum.mpy \
					$(BUILD)/checkNet.mpy \
					$(BUILD)/dataCall.mpy \
					$(BUILD)/ql_fs.mpy \
					$(BUILD)/app_fota_mount.mpy \
					$(BUILD)/app_fota_download.mpy\
					$(BUILD)/app_fota_updater.mpy \
					$(BUILD)/app_fota.mpy \
					$(BUILD)/backup_restore.mpy \
					$(BUILD)/file_crc32.mpy \
					$(BUILD)/system.mpy \
					$(BUILD)/gnss.mpy \
					$(BUILD)/unzip.mpy \
					$(BUILD)/queue.mpy \
					$(BUILD)/sys_bus.mpy \
					$(BUILD)/event_message.mpy \
					$(BUILD)/hls.mpy \
					$(BUILD)/wifilocator.mpy \
					$(BUILD)/cellLocator.mpy \
					$(BUILD)/bak_util.mpy \
					$(BUILD)/uwebsocket.mpy \
					$(BUILD)/uping.mpy \
					$(BUILD)/ymodem.mpy \

else ifeq ($(strip $(PLAT)),$(filter $(PLAT),Unisoc_8850 Unisoc_8850_R02))
ifeq ($(strip $(BOARD)),$(filter $(strip $(BOARD)),EC600GCN_LD EC600GCN_LD_YM EC800GCN_LD EC800GCN_LD_HRXM EC800GCN_LD_XBND EG700GCN_LC))
frozen_mpy_list = 	$(BUILD)/_boot.mpy \
					$(BUILD)/umqtt.mpy \
					$(BUILD)/ntptime.mpy \
					$(BUILD)/request.mpy \
					$(BUILD)/log.mpy \
					$(BUILD)/checksum.mpy \
					$(BUILD)/checkNet.mpy \
					$(BUILD)/dataCall.mpy \
					$(BUILD)/ql_fs.mpy \
					$(BUILD)/app_fota_mount.mpy \
					$(BUILD)/app_fota_download.mpy\
					$(BUILD)/app_fota_updater.mpy \
					$(BUILD)/app_fota.mpy \
					$(BUILD)/backup_restore.mpy \
					$(BUILD)/file_crc32.mpy \
					$(BUILD)/system.mpy \
					$(BUILD)/queue.mpy \
					$(BUILD)/sys_bus.mpy \
					$(BUILD)/event_message.mpy \
					$(BUILD)/bak_util.mpy \
					$(BUILD)/ymodem.mpy \
					
else ifeq ($(strip $(BOARD)),$(filter $(strip $(BOARD)),EC600GCN_LA_CDD))
frozen_mpy_list = 	$(BUILD)/_boot_dsds.mpy \
					$(BUILD)/umqtt.mpy \
					$(BUILD)/ntptime.mpy \
					$(BUILD)/request.mpy \
					$(BUILD)/log.mpy \
					$(BUILD)/aLiYun.mpy \
					$(BUILD)/hmac.mpy \
					$(BUILD)/hashlib.mpy \
					$(BUILD)/base64.mpy \
					$(BUILD)/TenCentYun.mpy \
					$(BUILD)/file_sha256.mpy \
					$(BUILD)/checksum.mpy \
					$(BUILD)/checkNet.mpy \
					$(BUILD)/dataCall.mpy \
					$(BUILD)/ql_fs.mpy \
					$(BUILD)/app_fota_mount.mpy \
					$(BUILD)/app_fota_download.mpy\
					$(BUILD)/app_fota_updater.mpy \
					$(BUILD)/app_fota.mpy \
					$(BUILD)/backup_restore.mpy \
					$(BUILD)/file_crc32.mpy \
					$(BUILD)/system.mpy \
					$(BUILD)/gnss.mpy \
					$(BUILD)/unzip.mpy \
					$(BUILD)/queue.mpy \
					$(BUILD)/sys_bus.mpy \
					$(BUILD)/event_message.mpy \
					$(BUILD)/hls.mpy \
					$(BUILD)/wifilocator.mpy \
					$(BUILD)/bak_util.mpy \
					$(BUILD)/ftplib.mpy \
					$(BUILD)/ftplibtls.mpy \
					$(BUILD)/uwebsocket.mpy \
					$(BUILD)/uping.mpy \
					$(BUILD)/ymodem.mpy 
else
frozen_mpy_list = 	$(BUILD)/_boot.mpy \
					$(BUILD)/umqtt.mpy \
					$(BUILD)/ntptime.mpy \
					$(BUILD)/request.mpy \
					$(BUILD)/log.mpy \
					$(BUILD)/aLiYun.mpy \
					$(BUILD)/hmac.mpy \
					$(BUILD)/hashlib.mpy \
					$(BUILD)/base64.mpy \
					$(BUILD)/TenCentYun.mpy \
					$(BUILD)/file_sha256.mpy \
					$(BUILD)/checksum.mpy \
					$(BUILD)/checkNet.mpy \
					$(BUILD)/dataCall.mpy \
					$(BUILD)/ql_fs.mpy \
					$(BUILD)/app_fota_mount.mpy \
					$(BUILD)/app_fota_download.mpy\
					$(BUILD)/app_fota_updater.mpy \
					$(BUILD)/app_fota.mpy \
					$(BUILD)/backup_restore.mpy \
					$(BUILD)/file_crc32.mpy \
					$(BUILD)/system.mpy \
					$(BUILD)/gnss.mpy \
					$(BUILD)/unzip.mpy \
					$(BUILD)/queue.mpy \
					$(BUILD)/sys_bus.mpy \
					$(BUILD)/event_message.mpy \
					$(BUILD)/hls.mpy \
					$(BUILD)/wifilocator.mpy \
					$(BUILD)/bak_util.mpy \
					$(BUILD)/ftplib.mpy \
					$(BUILD)/ftplibtls.mpy \
					$(BUILD)/uwebsocket.mpy \
					$(BUILD)/uping.mpy \
					$(BUILD)/ymodem.mpy 
					
endif

else ifeq ($(strip $(PLAT)),$(filter $(PLAT),EIGEN EIGEN_718))
ifeq ($(strip $(BOARD)),$(filter $(strip $(BOARD)),EC800ECN_LC_WDF))
frozen_mpy_list = 	$(BUILD)/_boot_WDF.mpy \
					$(BUILD)/umqtt.mpy \
					$(BUILD)/dataCall.mpy \
					$(BUILD)/log.mpy \
					$(BUILD)/request.mpy \
					$(BUILD)/aLiYun.mpy \
					$(BUILD)/hmac.mpy \
					$(BUILD)/hashlib.mpy \
					$(BUILD)/ntptime.mpy \
					$(BUILD)/checksum.mpy \
					$(BUILD)/checkNet.mpy \
					$(BUILD)/ql_fs.mpy \
					$(BUILD)/app_fota_mount.mpy \
					$(BUILD)/app_fota_download.mpy\
					$(BUILD)/app_fota_updater.mpy \
					$(BUILD)/app_fota.mpy \
					$(BUILD)/backup_restore.mpy \
					$(BUILD)/file_crc32.mpy \
					$(BUILD)/system.mpy \
					$(BUILD)/queue.mpy \
					$(BUILD)/sys_bus.mpy \
					$(BUILD)/event_message.mpy \
					$(BUILD)/bak_util.mpy \
					$(BUILD)/ftplib.mpy \
					$(BUILD)/ftplibtls.mpy \
					$(BUILD)/ymodem.mpy \

else
frozen_mpy_list = 	$(BUILD)/_boot.mpy \
					$(BUILD)/umqtt.mpy \
					$(BUILD)/dataCall.mpy \
					$(BUILD)/log.mpy \
					$(BUILD)/request.mpy \
					$(BUILD)/aLiYun.mpy \
					$(BUILD)/hmac.mpy \
					$(BUILD)/hashlib.mpy \
					$(BUILD)/ntptime.mpy \
					$(BUILD)/checksum.mpy \
					$(BUILD)/checkNet.mpy \
					$(BUILD)/ql_fs.mpy \
					$(BUILD)/app_fota_mount.mpy \
					$(BUILD)/app_fota_download.mpy\
					$(BUILD)/app_fota_updater.mpy \
					$(BUILD)/app_fota.mpy \
					$(BUILD)/backup_restore.mpy \
					$(BUILD)/file_crc32.mpy \
					$(BUILD)/system.mpy \
					$(BUILD)/queue.mpy \
					$(BUILD)/sys_bus.mpy \
					$(BUILD)/event_message.mpy \
					$(BUILD)/bak_util.mpy \
					$(BUILD)/ftplib.mpy \
					$(BUILD)/ftplibtls.mpy \
					$(BUILD)/cellLocator.mpy \
					$(BUILD)/ymodem.mpy \
					
endif
					
else ifeq ($(strip $(PLAT)),ECR6600)
frozen_mpy_list = 	$(BUILD)/_boot_WIFI.mpy \
					$(BUILD)/umqtt.mpy \
					$(BUILD)/request.mpy \
					$(BUILD)/ql_fs.mpy \
					$(BUILD)/queue.mpy \
					$(BUILD)/sys_bus.mpy \
					$(BUILD)/log.mpy \
					$(BUILD)/ymodem.mpy \
					$(BUILD)/code/v16_client_qpy_demo.mpy \
					$(BUILD)/code/tools/logging.mpy \
					$(BUILD)/code/tools/uuid.mpy \
					$(BUILD)/code/tools/uwebsocket.mpy \
					$(BUILD)/code/ocpp/charge_point.mpy \
					$(BUILD)/code/ocpp/dataclasses.mpy \
					$(BUILD)/code/ocpp/exceptions.mpy \
					$(BUILD)/code/ocpp/messages.mpy \
					$(BUILD)/code/ocpp/routing.mpy \
					$(BUILD)/code/ocpp/v16/__init__.mpy \
					$(BUILD)/code/ocpp/v16/call.mpy \
					$(BUILD)/code/ocpp/v16/call_result.mpy \
					$(BUILD)/code/ocpp/v16/datatypes.mpy \
					$(BUILD)/code/ocpp/v16/enums.mpy \

else ifeq ($(strip $(PLAT)),aic8800m40)
frozen_mpy_list = 	$(BUILD)/_boot_WIFI.mpy \
					$(BUILD)/umqtt.mpy \
					$(BUILD)/request.mpy \
					$(BUILD)/ql_fs.mpy \
					$(BUILD)/queue.mpy \
					$(BUILD)/sys_bus.mpy \
					$(BUILD)/log.mpy \
					$(BUILD)/ymodem.mpy \
					$(BUILD)/system.mpy \
					$(BUILD)/checksum.mpy \
					$(BUILD)/checkNet.mpy \
					$(BUILD)/ql_fs.mpy \
					$(BUILD)/app_fota_download.mpy \
					$(BUILD)/app_fota_mount.mpy \
					$(BUILD)/app_fota_updater.mpy \
					$(BUILD)/app_fota.mpy \
					$(BUILD)/file_crc32.mpy \
					$(BUILD)/ntptime.mpy \
					$(BUILD)/aLiYun.mpy \
					$(BUILD)/hmac.mpy \
					$(BUILD)/hashlib.mpy \


else
ifeq ($(strip $(BOARD)),$(filter $(strip $(BOARD)),EC600UEU_AB_POC))
frozen_mpy_list = 	$(BUILD)/_boot.mpy \
					$(BUILD)/ntptime.mpy \
					$(BUILD)/log.mpy \
					$(BUILD)/file_sha256.mpy \
					$(BUILD)/checksum.mpy \
					$(BUILD)/checkNet.mpy \
					$(BUILD)/dataCall.mpy \
					$(BUILD)/ql_fs.mpy \
					$(BUILD)/app_fota_mount.mpy \
					$(BUILD)/app_fota_download.mpy\
					$(BUILD)/app_fota_updater.mpy \
					$(BUILD)/app_fota.mpy \
					$(BUILD)/backup_restore.mpy \
					$(BUILD)/file_crc32.mpy \
					$(BUILD)/system.mpy \
					$(BUILD)/queue.mpy \
					$(BUILD)/bak_util.mpy \
					$(BUILD)/ymodem.mpy 
else
frozen_mpy_list = 	$(BUILD)/_boot.mpy \
					$(BUILD)/umqtt.mpy \
					$(BUILD)/ntptime.mpy \
					$(BUILD)/request.mpy \
					$(BUILD)/log.mpy \
					$(BUILD)/aLiYun.mpy \
					$(BUILD)/hmac.mpy \
					$(BUILD)/hashlib.mpy \
					$(BUILD)/base64.mpy \
					$(BUILD)/TenCentYun.mpy \
					$(BUILD)/file_sha256.mpy \
					$(BUILD)/checksum.mpy \
					$(BUILD)/checkNet.mpy \
					$(BUILD)/dataCall.mpy \
					$(BUILD)/ql_fs.mpy \
					$(BUILD)/app_fota_mount.mpy \
					$(BUILD)/app_fota_download.mpy\
					$(BUILD)/app_fota_updater.mpy \
					$(BUILD)/app_fota.mpy \
					$(BUILD)/backup_restore.mpy \
					$(BUILD)/file_crc32.mpy \
					$(BUILD)/system.mpy \
					$(BUILD)/gnss.mpy \
					$(BUILD)/unzip.mpy \
					$(BUILD)/queue.mpy \
					$(BUILD)/sys_bus.mpy \
					$(BUILD)/event_message.mpy \
					$(BUILD)/hls.mpy \
					$(BUILD)/wifilocator.mpy \
					$(BUILD)/bak_util.mpy \
					$(BUILD)/ftplib.mpy \
					$(BUILD)/ftplibtls.mpy \
					$(BUILD)/uwebsocket.mpy \
					$(BUILD)/uping.mpy \
					$(BUILD)/ymodem.mpy 
endif
endif

$(BUILD)/_frozen_mpy.c: $(frozen_mpy_list) $(HEADER_BUILD)/qstrdefs.generated.h
	@$(ECHO) "MISC freezing bytecode"
	$(Q)$(MPY_TOOL) -f -q $(HEADER_BUILD)/qstrdefs.preprocessed.h $(frozen_mpy_list) > $@

$(BUILD):
	$(Q)echo creating directory: $@
	-$(Q)$(MKDIR) $@

all:$(BUILD) $(HEADER_BUILD)/qstrdefs.generated.h $(HEADER_BUILD)/moduledefs.h  $(HEADER_BUILD)/compressed.data.h $(BUILD)/_frozen_mpy.c

clean:
	@echo ----- cleaning microPython private target
	@rm -rf $(HEADER_BUILD)
	@rm -rf $(BUILD)
	@echo ----- done.
