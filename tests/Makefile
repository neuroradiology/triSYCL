
## To enable asynchronous kernels
CXXFLAGS += -DTRISYCL_ASYNC=1
# For asynchronous kernels when OpenMP is not enabled
LDLIBS += -lpthread

# Compute the absolute directory name from the location of this Makefile
# so that we can compile from anywhere even if we use make -f
# <this_makefile> ... as used by make check:
triSYCL_DIR=$(abspath $(dir $(lastword $(MAKEFILE_LIST)))/..)

# Use all the .cpp C++ files from the subdirectories that have a 1-1
# mapping with the binaries to generate, that means the binary is made
# from only one compilation unit (1 source file)
TARGETS = $(basename $(wildcard */*.cpp))

# There are also .cc C++ files that are used to test binaries made from
# multiple compilation units. There should be specific dependencies added
# later to deal with other compilation units to be linked to the .cc main
# executable. To avoid messing up with LLVM/LIT, the other compilation
# units have .C extensions
TARGETS_CC = $(basename $(wildcard */*.cc))
TARGETS += $(TARGETS_CC)

# Since the .C files generates some .o, think to remove them
FILES_C =$(wildcard */*.C)
CLEANING_TARGETS = $(TARGETS) $(FILES_C:%.C=%.o)

# The implementation uses C++14 and OpenMP
CXXFLAGS += -Wall -std=c++1y -I$(triSYCL_DIR)/include \
	-I$(triSYCL_DIR)/tests/common

# To use OpenMP to execute SYCL kernels:
#CXXFLAGS += -fopenmp

# To use OpenCL
#CXXFLAGS += -DTRISYCL_OPENCL
#LDLIBS += -lOpenCL

# Specify where OpenCL includes files are with OpenCL_INCPATH
ifdef OpenCL_INCPATH
	CXXFLAGS += -I$(OpenCL_INCPATH)
endif

# Specify where Bost.Compute is with BOOST_COMPUTE_INCPATH
ifdef BOOST_COMPUTE_INCPATH
	CXXFLAGS += -I$(BOOST_COMPUTE_INCPATH) \
	-DBOOST_COMPUTE_DEBUG_KERNEL_COMPILATION \
	-DBOOST_COMPUTE_HAVE_THREAD_LOCAL \
	-DBOOST_COMPUTE_THREAD_SAFE
endif

# Work-around an old Boost.CircularBuffer bug if a pre 1.62 Boost
# version is used
CXXFLAGS += -DMAKE_BOOST_CIRCULARBUFFER_THREAD_SAFE

# Specify where OpenCL library files are with OpenCL_LIBPATH
ifdef OpenCL_LIBPATH
  LDFLAGS += -L$(OpenCL_LIBPATH)
endif

# There are some tests using small array display() method
# To enable debug and tracing:
CXXFLAGS += -g
#CXXFLAGS += -g -DTRISYCL_TRACE_KERNEL -DBOOST_LOG_DYN_LINK
#CXXFLAGS += -g -DTRISYCL_DEBUG -DBOOST_LOG_DYN_LINK
#CXXFLAGS += -g -DTRISYCL_DEBUG -DTRISYCL_DEBUG_STRUCTORS -DBOOST_LOG_DYN_LINK
#LDLIBS+=-lboost_log -lpthread
#CXXFLAGS += -fsanitize=address -fno-omit-frame-pointer
#CXXFLAGS += -fsanitize=thread -fno-omit-frame-pointer
#CXXFLAGS += -fsanitize=undefined -fno-omit-frame-pointer

# By default run lit checking in the current directory
CHECKING_DIR ?= .

# Set default compiler instead of default CXX=g++ value

# Assume Clang 3.8 at least
#CXX = clang++
# But everything works fine with GCC 5.4 at least
CXX = g++


# Some hack to force the compiler from the test infrastructure:
ifdef FORCE_CXX
  CXX=$(FORCE_CXX)
endif

all: $(TARGETS)

run: $(TARGETS)
	# Execute each element of TARGETS
	$(foreach command, $(TARGETS), echo; echo $(command):; ./$(command); )

# Useful before a check...
clean:
	$(RM) $(CLEANING_TARGETS)

# Force recompilation of $(TARGETS_CC) binaries through the Makefile since
# LIT does not know the dependencies
check: $(TARGETS_CC)
	# Launch testing with lit tool from LLVM in current directory
	echo Using $(CXX) compiler:
	# lit can be found for example on Debian/Ubuntu in package
	# llvm-3.7-tools in /usr/lib/llvm-3.7/build/utils/lit/lit.py
	# so try before running the check:
	# export TRISYCL_LIT=/usr/lib/llvm-3.7/build/utils/lit/lit.py
	# The config file for triSYCL needs at least Python 3.3
	test "unset$$TRISYCL_LIT == unset" && echo 'Initialize TRISYCL_LIT variable to the path of "lit" command' ; echo 1
	python3 $$TRISYCL_LIT $(LITFLAGS) $(CHECKING_DIR)

check-compilers:
	# Launch the check with various compilers
	-FORCE_CXX=g++-4.9 $(MAKE) clean check
	-FORCE_CXX=clang++-3.8 $(MAKE) clean check
	-FORCE_CXX=g++ $(MAKE) clean check

# A special target to be called as from the test as "make execute
# TARGET=%s" with the right Makefile. There is a short-cut in the lit.cfg
# to use "RUN: %{execute}%s | %{filecheck} %s" in test files instead.

# Add a dependency on the binary name, i.e. without the extension
execute: $(basename $(TARGET))
	# Execute the compiled binary
	$<

# To verify everything is self-contained, run the target in a clone of the
# current branch of the current repository.
#
# Forward any clone-T target into a clone and make T in it
#
# Use for example:
# make clone-check
clone-%:
	rm -rf test-clone
	git clone --branch `git rev-parse --abbrev-ref HEAD` .. test-clone
	cd test-clone/tests; $(MAKE) $*


multiple_compilation_units/parallel_for: multiple_compilation_units/parallel_for_other.o
