CROSSPREFIX = c:\Users\administrator\Desktop\xpack-riscv-none-elf-gcc-11.3.0-1\bin\riscv-none-elf-

CC		= $(CROSSPREFIX)gcc.exe
#AS		= $(CROSSPREFIX)as.exe
LD 		= $(CROSSPREFIX)ld.exe
OBJCOPY	= $(CROSSPREFIX)objcopy.exe
OBJDUMP	= $(CROSSPREFIX)objdump.exe
SIZE	= $(CROSSPREFIX)size.exe

INCDIR	:= ..\SRC\StdPeriphDriver\inc -I..\SRC\RVMSIS -I.\FreeRTOS\include -I.\FreeRTOS\portable -I.\FreeRTOS\portable\GCC\RISC-V
SRCDIR  := ..\SRC\StdPeriphDriver

# RISC-V ISA updates
# Compared to previous releases, starting from 12.x, the compiler implements the new RISC-V ISA, which introduces an incompatibility issue, and builds might throw error messages like unrecognized opcode csrr.
# The reason is that csr read/write (csrr*/csrw*) instructions and fence.i instruction were separated from the I extension, becoming two standalone extensions: Zicsr and Zifencei.
# The solution is to add _zicsr and/or _zifencei to the -march option, e.g. -march=rv32imac becomes -march=rv32imac_zicsr_zifencei.
# newlib-nano : Support for newlib-nano is available using the --specs=nano.specs option. For better results, this option must be added to both compile and link time.
# nosys.specs : If no syscalls are needed, --specs=nosys.specs can be used at link time to provide empty implementations for the POSIX system calls.
# -mcmodel=medany : The libraries are compiled with -O2 -mcmodel=medany. The nano version is compiled with -Os -mcmodel=medany

DFLAGS      := -march=rv32imac_zicsr -mabi=ilp32 -mcmodel=medany \
               -msmall-data-limit=8 -fmessage-length=0 -fsigned-char -ffunction-sections -fdata-sections -fno-common
CPPFLAGS	:= $(DFLAGS) -I$(INCDIR)
CFLAGS		:=  --specs=nano.specs -Os -DDEBUG=Debug_UART1 -DINT_SOFT=1
SFLAGS		:= $(DFLAGS) -x assembler-with-cpp
LFLAGS		:= $(DFLAGS) --specs=nano.specs --specs=nosys.specs -nostartfiles

objs := .\build\Main.o \
        .\build\CH58x_sys.o .\build\CH58x_gpio.o .\build\CH58x_uart1.o .\build\CH58x_pwr.o \
        .\build\startup_CH583.o \
		.\build\tasks.o .\build\list.o .\build\queue.o .\build\timers.o \
		.\build\heap_4.o .\build\port.o .\build\portASM.o

app.bin: $(objs)
	$(CC) $(LFLAGS) -T.\Ld\Link.ld -Xlinker --print-memory-usage -Wl,-Map,"app.map" -o app.elf $^ $(SRCDIR)\libISP583.a
	$(OBJCOPY) -O binary app.elf app.bin
	$(OBJDUMP) -D app.elf > app.dis
	$(SIZE) app.elf

# StdPeriph
.\build\\%.o: $(SRCDIR)\%.c
	$(CC) $(CPPFLAGS) $(CFLAGS) -o $@ -c $<

#FreeRTOS
.\build\\%.o: .\FreeRTOS\%.c
	$(CC) $(CPPFLAGS) $(CFLAGS) -o $@ -c $<

#Exceptions (Main/Startup/port)	
.\build\Main.o : .\APP\main.c
	$(CC) $(CPPFLAGS) $(CFLAGS) -o $@ -c $<
	
.\build\startup_CH583.o : .\Startup\startup_CH583.S
	$(CC) $(SFLAGS) -o $@ -c $<

.\build\heap_4.o : .\FreeRTOS\portable\MemMang\heap_4.c
	$(CC) $(CPPFLAGS) $(CFLAGS) -o $@ -c $<

.\build\port.o: .\FreeRTOS\portable\GCC\RISC-V\port.c
	$(CC) $(CPPFLAGS) $(CFLAGS) -o $@ -c $<

.\build\portASM.o: .\FreeRTOS\portable\GCC\RISC-V\portASM.S
	$(CC) $(CPPFLAGS) $(SFLAGS) -o $@ -c $<

clean:
#rm -f build\*.o app.*
	del /Q build\*.o
	del /Q app*
	
