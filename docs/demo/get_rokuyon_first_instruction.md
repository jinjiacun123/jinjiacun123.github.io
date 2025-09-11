```
// compile command:g++ main.cpp -o main
// run: ./main <rom_path>
// File name: main.cpp
#include <stdbool.h>
#include <thread>
#include <cstdint>
#include <string>
#include <string.h>
#include <stdio.h>


//读rokuyon的第一条指令

namespace Core{
	/* variable */
	uint8_t *rom;
	uint32_t romSize;
	std::thread *emuThread;
	
	/* function */
	bool bootRom(const std::string &path);
	void runLoop();
	void start();
}

namespace Memory{
	/* variable */
	uint8_t rspMem[0x2000];  // 4KB RSP DMEM + 4KB RSP IMEM
	
	/* function */
	void reset();
	template <typename T> T read(uint32_t address);
	template <typename T> void write(uint32_t address, T value);
}

namespace CPU{
	/* variable */
	 uint32_t programCounter;
	 uint32_t nextOpcode;
	 
	/* function */
	void reset();
	void runOpcode();
}

namespace PIF{
	/* variable */
	
	/* function */
	void reset();
}


bool Core::bootRom(const std::string &path){
	 // Try to open the specified ROM file
    FILE *romFile = fopen(path.c_str(), "rb");
    if (!romFile) return false;

    // Load the ROM into memory
    if (rom) delete[] rom;
    fseek(romFile, 0, SEEK_END);
    romSize = ftell(romFile);
    fseek(romFile, 0, SEEK_SET);
    rom = new uint8_t[romSize];
    fread(rom, sizeof(uint8_t), romSize, romFile);
    fclose(romFile);
	
	 Memory::reset();
	 CPU::reset();       //[by jim]: main cpu interpreter
	 PIF::reset();

	start();
    return true;
}

void Memory::reset(){
	memset(rspMem, 0, sizeof(rspMem));
}

template uint8_t  Memory::read(uint32_t address);
template uint16_t Memory::read(uint32_t address);
template uint32_t Memory::read(uint32_t address);
template uint64_t Memory::read(uint32_t address);
template <typename T> T Memory::read(uint32_t address){
	uint8_t *data = nullptr;
    uint32_t pAddr = 0x80000000;	//[by jim]:phisical address

	 // Get a physical address from a virtual one
    if ((address & 0xC0000000) == 0x80000000) // kseg0, kseg1
    {
        // Mask the virtual address to get a physical one
        pAddr = address & 0x1FFFFFFF;
    }

	if (pAddr >= 0x4000000 && pAddr < 0x4040000) //[by jim] mapping rsp data memory / instruction memory
    {
        // Read a value from RSP DMEM/IMEM, with wraparound
        T value = 0;
        for (size_t i = 0; i < sizeof(T); i++)
            value |= (T)rspMem[(pAddr & 0x1000) | ((pAddr + i) & 0xFFF)] << ((sizeof(T) - 1 - i) * 8);
        return value;
    }
}


template void Memory::write(uint32_t address, uint8_t  value);
template void Memory::write(uint32_t address, uint16_t value);
template void Memory::write(uint32_t address, uint32_t value);
template void Memory::write(uint32_t address, uint64_t value);
template <typename T> void Memory::write(uint32_t address, T value){
	uint8_t *data = nullptr;
    uint32_t pAddr = 0x80000000;
	
#if 1	// translate from virtual address to physical address
	// Get a physical address from a virtual one
    if ((address & 0xC0000000) == 0x80000000) // kseg0, kseg1
    {
        // Mask the virtual address to get a physical one
        pAddr = address & 0x1FFFFFFF;
    }
#endif	

	if (pAddr >= 0x4000000 && pAddr < 0x4040000)//[by jim]setting rsp imem/dmem
    {
        // Write a value to RSP DMEM/IMEM, with wraparound
        for (size_t i = 0; i < sizeof(T); i++)
            rspMem[(pAddr & 0x1000) | ((pAddr + i) & 0xFFF)] = value >> ((sizeof(T) - 1 - i) * 8);
        return;
    }

}


void CPU::reset()//[by jim] need
{
	programCounter = 0xBFC00000 - 4;	//[by jim] ignore
    nextOpcode = 0;
}

void PIF::reset() //[by jim] need
{
	 for (uint32_t i = 0; i < 0x1000; i++)
            Memory::write(0xA4000000 + i, Core::rom[i]);//[by jim] Memory::write(0xA4000000 + i, Memory::read(0xB0000000 + i))
     CPU::programCounter = 0xA4000040 - 4;
}


void Core::start()//[by jim]:<---------------------------
{
    // Start the threads if emulation wasn't running
    emuThread = new std::thread(runLoop);//[by jim]:<--------------
	printf("finish start child pthread\n");
}

void Core::runLoop()//[by jim] need
{
	 CPU::runOpcode();
}

void CPU::runOpcode()//[by jim] need
{
	// Move an opcode through the pipeline
    // TODO: unaligned address exception
    uint32_t opcode = nextOpcode;
	static uint32_t line = 0;
	printf("%d\t0x%02x:0x%02x\n", line++, programCounter, opcode);
	nextOpcode = Memory::read<uint32_t>(programCounter += 4);//[by jim] init program counter is 0xBFC00000
	opcode = nextOpcode;
	printf("%d\t0x%02x:0x%02x\n", line++, programCounter, opcode);
}


int main(int argc, char *argv[]){
	if (!Core::bootRom(argv[1]))  //[by jim] load rom
    {
        printf("Make sure the ROM file is accessible and try again.");
        return 0;
    }
}

```
