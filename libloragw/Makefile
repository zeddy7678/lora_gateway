### get external defined data

LIBLORAGW_VERSION := `cat ../VERSION`
include library.cfg

### constant symbols

ARCH ?=
CROSS_COMPILE ?=
CC := $(CROSS_COMPILE)gcc
AR := $(CROSS_COMPILE)ar


### Test if the file has been configured, do not accept the default

ifeq ($(CFG_SPI),???) 
  $(error Please specify CFG_SPI in lora_gateway/libloragw/library.cfg first!)
endif

ifeq ($(PLATFORM),???) 
  $(error Please specify PLATFORM in lora_gateway/libloragw/library.cfg first!)
endif


### library.cfg configuration file processing

ifeq ($(CFG_SPI),native)
  CFLAGS      := -O2 -Wall -Wextra -std=c99 -Iinc -I.
  CFG_SPI_MSG := Linux native SPI driver
  CFG_SPI_OPT := CFG_SPI_NATIVE
else ifeq ($(CFG_SPI),ftdi)
ifeq ($(PLATFORM),multitech)
  CFLAGS      := -O2 -Wall -Wextra -std=c99 -Iinc -I. -I/opt/mlinux/3.2.0/sysroots/arm926ejste-mlinux-linux-gnueabi/usr/include/libftdi1
else
  CFLAGS      := -O2 -Wall -Wextra -std=c99 -Iinc -I. -I/usr/include/libftdi1
endif
  CFG_SPI_MSG := FTDI SPI-over-USB bridge using libmpsse/libftdi/libusb
  CFG_SPI_OPT := CFG_SPI_FTDI
else ifeq ($(CFG_SPI),mac)
  CFLAGS      := -O2 -Wall -Wextra -std=c99 -Iinc -I. -I/opt/local/include  -I/usr/local/include -I/opt/local/include/libftdi1 
  CFG_SPI_MSG := FTDI SPI-over-USB bridge on the MAC using libmpsse/libftdi/libusb
  CFG_SPI_OPT := CFG_SPI_FTDI
else
  $(error No SPI physical layer selected, check ../target.cfg file.)
endif

OBJDIR = obj
INCLUDES = $(wildcard inc/*.h)

### linking options

ifeq ($(CFG_SPI),native)
  LIBS := -lloragw -lrt -lm
else ifeq ($(CFG_SPI),ftdi)
  LIBS := -lloragw -lrt -lmpsse -lm -lusb-1.0
else ifeq ($(CFG_SPI),mac)
  LIBS := -lloragw -lmpsse -lm
endif

### general build targets

all: libloragw.a test_loragw_spi test_loragw_reg test_loragw_hal test_loragw_gps test_loragw_cal

clean:
	rm -f libloragw.a
	rm -f test_loragw_*
	rm -f $(OBJDIR)/*.o
	rm -f inc/config.h

### transpose library.cfg into a C header file : config.h

inc/config.h: ../VERSION library.cfg
	@echo "*** Checking libloragw library configuration ***"
	@rm -f $@
	#File initialization
	@echo "#ifndef _LORAGW_CONFIGURATION_H" >> $@
	@echo "#define _LORAGW_CONFIGURATION_H" >> $@
	# Release version
	@echo "Release version   : $(LIBLORAGW_VERSION)"
	@echo "	#define LIBLORAGW_VERSION	"\"$(LIBLORAGW_VERSION)\""" >> $@
  # SPI interface
	@echo "SPI interface     : $(CFG_SPI_MSG)"
	@echo "	#define $(CFG_SPI_OPT)	1" >> $@
	# Debug options
	@echo "	#define DEBUG_AUX	$(DEBUG_AUX)" >> $@
	@echo "	#define DEBUG_SPI	$(DEBUG_SPI)" >> $@
	@echo "	#define DEBUG_REG	$(DEBUG_REG)" >> $@
	@echo "	#define DEBUG_HAL	$(DEBUG_HAL)" >> $@
	@echo "	#define DEBUG_GPS	$(DEBUG_GPS)" >> $@
	@echo "	#define DEBUG_GPIO	$(DEBUG_GPIO)" >> $@
	@echo "	#define DEBUG_LBT	$(DEBUG_LBT)" >> $@
  # Platform selection
	@echo "	#include \"$(PLATFORM).h\"" >> $@
	# end of file
	@echo "#endif" >> $@
	@echo "*** Configuration seems ok ***"

### library module target

$(OBJDIR):
	mkdir -p $(OBJDIR)

$(OBJDIR)/%.o: src/%.c $(INCLUDES) inc/config.h | $(OBJDIR)
	$(CC) -c $(CFLAGS) $< -o $@

ifeq ($(CFG_SPI),native)
$(OBJDIR)/loragw_spi.o: src/loragw_spi.native.c inc/loragw_spi.h inc/config.h | $(OBJDIR) 
	$(CC) -c $(CFLAGS) $< -o $@
else ifeq ($(CFG_SPI),ftdi)
$(OBJDIR)/loragw_spi.o: src/loragw_spi.ftdi.c inc/loragw_spi.h inc/config.h | $(OBJDIR)
	$(CC) -c $(CFLAGS) $< -o $@
else ifeq ($(CFG_SPI),mac)
$(OBJDIR)/loragw_spi.o: src/loragw_spi.ftdi.c inc/loragw_spi.h inc/config.h | $(OBJDIR)
	$(CC) -c $(CFLAGS) $< -o $@
endif

$(OBJDIR)/loragw_hal.o: src/loragw_hal.c $(INCLUDES) src/arb_fw.var src/agc_fw.var src/cal_fw.var inc/config.h | $(OBJDIR)
	$(CC) -c $(CFLAGS) $< -o $@

### static library

libloragw.a: $(OBJDIR)/loragw_hal.o $(OBJDIR)/loragw_gps.o $(OBJDIR)/loragw_reg.o $(OBJDIR)/loragw_spi.o $(OBJDIR)/loragw_aux.o $(OBJDIR)/loragw_radio.o $(OBJDIR)/loragw_fpga.o $(OBJDIR)/loragw_lbt.o
	$(AR) rcs $@ $^

### test programs

test_loragw_spi: tst/test_loragw_spi.c libloragw.a
	$(CC) $(CFLAGS) -L. $< -o $@ $(LIBS)

test_loragw_reg: tst/test_loragw_reg.c libloragw.a
	$(CC) $(CFLAGS) -L. $< -o $@ $(LIBS)

test_loragw_hal: tst/test_loragw_hal.c libloragw.a
	$(CC) $(CFLAGS) -L. $< -o $@ $(LIBS)

test_loragw_gps: tst/test_loragw_gps.c libloragw.a
	$(CC) $(CFLAGS) -L. $< -o $@ $(LIBS)

test_loragw_cal: tst/test_loragw_cal.c libloragw.a src/cal_fw.var
	$(CC) $(CFLAGS) -L. $< -o $@ $(LIBS)

### EOF
