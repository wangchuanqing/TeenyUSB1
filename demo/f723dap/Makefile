TARGET = f723dap
BOARD = stm32f723e_disco

BOARD_PATH = ../boards/$(BOARD)
include $(BOARD_PATH)/board.mk

# TeenyDT format descriptor
TEENYDT = cmsis_dap_desc.lua
USBMEM = 1280
USBEP = 6
# project source
SOURCES  = hid_class.c \
           f723_dap.c \
           teeny_usb_desc.c
INCLUDES = . 
DEFINES  = __forceinline=inline NO_HOST
# board source
SOURCES  += $(BOARD_SRC)
INCLUDES += $(BOARD_INC)
DEFINES  += $(BOARD_DEF)
# CMSIS-DAP source
DAP_PATH = ../cmsis-dap
SOURCES  += $(DAP_PATH)/DAP_queue.c \
            $(DAP_PATH)/DAP.c \
            $(DAP_PATH)/JTAG_DP.c \
            $(DAP_PATH)/SW_DP.c \
            $(DAP_PATH)/SWO.c
INCLUDES += $(DAP_PATH)
# Teeny USB source
TUSB_PATH = ../../usb_stack
SOURCES  += $(wildcard $(TUSB_PATH)/src/*.c)
INCLUDES += $(TUSB_PATH)/inc
INCLUDES += $(TUSB_PATH)/Drivers/CMSIS/Include

TOOLSET     ?= arm-none-eabi-
CC           = $(TOOLSET)gcc
LD           = $(TOOLSET)gcc
AR           = $(TOOLSET)ar
AS           = $(TOOLSET)as
SIZE         = $(TOOLSET)size
OBJCOPY      = $(TOOLSET)objcopy
RM           = rm -rf
CP           = cp

CPU ?= cortex-m3
CCFLAGS     += -Wall -mcpu=$(CPU) -mthumb -g -Os
CCFLAGS     += $(addprefix -D, $(DEFINES)) 
CCFLAGS     += $(addprefix -I, $(INCLUDES))
CCFLAGS     += -ffunction-sections -fdata-sections
LDFLAGS     += $(CCFLAGS)
LDFLAGS     += -Wl,--script='$(LDSCRIPT)' -Wl,-Map=$(OUTDIR)/$(TARGET).map
LDFLAGS     += -Wl,--gc-sections

OUTDIR = ./output
OBJDIR = ./obj
OBJECTS      = $(addprefix $(OBJDIR)/, $(addsuffix .o, $(notdir $(basename $(SOURCES)))))
DEPENDS      = $(addprefix $(OBJDIR)/, $(addsuffix .d, $(notdir $(basename $(filter %c, $(SOURCES))))))
SRCPATH      = $(sort $(dir $(SOURCES)))
vpath %.c $(SRCPATH)
vpath %.S $(SRCPATH)
vpath %.s $(SRCPATH)

all: $(OUTDIR) $(OUTDIR)/$(TARGET).bin $(OUTDIR)/$(TARGET).hex

$(OBJDIR)/%.o:%.c
	@echo building $<
	@$(CC) $(CCFLAGS) -c $< -o $@
$(OBJDIR)/%.o:%.s
	@echo building $<
	@$(CC) $(ASFLAGS) -c $< -o $@
$(OBJDIR)/%.o:%.S
	@echo building $<
	@$(CC) $(ASFLAGS) -c $< -o $@

$(OBJDIR)/%.d:%.c
	@echo make depend $<
	@set -e; $(RM) $@; $(CC) -MM $< $(CCFLAGS) > $@.$$$$; \
	sed 's,\($*\)\.o[ :]*,$(OBJDIR)/\1.o $@ : ,g' < $@.$$$$ > $@; \
	$(RM) $@.$$$$

include $(wildcard $(DEPENDS))

$(OBJDIR):
	@mkdir $@
$(OUTDIR):
	@mkdir $@

$(OUTDIR)/$(TARGET).hex:$(OUTDIR)/$(TARGET).elf
	@echo generating $@
	@$(OBJCOPY) -Oihex $< $@
$(OUTDIR)/$(TARGET).bin:$(OUTDIR)/$(TARGET).elf
	@echo generating $@
	@$(OBJCOPY) -Obinary $< $@
$(OUTDIR)/$(TARGET).elf: $(OBJDIR) $(DEPENDS) $(OBJECTS)
	@echo linking $@
	@$(LD) $(LDFLAGS) $(OBJECTS) -o $@
	@$(SIZE) $@

USBMEM ?= 512
USBEP  ?= 7

PWD = $(shell pwd)
teeny_usb_desc.c teeny_usb_init.h:$(TEENYDT)
	@echo generate USB descriptor
	cd $(PWD)/../../TeenyDT && \
	lua gen_descriptor.lua $(PWD)/$< -maxmem=$(USBMEM) -maxep=$(USBEP) && \
	cd $(PWD)

clean:
	$(RM) $(OUTDIR)/* $(OBJDIR)/*
clear:
	$(RM) $(OUTDIR) obj

.PHONY: all clean clear
