# Author: Ajay Ladsaria
# Example Makefile to compile a C source file into LC-3 object code

# Name of C source file
CODE = fig16.14

# Directory with rcc, cpp, lcc
LCCDIR = $(HOME)/.lc3

# Directory with LC-3 library routines
#LC3LIB = ../lc3lib

all: $(CODE).obj

remake: clean all

# C compiler (lcc) 
$(CODE).obj: $(CODE).c 
	$(LCCDIR)/lcc $(CODE).c -o $(CODE).obj

$(CODE).c:

clean:
	rm -f $(CODE).lcc $(CODE).asm $(CODE).obj $(CODE).sym