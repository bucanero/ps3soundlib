.SUFFIXES:
ifeq ($(strip $(PSL1GHT)),)
$(error "PSL1GHT must be set in the environment.")
endif

PREFIX		:= spu-

########################################################
OBJCOPY		:=	$(PREFIX)objcopy
AR			:=	$(PREFIX)ar
AS			:=	$(PREFIX)gcc
CC			:=	$(PREFIX)gcc
CXX			:=	$(PREFIX)g++
STRIP		:=	$(PREFIX)strip
RAW2H		:=	$(PS3DEV)/bin/raw2h

ifeq ($(findstring $(PREFIX),$(LD)),)
LD			:=	$(CC)
endif

DEPSOPTIONS	=	-MMD -MP -MF $(DEPSDIR)/$*.d

%.o: %.c
	@echo "[CC]  $(notdir $<)"
	@$(CC) $(DEPSOPTIONS) $(CFLAGS) $(INCLUDES) -c $< -o $@

%.o: %.cpp
	@echo "[CXX] $(notdir $<)"
	@$(CXX) $(DEPSOPTIONS) $(CXXFLAGS) $(INCLUDES) -c $< -o $@

%.o: %.S
	@echo "[CC]  $(notdir $<)"
	@$(AS) $(DEPSOPTIONS) -x assembler-with-cpp $(ASFLAGS) $(INCLUDES) -c $< -o $@

%.a:
	@echo "[AR]  $(notdir $@)"
	@$(AR) -rcs $@ $^

%.elf:
	@echo "[LD]  $(notdir $@)"
	@$(LD) $^ $(LIBPATHS) $(LIBS) $(LDFLAGS) -o $@

%.bin.h: %.bin
	@echo "[R2H] $(notdir $<)"
	@$(RAW2H) $< $(BUILDDIR)/$(notdir $<).h $(BUILDDIR)/$(notdir $<).S $(notdir $(basename $<)_bin)
	@$(AS) -x assembler-with-cpp $(ASFLAGS) -c $(BUILDDIR)/$(notdir $<).S -o $(BUILDDIR)/$(notdir $<).o


########################################################

TARGET		:=	$(notdir $(CURDIR))
BUILD		:=	build
SOURCE		:=	source
INCLUDE		:=	include
DATA		:=	data
LIBS		:=	-lc

PORTLIBS	:=	$(PS3DEV)/portlibs/ppu

CFLAGS		:=	-g -O2 -Wall --std=gnu99
CXXFLAGS	:=	-g -O2 -Wall
LDFLAGS		:=	

ifneq ($(BUILD),$(notdir $(CURDIR)))

export OUTPUT	:=	$(CURDIR)/$(TARGET)
export VPATH	:=	$(foreach dir,$(SOURCE),$(CURDIR)/$(dir)) \
					$(foreach dir,$(DATA),$(CURDIR)/$(dir))
export BUILDDIR	:=	$(CURDIR)/$(BUILD)
export DEPSDIR	:=	$(BUILDDIR)

CFILES			:=	$(foreach dir,$(SOURCE),$(notdir $(wildcard $(dir)/*.c)))
CXXFILES		:=	$(foreach dir,$(SOURCE),$(notdir $(wildcard $(dir)/*.cpp)))
SFILES			:=	$(foreach dir,$(SOURCE),$(notdir $(wildcard $(dir)/*.S)))
BINFILES		:=	$(foreach dir,$(DATA),$(notdir $(wildcard $(dir)/*.bin)))

export OFILES	:=	$(CFILES:.c=.o) \
					$(CXXFILES:.cpp=.o) \
					$(SFILES:.S=.o)

export BINFILES	:=	$(BINFILES:.bin=.bin.h)

export INCLUDES	:=	$(foreach dir,$(INCLUDE),-I$(CURDIR)/$(dir)) \
					-I$(CURDIR)/$(BUILD)

.PHONY: $(BUILD) clean

$(BUILD):
	@[ -d $@ ] || mkdir -p $@
	@make --no-print-directory -C $(BUILD) -f $(CURDIR)/Makefile
	@[ -d lib ] || mkdir -p lib
	@$(RAW2H) $(POSTFIX) $(TARGET).elf lib/$(TARGET)_bin.h lib/$(TARGET).S $(TARGET)_bin
	@ppu-as -mcell lib/$(TARGET).S -o lib/$(TARGET).o
	@ppu-ar -rcs lib$(TARGET).a lib/$(TARGET).o

install: $(BUILD) 
	@[ -d $(PORTLIBS)/modules ] || mkdir -p $(PORTLIBS)/modules
	@[ -d $(PORTLIBS)/modules/include ] || mkdir -p $(PORTLIBS)/modules/include
	@[ -d $(PORTLIBS)/modules/lib ] || mkdir -p $(PORTLIBS)/modules/lib
	@cp -v -f lib/*.h $(PORTLIBS)/modules/include
	@cp -v -f lib/../*.a $(PORTLIBS)/modules/lib
	@cp -v -f $(TARGET).elf $(PORTLIBS)/modules/$(TARGET).bin

clean:
	@echo Clean...
	@rm -rf $(BUILD) $(OUTPUT).elf $(OUTPUT).a $(OUTPUT).bin.a lib

else

DEPENDS	:= $(OFILES:.o=.d)

$(OUTPUT).elf: $(OFILES)
$(OFILES): $(BINFILES)

-include $(DEPENDS)

endif
