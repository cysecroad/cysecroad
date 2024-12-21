---
title: "My simple c++ makefile "
date: 2024-12-21T21:00:47+01:00
draft: false
ShowToc: true

section: "guides"
url: "/guides/makefile"

tags: [makefile, c++, cpp]
categories: [makefile, programming, c++, cpp, debugging, building, make]
keywords: [makefile, programming, c++, cpp, debugging, building, make]

params:
  author: "El Viajero"

summary: " "
  
cover:
    image: "guide-make.png"
    alt: "Guide image"
    relative: false
    hidden: false
    linkFullImages: true
    responsiveImages: false
editPost:
  URL: "mailto:cysecroad@gmail.com"
  Text: "Suggest Changes" # edit text
  appendFilePath: true # to append file path to Edit link
---

# My simple c++ makefile 

¡Hola tech enthusiasts! Remember that time when you had to type the same 15 commands to compile your project, and then your colleague asked, "Why don't you just use a Makefile?" *¡Qué vergüenza!* (How embarrassing!) Well, let's fix that today.

## Why Makefiles are like Having a Good Assistant

You know what's funny about being a developer? We spend hours automating tasks that would take minutes to do manually, just to save seconds in the future. ¡Pero espera! (But wait!) Sometimes this actually makes sense, and Makefiles are the perfect example.\
Think of a Makefile as your project's personal mayordomo (butler).\
Instead of remembering and typing:

```bash
gcc -Wall -Wextra -o myprogram main.c utils.c
./myprogram
rm myprogram
```

```bash
make run
```

## Let's create our butler

Here's a not so simple Makefile, that keeps track of our steps (or goes *muy rapido* if we want it to):

```makefile
# Main compiler
CXX := g++ 

# Define base flags for preprocessor definitions
DEFINES := 

# Variable that can be defined externally
# Used to add additional preprocessor definitions
def ?=

# Debug mode flag with default value of 0 (off)
# Can be enabled by passing dbg=1 when running make
dbg ?= 0

#**************************************************************************************************
# Setting up basic directory structure.
# Root directory for all source files
SRCDIR := src

# Root directory for all build artifacts
BUILDDIR := build

# Directory for preprocessed files (.i extension)
PPDIR := $(BUILDDIR)/preproc
# Directory for assembly files (.asm extension)
ASMDIR := $(BUILDDIR)/asm

# Directory for object files (.o extension)
OBJDIR := $(BUILDDIR)/obj

# Target executable configuration
# If 'path' is specified, uses that path for output
# Otherwise defaults to bin/runner
ifdef path
    OUT_DIR := $(dir $(path))
    FILE_NAME := $(notdir $(basename $(path)))
    TARGET := $(OUT_DIR)$(FILE_NAME)
else
    TARGET := bin/runner
endif

# File extensions
SRCEXT := cpp

ASMEXT := asm    # For NASM assembly files
SEXT := s        # For GNU assembly files
#**************************************************************************************************

#**************************************************************************************************
# Source file discovery
# Automatically finds all .cpp files in the src directory. 
# Shell function executed find command to locate all files(-type f) with the .cpp extension
SOURCES := $(shell find $(SRCDIR) -type f -name *.$(SRCEXT))
#**************************************************************************************************
# Find all source files of different types
ASMSOURCES := $(shell find $(SRCDIR) -type f -name *.$(ASMEXT))
GASSOURCES := $(shell find $(SRCDIR) -type f -name *.$(SEXT))
#**************************************************************************************************

#**************************************************************************************************
# Maps source files to their preprocessed equivalents
# Changes directory prefix from src to build/preproc and extension from .cpp to .i
PREPROCESSED := $(patsubst $(SRCDIR)/%,$(PPDIR)/%,$(SOURCES:.$(SRCEXT)=.i))
#**************************************************************************************************
#**************************************************************************************************

#**************************************************************************************************
# Maps source files to their assembly equivalents
# Changes directory prefix from src to build/asm and extension from .cpp to .s
GENASM := $(patsubst $(SRCDIR)/%,$(ASMDIR)/%,$(SOURCES:.$(SRCEXT)=.s))
#**************************************************************************************************

#**************************************************************************************************
# Generate corresponding object file paths
CPPOBJECTS := $(patsubst $(SRCDIR)/%,$(OBJDIR)/%,$(SOURCES:.$(SRCEXT)=.o))
ASMOBJECTS := $(patsubst $(SRCDIR)/%,$(OBJDIR)/%,$(ASMSOURCES:.$(ASMEXT)=.o))
GASOBJECTS := $(patsubst $(SRCDIR)/%,$(OBJDIR)/%,$(GASSOURCES:.$(SEXT)=.o))
#**************************************************************************************************
# Objects line
# Transforms source paths to object file paths
# 1. Changes .cpp to .o using the `.$(SRCEXT)=.0 substitution
# 2. Changes the directory from src to build using patsubst
OBJECTS := $(CPPOBJECTS) $(ASMOBJECTS) $(GASOBJECTS)
#**************************************************************************************************

#**************************************************************************************************
# Compiler configuration
# Debug mode configuration
# If debug mode is enabled (dbg=1), adds _STRING_DBG_ definition
ifeq ($(dbg),1)
    DEFINES += _STRING_DBG_
endif

# Adds any additional definitions specified via 'def'
DEFINES += $(def)

# Converts DEFINES into compiler flags
# Each definition is prefixed with -D
DEFINE_FLAGS := $(patsubst %,-D%,$(DEFINES))

# Enhanced compiler flags for debugging
# -g3: Maximum debug information
# -O0: No optimization for better debugging
# -ggdb3: Generate debug info in GDB's preferred format
DEBUG_FLAGS := -g3 -O0 -ggdb3 -fno-inline -Wall -Wextra

# Assembler flags for different assembly formats
# NASM flags for .asm files
NASM_FLAGS := -f elf64           # 64-bit output format
ifeq ($(dbg),1)
    NASM_FLAGS += -g dwarf      # Debug symbols for NASM
endif

# GNU AS flags for .s files
GAS_FLAGS := --64                # 64-bit mode
ifeq ($(dbg),1)
    GAS_FLAGS += --gdwarf       # Debug symbols for GNU AS
endif

# Compiler flags:
# -g: Enables debugging symbols
# $(DEFINE_FLAGS): Includes all preprocessor definitions
# -Wall: Enables all warning messages
CFLAGS := -g $(DEFINE_FLAGS) -Wall

# Specifies required libraries
# Links against pthread library
Lib := -pthread

# Adds the include directory to the compiler's search path
INC := -I include
#**************************************************************************************************

# Add this section right after your main variable definitions, before the build rules

#**************************************************************************************************
# Help Configuration
# Colors for help text
HELP_YELLOW := \033[33m
HELP_BLUE := \033[34m
HELP_BOLD := \033[1m
HELP_RESET := \033[0m

# Help target
help:
	@echo "$(HELP_BOLD)Available Targets:$(HELP_RESET)"
	@echo ""
	@echo "$(HELP_BLUE)Main Compilation Targets:$(HELP_RESET)"
	@echo "  make                    - Same as 'make all'"
	@echo "  make all                - Clean, build with all intermediate files, and run"
	@echo "  make run                - Same as 'make all'"
	@echo "  make quick              - Fast compilation without intermediate files"
	@echo ""
	@echo "$(HELP_BLUE)Debug Targets:$(HELP_RESET)"
	@echo "  make debug              - Build with debug symbols and launch GDB"
	@echo "  make debug-build        - Build with debug symbols without launching GDB"
	@echo "  make debug_gui          - Build with debug symbols for VSCode debugging"
	@echo ""
	@echo "$(HELP_BLUE)Cleaning Targets:$(HELP_RESET)"
	@echo "  make clean              - Remove build directory and executable"
	@echo "  make xclean             - Extended clean with interactive confirmation"
	@echo ""
	@echo "$(HELP_BLUE)Available Options:$(HELP_RESET)"
	@echo "  path=<file>            - Specify a single file to compile"
	@echo "                           Example: make quick path=./src/main.cpp"
	@echo ""
	@echo "  dbg=1                  - Enable debug mode"
	@echo "                           - Adds _STRING_DBG_ definition"
	@echo "                           - Adds debug symbols to assembly"
	@echo ""
	@echo "  def=<DEFINE>          - Add custom preprocessor definitions"
	@echo "                           Example: make all def=MY_CUSTOM_FLAG"
	@echo ""
	@echo "$(HELP_BLUE)File Types Supported:$(HELP_RESET)"
	@echo "  .cpp                   - C++ source files"
	@echo "  .asm                   - NASM assembly files"
	@echo "  .s                     - GNU AS assembly files"
	@echo ""
	@echo "$(HELP_BLUE)Build Directories:$(HELP_RESET)"
	@echo "  src/                   - Source files directory"
	@echo "  build/preproc/         - Preprocessed files (.i)"
	@echo "  build/asm/             - Assembly files (.s)"
	@echo "  build/obj/             - Object files (.o)"
	@echo "  bin/                   - Output executables"
	@echo ""
	@echo "$(HELP_YELLOW)Build Process Steps:$(HELP_RESET)"
	@echo "  1. Preprocessing       - Expands macros and includes (.cpp → .i)"
	@echo "  2. Assembly Generation - Creates assembly code (.i → .s)"
	@echo "  3. Object Compilation  - Compiles to object files (.s → .o)"
	@echo "  4. Linking            - Creates final executable"
	@echo ""
	@echo "$(HELP_YELLOW)Examples:$(HELP_RESET)"
	@echo "  make                   # Full build with intermediate files"
	@echo "  make quick            # Quick build for rapid testing"
	@echo "  make debug dbg=1      # Debug build with GDB and debug symbols"
	@echo "  make all def=DEBUG    # Build with DEBUG defined"
	@echo ""
	@echo "$(HELP_YELLOW)Notes:$(HELP_RESET)"
	@echo "  - 'make quick' skips intermediate file generation for faster builds"
	@echo "  - Debug symbols are automatically added in debug mode"
	@echo "  - Assembly files can be mixed with C++ sources freely"
	@echo "  - All intermediate files are preserved for inspection"
#**************************************************************************************************

#**************************************************************************************************
# Build rules

# -----------------------------------------------
# Default target - shows all compilation stages
all: run
	@echo "All compilation stages completed!"
# -----------------------------------------------

# -----------------------------------------------
# Complete build process:
# 1. Cleans previous build
# 2. Runs preprocessing
# 3. Generates assembly
# 4. Creates object files
# 5. Links final executable
# 6. Runs the program
run: clean $(PREPROCESSED) $(GENASM) $(OBJECTS)  $(TARGET)
	@echo "Running the program..."
	@./$(TARGET)
# -----------------------------------------------

# -----------------------------------------------
# Quick compilation without intermediate files
# Can be run as 'make quick' or 'make quick path=./spike/spike.cpp'
quick: clean
ifdef path
	@echo "Compiling and running from specified path: $(path)";
	@$(CXX) $(CFLAGS) $(INC) $(path) $(LIB) -o $(TARGET)
else
	@echo "Running the programm..."
	@$(CXX) $(CFLAGS) $(INC) $(SOURCES) $(LIB) -o $(TARGET)
endif
	@./$(TARGET)
# -----------------------------------------------


# -----------------------------------------------
# Debug target
# Compiles with debug symbols and launches GDB
debug: CFLAGS = $(DEBUG_FLAGS) $(DEFINE_FLAGS)
debug: clean $(TARGET)
	@echo "Starting GDB debugging session..."
	@gdb --quiet $(TARGET)

# Debug build without launching GDB
debug-build: CFLAGS = $(DEBUG_FLAGS) $(DEFINE_FLAGS)
debug-build: clean $(TARGET)
	@echo "Debug build complete. Run 'gdb $(TARGET)' to start debugging."

# VSCode debug target
# Compiles with debug symbols for VSCode debugging
debug_gui: CFLAGS = $(DEBUG_FLAGS) $(DEFINE_FLAGS)
debug_gui: clean $(TARGET)
	@echo "Debug build complete."
	@echo "Now you can start debugging in VSCode using F5 or the Run and Debug panel"
# -----------------------------------------------


# -----------------------------------------------
# Clean target
# Basic clean - removes build directory and target executable
clean:
	@echo "Cleaning...";
	@echo " $(RM) -r $(BUILDDIR) $(TARGET)"; $(RM) -r $(BUILDDIR) $(TARGET)
# -----------------------------------------------

# -----------------------------------------------
# Extended clean - removes additional executable files
# Provides interactive confirmation before deletion
xclean: clean
	@echo "Files to be deleted:"
	@find spike -type f -executable \
		! -name "*.cpp" \
		! -name "*.h" \
		! -name "*.hpp" \
		! -name "*.o" \
		! -name "*.txt" \
		-printf "  %p\n"
	@echo -n "Do you want to proceed with deletion? [y/N] "; \
	read answer; \
	if [ "$$answer" = "y" ] || [ "$$answer" = "Y" ]; then \
		echo "Proceeding with deletion..."; \
		find spike -type f -executable \
			! -name "*.cpp" \
			! -name "*.h" \
			! -name "*.hpp" \
			! -name "*.o" \
			! -name "*.txt" \
			-delete; \
	else \
		echo "Deletion cancelled."; \
	fi
# -----------------------------------------------

#**************************************************************************************************
# Compilation stages

# -----------------------------------------------
# Preprocessing stage (.cpp -> .i)
# Creates preprocessed files with expanded macros and includes
$(PPDIR)/%.i: $(SRCDIR)/%.$(SRCEXT)
	@mkdir -p $(PPDIR)
	@echo "Preprocessing $<"
	@$(CXX) $(CFLAGS) $(INC) -E $< -o $@
# -----------------------------------------------

# -----------------------------------------------
# Assembly generation stage (.i -> .s)
# Creates Intel syntax assembly with verbose annotations
$(ASMDIR)/%.s: $(PPDIR)/%.i
	@mkdir -p $(ASMDIR)
	@echo "Generating assembly for $<"
	@$(CXX) $(CFLAGS) -S -masm=intel -fverbose-asm $< -o $@
# -----------------------------------------------

# -----------------------------------------------
# Object file compilation stage (.s -> .o)
# Creates object files from assembly
$(OBJDIR)/%.o: $(ASMDIR)/%.s
	@mkdir -p $(OBJDIR)
	@echo "Compiling $<"
	@$(CXX) $(CFLAGS) -c $< -o $@
# -----------------------------------------------

# -----------------------------------------------
# For hand-written NASM assembly files
$(OBJDIR)/%.o: $(SRCDIR)/%.$(ASMEXT)
	@mkdir -p $(OBJDIR)
	@echo "Assembling NASM file $<"
	@$(AS) $(NASM_FLAGS) $< -o $@
# -----------------------------------------------

# -----------------------------------------------
# For hand-written GNU AS assembly files
$(OBJDIR)/%.o: $(SRCDIR)/%.$(SEXT)
	@mkdir -p $(OBJDIR)
	@echo "Assembling GNU AS file $<"
	@$(GAS) $(GAS_FLAGS) $< -o $@
# -----------------------------------------------

# -----------------------------------------------
# Final linking stage (.o -> executable)
# Links all object files into final executable
$(TARGET): $(OBJECTS)
	@echo "Linking..."
	@echo " $(CXX) $^ -o $(TARGET) $(LIB)"; $(CXX) $^ -o $(TARGET) $(LIB)
# -----------------------------------------------

#**************************************************************************************************

#**************************************************************************************************
# Tests

#**************************************************************************************************

# Declares targets that don't create files
.PHONY: all clean run quick debug debug-build debug_gui  help

# Prevents automatic deletion of intermediate files
.PRECIOUS: $(ASMDIR)/%.s $(PPDIR)/%.i

#**************************************************************************************************
# General makefile

# := Immediate (Simple) Assignment
# 	Variables are expanded at definition time
# = Recursive Assignment
# 	Variables are expanded at use time
# ?= Conditional Assignment
# 	Only sets if variable is not already set
# += Append Assignment
# 	Adds content to existing variable

# @ makes the commands run silently 	| Only shows the output, not the command
# - (hyphen) - Continues on error 		| -rm -f *.o             # Continues even if no .o files exist
# + (plus) - Forces command execution	| +$(MAKE) -C subdir     # Always executes, even in dry-run

# Automatic variables
# $@ - Target name
# $< - First prerequisite
# $^ - All prerequisites
# $* - Stem (matched by %)									|     %.o: %.cpp \\ $(CXX) -c $*.cpp     # $* is filename without extension
# $? - All prerequisites newer than target	| 		command $?           # Only newer dependencies

# ifeq - Equality condition
# ifdef - Variable defined check
# ifndef - Variable undefined check

# .PHONY - Declares non-file targets
# .PRECIOUS - Keeps intermediate files
# .DEFAULT - Default recipe for unnamed targets

# Create directory if not exists
# 	@mkdir -p $(PPDIR)     # -p creates parent directories too
# Remove directories and files
#		$(RM) -r $(BUILDDIR)   # -r for recursive removal

# Pattern Rule (% wildcard)
# 	$(OBJDIR)/%.o: $(SRCDIR)/%.$(SRCEXT) # Matches any filename

# $(patsubst pattern,replacement,text)
#		OBJECTS := $(patsubst $(SRCDIR)/%,$(OBJDIR)/%,$(SOURCES)) # Transforms source paths to object paths

# $(shell command)
#		SOURCES := $(shell find $(SRCDIR) -type f -name *.$(SRCEXT)) # Executes shell command and captures output

# $(dir path)
# 	OUT_DIR := $(dir $(path)) # Extracts directory part

# $(notdir path)
# 	FILE_NAME := $(notdir $(basename $(path))) # Extracts file name without directory

# $(basename path)
    # Removes file extension
#**************************************************************************************************
```

## Why this works better?

Let me tell you a secret - before I learned about Makefiles, I had a text file called `commands_i_always_forget.txt.` *¡No es broma!* (No joke!) I would literally copy-paste from it every time. If you're doing something similar, mi amigo, it's time for an upgrade.
The beauty of Makefiles is that they're:

- Self-documenting (unlike my old text file)
- Intelligent enough to only recompile what's necessary
- Shareable with your team (goodbye, personal command cheat sheets!)

Remember: A good Makefile is like a good recipe - it should be clear enough that even your future self can understand it six months from now, ¿entiendes? (get it?)