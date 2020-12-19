# Segmentation

This took me more than I thought was necassary to undersrand this concept. it seems all descriptions assume certain knowledge and go on to describe.

So what is segmenation? CPU in real mode has address bus of 16 bits. so that maximum address that CPU can refer to us in binary 0000000000000000 to 1111111111111111 or in hex from 0x0000 to 0xFFFF. that is 65,536 or 64KB or memory. Now what if we want to access more than 64KB of memory?

It turns our a method was intenved to access 1MB of memory, it was assumed that 1Meg of memeory would be more than succifient.

So how how much is 1Meg is 2^20 that is the highest memory that we can access is 1048576, how do we can access this with 16bit address bus?

Enter segment registers. a new set of registers were addedd to architecture which hold what is known as segment registers these are called CS, DS, ES <update list here>

now the memory which is whatever size we do not care is divided into segments which corresponds to segments of program. that is if you have seen a c program, it is divied into .text which is code segment, .data which is data segment etc

now ds will hold memory of data segment in memory which is of 16Bits say 0x1234 now as a programmer once you setup ds, you only provide what is known as offset address to access memory into this segment and this offset address is also 16bits say 0x1256. but the actual address is calculated by this formula

(segment address \* 16) + offset address

the \* by 16 can also be achived by left shift by 4

so if we to take our segment 0x1234 and shift left by 4, we would get 0x12340 now add offset address to this and we get 0x13596 and this is absolute memory location.

by this logic lets compute the highest memory that can be refrenced

0xFFFF shift left4 is 0xFFFF0+0xFFFF = 0x10FFEF or 1114095 in decimal but in reality this was capped at 0xFFEF

 the CPU automatically calculates the absolute address as the appropriate segment’s start address offseted by our specified address [?]. By appropriate segment, I mean that, unless explicitly told otherwise, the CPU will offset our address from the segment register appropriate for the context of our instruction, for example: the address used in the instruction mov ax, [0x45ef] would by default be offset from the data segment, indexed by ds; 


Refrence: https://thestarman.pcministry.com/asm/debug/Segments.html
