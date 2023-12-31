	
	STRIP ?= strip
	# check if user is root
	user = $(shell whoami)
	ifeq ($(user),root)
	INSTALL_DIR = /usr/lib/lv2
	else 
	INSTALL_DIR = ~/.lv2
	endif

	# check CPU and supported optimization flags
	ifneq ($(shell cat /proc/cpuinfo | grep sse3 ) , )
		SSE_CFLAGS = -msse3 -mfpmath=sse
	else ifneq ($(shell cat /proc/cpuinfo | grep sse2 ) , )
		SSE_CFLAGS = -msse2 -mfpmath=sse
	else ifneq ($(shell cat /proc/cpuinfo | grep sse ) , )
		SSE_CFLAGS = -msse -mfpmath=sse
		else ifneq ($(shell cat /proc/cpuinfo | grep ARM ) , )
		ifneq ($(shell cat /proc/cpuinfo | grep ARMv7 ) , )
			ifneq ($(shell cat /proc/cpuinfo | grep vfpd32 ) , )
				SSE_CFLAGS = -march=armv7-a -mfpu=vfpv3 
			else ifneq ($(shell cat /proc/cpuinfo | grep vfpv3 ) , )
				SSE_CFLAGS = -march=armv7-a -mfpu=vfpv3
			endif
		else
			ARMCPU = "YES"
		endif
	else
		SSE_CFLAGS =
	endif

	# check LD version
	ifneq ($(shell xxd --version 2>&1 | head -n 1 | grep xxd),)
		USE_XXD = 1
	else ifneq ($(shell $(LD) --version 2>&1 | head -n 1 | grep LLD),)
		ifneq ($(shell uname -a | grep  x86_64), )
			LDEMULATION := elf_x86_64
		else ifneq ($(shell uname -a | grep amd64), )
			LDEMULATION := elf_x86_64
		else ifneq ($(shell uname -a | grep i386), )
			LDEMULATION := elf_i386
		endif
		USE_LDD = 1
	else ifneq ($(shell gold --version 2>&1 | head -n 1 | grep gold),)
		LD = gold
	endif

	# set bundle name
	NAME = NeuralRecord
	BUNDLE = $(NAME).lv2
	VER = 0.2

	LIB_DIR := ../libxputty/libxputty/
	HEADER_DIR := $(LIB_DIR)include/

	# set compile flags
	CXXFLAGS += -I. -I./dsp  -fPIC -DPIC -O2 -Wall -funroll-loops `pkg-config --cflags sndfile`\
	-ffast-math -fomit-frame-pointer -fstrength-reduce -fdata-sections -Wl,--gc-sections $(SSE_CFLAGS)
	LDFLAGS += -I.  -lm -pthread -shared -Llibrary -lc -lm -lrt -fPIC -DPIC `pkg-config --libs sndfile` 
	GUI_LDFLAGS += -I. -I$(HEADER_DIR) \
	-L. $(LIB_DIR)libxputty.a -shared `pkg-config --static --cflags --libs cairo x11` -lm 
	# invoke build files
	OBJECTS = profiler.cpp 
	GUI_OBJECTS = profiler_x11ui.c
	## output style (bash colours)
	BLUE = "\033[1;34m"
	RED =  "\033[1;31m"
	NONE = "\033[0m"

.PHONY : mod modnogui all clean install uninstall 

all : check $(NAME)
	@mkdir -p ../$(BUNDLE)/resources
	@cp ./*.wav ../$(BUNDLE)/resources
	@cp ./*.ttl ../$(BUNDLE)
	@mv ./*.so ../$(BUNDLE)
	@if [ -f ../$(BUNDLE)/$(NAME).so ]; then echo $(BLUE)"build finish, now run make install"; \
	else echo $(RED)"sorry, build failed"; fi
	@echo $(NONE)

mod : nogui clean
	@mkdir -p ../$(BUNDLE)/resources
	@cp ./*.wav ../$(BUNDLE)/resources
	@cp -R ./MOD/* ../$(BUNDLE)
	@mv ./$(NAME).so ../$(BUNDLE)
	@if [ -f ../$(BUNDLE)/$(NAME).so ]; then echo $(BLUE)"build finish, now run make install"; \
	else echo $(RED)"sorry, build failed"; fi
	@echo $(NONE)

modnogui : nogui clean
	@mkdir -p ../$(BUNDLE)/resources
	@cp ./*.wav ../$(BUNDLE)/resources
	@cp ./MOD/$(NAME).ttl ../$(BUNDLE)
	@cp ./manifest.ttl ../$(BUNDLE)
	@mv ./$(NAME).so ../$(BUNDLE)
	@if [ -f ../$(BUNDLE)/$(NAME).so ]; then echo $(BLUE)"build finish, now run make install"; \
	else echo $(RED)"sorry, build failed"; fi
	@echo $(NONE)

check :
ifdef ARMCPU
	@echo $(RED)ARM CPU DEDECTED, please check the optimization flags
	@echo $(NONE)
endif

clean :
	rm -f *.a *.o *.so
	@rm -f $(NAME).so
	@rm -rf ../$(BUNDLE)
	@echo ". ." $(BLUE)", clean up"$(NONE)

install :
ifneq ("$(wildcard ../$(BUNDLE))","")
	@mkdir -p $(DESTDIR)$(INSTALL_DIR)/$(BUNDLE)
	cp -r ../$(BUNDLE)/* $(DESTDIR)$(INSTALL_DIR)/$(BUNDLE)
	@echo ". ." $(BLUE)", done"$(NONE)
else
	@echo ". ." $(BLUE)", you must build first"$(NONE)
endif

uninstall :
	@rm -rf $(INSTALL_DIR)/$(BUNDLE)
	@echo ". ." $(BLUE)", done"$(NONE)

$(NAME) : clean 
	$(CXX) $(CXXFLAGS) $(OBJECTS) $(LDFLAGS) -o $(NAME).so
	$(CC) $(CXXFLAGS) -Wl,-z,nodelete $(GUI_OBJECTS) -L. $(RESOURCES_LIB) $(GUI_LDFLAGS) -o $(NAME)_ui.so
	$(STRIP) -s -x -X -R .note.ABI-tag $(NAME).so
	$(STRIP) -s -x -X -R .note.ABI-tag $(NAME)_ui.so

nogui : clean
	$(CXX) $(CXXFLAGS) $(OBJECTS) $(LDFLAGS) -o $(NAME).so
	$(STRIP) -s -x -X -R .note.ABI-tag $(NAME).so

doc:
	#pass
