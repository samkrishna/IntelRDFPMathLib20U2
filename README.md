# IntelRDFPMathLib20U2

     ==================================================================
     ======= Intel(R) Decimal Floating-Point Math Library v2.2 ========
     ==================================================================

                                                         June 8, 2018


     ******************************************************************
     *** To report issues, please send email to decimalfp@intel.com ***
     ******************************************************************

     Release History:
     ================
        Jul 2009 - Version 1.0 - implemented all the mandatory functions 
                   from the IEEE Standard 754-2008
        Jun 2011 - Version 2.0 - implemented also the functions 
                   recommended in the ISO/IEC Technical Report 24732, 
                   'Extension for the programming language C to support 
                   decimal floating-point arithmetic'
        Aug 2011 - Version 2.0 Update 1 - fixed a small issue in fma128
        Jun 2018 - Version 2.0 Update 2 - fixed issues in powd64, 
                   acos64, and acos128. Added the llround, llquantexp, 
                   and quantum functions, as well as corresponding new tests.
		   This version has been tested in Linux, Windows, OSX, and Solaris.

  1. INTRODUCTION
  ===============

  This package contains the release 2.0 Update 2(or V2.2) of the Intel(R) Decimal
Floating-Point Math Library, conforming to the IEEE Standard 754-2008 for
Floating-Point Arithmetic. This is an extension of Release 2.0 Update 1 (or V2.1) of 2011.

  The library implements the functions defined for decimal floating-point
arithmetic operations in the IEEE Standard 754-2008 for Floating-Point
Arithmetic, which is a revision of the IEEE Standard 754-1985 for Binary
Floating-Point Arithmetic.

The IEEE Standard 754-2008 for Floating-Point Arithmetic supports two
encoding formats: the decimal encoding format, and the binary encoding format.
The Intel(R) Decimal Floating-Point Math Library supports primarily the binary
encoding format for decimal floating-point values, but the decimal encoding
format is supported too in the library, by means of conversion functions
between the two encoding formats.

  Release 2.2 of the library contained in this package implements all
the operations mandated by the IEEE Standard 754-2008. Alternate exception
handling (not a mandatory feature) is not supported currently in the library,
but the design facilitates adding it in the future. It is worth noting that
several useful functions (not part of the IEEE 754-2008 definition) are
provided in LIBRARY/src/bid_round.c. They can be used to round a q-digit decimal
integer number represented in binary to q - x digits (1 <= x <= q - 1).

  For operations involving integer operands or results, the library supports
signed and unsigned 8-, 16-, 32-, and 64-bit integers.

Note: Release 2.0 added transcendental functions (supported in 128-bit, 64-bit, 
and 32-bit decimal formats), including the ones specified in the technical 
report on decimal arithmetic ISO/IEC TR 24732 (available from www.iso.org).
These functions are not correctly rounded, a fact that should be considered 
when using them (similary, binary floating-point mathematical functions 
implemented for example in C or other compiler math libraries are in general
not correctly rounded either). 


  2. PACKAGE CONTENTS
  ===================

  This package contains:

  - eula.txt, a copy of the end user license agreement that applies to
    everything in this package

  - this README FILE

  - the LIBRARY subdirectory with all the source files necessary to build the
    library, and a README file which specifies how to build the library
    in Linux**, HP-UX**, Windows***, and other operating systems; a small set
    of command files (RUNLINUX, RUNWINDOWS, etc.) can be used to build the
    library with different options

  - the TESTS subdirectory with source and input files necessary to build and
    run a reduced set of tests for the library, and a README
    file which specifies how to build and run these tests;
    the test program will print the number of errors detected;
    note that tests involving 80-bit binary floating-point values (these are
    only conversions to and from decimal floating-point formats) are skipped
    if the 80-bit floating-point data type is not supported; a small set
    of command files (RUNLINUX, RUNWINDOWS, etc.) can be used to build and
    run the tests with different options

  - the EXAMPLES subdirectory containing eight examples of calls to library
    functions with various combinations of build options (see Section 8
    below); a README file is included; a small set of command files (RUNLINUX,
    RUNWINDOWS, etc.) can be used to build and run the examples with different
    options

  3. FUNCTION NAMES
  =================

  The function names used in the library are not identical to the names from
the IEEE Standard 754-2008 for Floating-Point Arithmetic. The mapping between
the two sets is given in Section 10 below. The function names can be changed
by editing the #define statements at the beginning of bid_conf.h.


  4. LIBRARY BUILD OPTIONS
  ========================

  The API for the library functions is intended to support constant modes,
global dynamic modes, and scoped dynamic modes. It should be convenient for
compilers with various requirements, even where default modes and no flag
tests can be assumed, as in C99 FENV_ACCESS OFF.

  Three build options are provided, that can be set by editing
LIBRARY/src/bid_conf.h, or (more conveniently) can be set on the compile
command line.

  (a) Function arguments and return values can be passed by reference if
DECIMAL_CALL_BY_REFERENCE is set to 1, or by value otherwise. However, the
floating-point status flags argument is passed by reference even when
DECIMAL_CALL_BY_REFERENCE is 0, unless it is stored in a global variable
(see (c) below).

  (b) The value of the rounding mode can be passed by reference or by value,
or it can be stored in a global variable if DECIMAL_GLOBAL_ROUNDING is
set to 1.
  If DECIMAL_GLOBAL_ROUNDING is set to 1 then the rounding mode is stored
in a global variable that must be declared as

        _IDEC_round __bid_IDEC_glbround;

In this case __bid_IDEC_glbround is a fixed name that *must* be used (but
it can be changed by editing the corresponding #define in bid_conf.h).  Its
initial value should be ROUNDING_TO_NEAREST (or an equivalent value with a name
chosen by the user, as shown in code samples from EXAMPLES where
_IDEC_nearesteven is used). The _IDEC_round type name can be different
(but equivalent), chosen by the user.

  (c) The value of the exception status flags is passed by reference if
they are not represented by a global variable, or it can be stored in a
global variable if DECIMAL_GLOBAL_EXCEPTION_FLAGS is set to 1.
  If DECIMAL_GLOBAL_EXCEPTION_FLAGS is set to 1 then the exception status
flags are stored in a global variable that must be declared as

        _IDEC_flags __bid_IDEC_glbflags;

In this case __bid_IDEC_glbflags is a fixed name that *must* be used but
it can be changed by editing the corresponding #define in bid_conf.h). Its
initial value should be EXACT_STATUS (or an equivalent value with a name chosen
by the user, as shown in code samples from EXAMPLES where _IDEC_allflagsclear
is used). The _IDEC_flags type name can be different (but equivalent),
chosen by the user.

  The three build options supported in this release are selected on the
'make' command line when building the library in LIBRARY and the tests in
TESTS using the makefile-s provided here (so editing bid_conf.h is not
necessary). For example when using the Intel(R) C++ Compiler in Linux**:

    make CC=icc CALL_BY_REF=0 GLOBAL_RND=0 GLOBAL_FLAGS=0 UNCHANGED_BINARY_FLAGS=0

selects parameter passing by value; the rounding mode is a parameter
passed to each function that requires it; the status flags are passed
as a parameter to functions that may modify them (note however that the
status flags represent an exception in that the CALL_BY_REF setting
is ignored - if not global, they are always passed by reference); the
decimal operations may change the inexact binary status flag.

  Another example, when using the Intel(R) C++ Compiler in Windows*** is:

    nmake -fmakefile.mak CC=icl CALL_BY_REF=1 GLOBAL_RND=1 GLOBAL_FLAGS=1 UNCHANGED_BINARY_FLAGS=1

where parameters are passed by reference; the rounding mode is a global
variable; the status flags are stored in a global variable; the
decimal operations will not change the inexact binary status flag.


  5. FUNCTION PROTOTYPES
  ======================

  Function prototypes are provided in LIBRARY/src/bid_functions.h (starting at
line 396 if parameters are passed by reference, and at line 2994 if they
are passed by value).


  6. TESTING
  ==========

  The library was tested on several different platforms (IA-32 Architecture, 
Intel(R) 64, and IA-64 Architecture running Linux**, Windows***, HP-UX**, 
Solaris**, and OSX**).  For example in Linux** icc (Intel(R) C++ Compiler 9.1 
or newer) and gcc** were used. In Windows*** icl (Intel(R) C++ Compiler 9.1 
or newer) and cl (Microsoft*** Visual C++ Compiler) were used. For each of 
these combinations eight combinations of parameter passing method and global 
or local rounding mode and status flags were tested.

  In limited situations, it is possible for incorrect compiler behavior to 
lead to incorrect results in the Intel(r) Decimal Floating-Point Math Library. 
For example, results of round-to-integer functions are incorrect when the 
library is built using gcc 4.2/4.3. Also, some gcc versions in an IA-32 
Linux environment cause slightly incorrect results in a few corner cases for 
the 64-bit decimal square root. (This is not an exhaustive list.) Such errors
should be caught by the tests provided with the library.


  7. USAGE EXAMPLES
  =================

  Eight usage examples are given in the EXAMPLES subdirectory, which illustrate
calls to library functions with various combinations of build options (see
Section 4 above). A README file is included. Note that these examples do not
use any include files from the library: the necessary data types are defined
instead directly in the user-defined decimal.h. However, if the rounding mode
or exception status flags are stored in global variables, then these must have
the fixed names of _IDEC_glbround and _IDEC_glbflags.
  Alternatively - and this is the preferred method, one could use the data
types defined in the library by including two header files (and in this case
DECIMAL_CALL_BY_REFERENCE, DECIMAL_GLOBAL_ROUNDING, and
DECIMAL_GLOBAL_EXCEPTION_FLAGS would have to be properly defined not only when
building the library, but also for the application calling the library
functions):

        #include "../LIBRARY/src/bid_conf.h"
        #include "../LIBRARY/src/bid_functions.h"

Such an example is represented by the readtest program in TESTS/readtest.c.

  8. DECIMAL AND BINARY STATUS FLAGS
  ==================================

  The IEEE Standard 754-2008 specifies distinct rounding modes for binary
and for decimal floating-point operations. However, the floating-point status
flags may be identical for decimal and binary computations. In this
implementation of the decimal arithmetic, the decimal floating-point status
flags are kept separate from the binary flags. Still, it is possible to merge
the two sets (outside the library).
  One issue that needs to be pointed out is that the current implementation of
the decimal floating-point library may set the binary inexact status flag for
certain operations: division, square root, and several other operations where
integers are converted to floating-point values, and the conversion
is inexact. In order to avoid setting the binary inexact flag by decimal
functions, uncomment the following line in bid_conf.h
prior to building the library:
        // #define UNCHANGED_BINARY_STATUS_FLAGS

  9. MAPPING OF IEEE 754-2008 NAMES TO INTEL (R) DECIMAL FLOATING-POINT MATH
                          LIBRARY FUNCTION NAMES
  =========================================================================

Operand and result types are included, where:
        BID64 =  the 64-bit decimal floating-point format using the binary
            encoding; this becomes BID_UINT64 in the library
        BID128 =  the 128-bit decimal floating-point format using the binary
            encoding ; this becomes BID_UINT128 in the library
        binary32 = 32-bit binary floating-point data format
        binary64 = 64-bit binary floating-point data format
        binary80 = 80-bit binary floating-point data format
        binary128 = 128-bit binary floating-point data format
        string = char *
        boolean = int
        enum = int
        _IDEC_flags = int
        _IDEC_round = int

The library function names shown here can be changed by editing #define
statements in bid_conf.h.

================================================================================
IEEE 754-2008 Name                      Opd1   Opd2   Opd3   Result
                                                Intel(R) DFP Math Library Name
================================================================================
roundToIntegralTiesToEven           BID64                BID64
                                           __bid64_round_integral_nearest_even
                                    BID128               BID128
                                           __bid128_round_integral_nearest_even
roundToIntegralTiesToAway           BID64                BID64
                                           __bid64_round_integral_nearest_away
                                    BID128               BID128
                                           __bid128_round_integral_nearest_away
roundToIntegralTiesTowardZero       BID64                BID64
                                           __bid64_round_integral_zero
                                    BID128               BID128
                                           __bid128_round_integral_zero
roundToIntegralTiesTowardPositive   BID64                BID64
                                           __bid64_round_integral_positive
                                    BID128               BID128
                                           __bid128_round_integral_positive
roundToIntegralTiesTowardNegative   BID64                BID64
                                           __bid64_round_integral_negative
                                    BID128               BID128
                                           __bid128_round_integral_negative
roundToIntegralExact                BID64                BID64
                                           __bid64_round_integral_exact
                                    BID128               BID128
                                           __bid128_round_integral_exact
nextUp                              BID64                BID64
                                           __bid64_nextup
                                    BID128               BID128
                                           __bid128_nextup
nextDown                            BID64                BID64
                                           __bid64_nextdown
                                    BID128               BID128
                                           __bid128_nextdown
N/A                                 BID64  BID64         BID64
                                           __bid64_nextafter
                                    BID128 BID128        BID128
                                           __bid128_nextafter
remainder                           BID64  BID64         BID64
                                           __bid64_rem
                                    BID128 BID128        BID128
                                           __bid128_rem
minNum                              BID64  BID64         BID64
                                           __bid64_minnum
                                    BID128 BID128        BID128
                                           __bid128_minnum
maxNum                              BID64  BID64         BID64
                                           __bid64_maxnum
                                    BID128 BID128        BID128
                                           __bid128_maxnum
minNumMag                           BID64  BID64         BID64
                                           __bid64_minnum_mag
                                    BID128 BID128        BID128
                                           __bid128_minnum_mag
maxNumMag                           BID64  BID64         BID64
                                           __bid64_maxnum_mag
                                    BID128 BID128        BID128
                                           __bid128_maxnum_mag
quantize                            BID64  BID64         BID64
                                           __bid64_quantize
                                    BID128 BID128        BID128
                                           __bid128_quantize
logB                                BID64                BID64
                                           __bid64_ilogb
                                    BID128               BID128
                                           __bid128_ilogb
scaleB                              BID64  BID64         BID64
                                           __bid64_scalbn
                                    BID128 BID128        BID128
                                           __bid128_scalbn
addition                            BID64  BID64         BID64
                                           __bid64_add
                                    BID128 BID128        BID128
                                           __bid128_add
subtraction                         BID64  BID64         BID64
                                           __bid64_sub
                                    BID128 BID128        BID128
                                           __bid128_sub
multiplication                      BID64  BID64         BID64
                                           __bid64_mul
                                    BID128 BID128        BID128
                                           __bid128_mul
division                            BID64  BID64         BID64
                                           __bid64_div
                                    BID128 BID128        BID128
                                           __bid128_div
squareRoot                          BID64                BID64
                                           __bid64_sqrt
                                    BID128               BID128
                                           __bid128_sqrt
fusedMultiplyAdd                    BID64  BID64  BID64  BID64
                                           __bid64_fma
                                    BID128 BID128 BID128 BID128
                                           __bid128_fma
convertFromInt                      int32                BID64
                                           __bid64_from_int32
                                    uint32               BID64
                                           __bid64_from_uint32
                                    int64                BID64
                                           __bid64_from_int64
                                    uint64               BID64
                                           __bid64_from_uint64
                                    int32                BID128
                                           __bid128_from_int32
                                    uint32               BID128
                                           __bid128_from_uint32
                                    int64                BID128
                                           __bid128_from_int64
                                    uint64               BID128
                                           __bid128_from_uint64
convertToIntegerTiesToEven          BID64                int32
                                           __bid64_to_int32_rnint
                                    BID64                uint32
                                           __bid64_to_uint32_rnint
                                    BID64                int64
                                           __bid64_to_int64_rnint
                                    BID64                uint64
                                           __bid64_to_uint64_rnint
                                    BID128               int32
                                           __bid128_to_int32_rnint
                                    BID128               uint32
                                           __bid128_to_uint32_rnint
                                    BID128               int64
                                           __bid128_to_int64_rnint
                                    BID128               uint64
                                           __bid128_to_uint64_rnint
convertToIntegerTowardZero          BID64                int32
                                           __bid64_to_int32_int
                                    BID64                uint32
                                           __bid64_to_uint32_int
                                    BID64                int64
                                           __bid64_to_int64_int
                                    BID64                uint64
                                           __bid64_to_uint64_int
                                    BID128               int32
                                           __bid128_to_int32_int
                                    BID128               uint32
                                           __bid128_to_uint32_int
                                    BID128               int64
                                           __bid128_to_int64_int
                                    BID128               uint64
                                           __bid128_to_uint64_int
convertToIntegerTowardPositive      BID64                int32
                                           __bid64_to_int32_ceil
                                    BID64                uint32
                                           __bid64_to_uint32_ceil
                                    BID64                int64
                                           __bid64_to_int64_ceil
                                    BID64                uint64
                                           __bid64_to_uint64_ceil
                                    BID128               int32
                                           __bid128_to_int32_ceil
                                    BID128               uint32
                                           __bid128_to_uint32_ceil
                                    BID128               int64
                                           __bid128_to_int64_ceil
                                    BID128               uint64
                                           __bid128_to_uint64_ceil
convertToIntegerTowardNegative      BID64                int32
                                           __bid64_to_int32_floor
                                    BID64                int32
                                           __bid64_to_uint32_floor
                                    BID64                int64
                                           __bid64_to_int64_floor
                                    BID64                uint64
                                           __bid64_to_uint64_floor
                                    BID128               int32
                                           __bid128_to_int32_floor
                                    BID128               uint32
                                           __bid128_to_uint32_floor
                                    BID128               int64
                                           __bid128_to_int64_floor
                                    BID128               uint64
                                           __bid128_to_uint64_floor
convertToIntegerTiesToAway          BID64                int32
                                           __bid64_to_int32_rninta
                                    BID64                uint32
                                           __bid64_to_uint32_rninta
                                    BID64                int64
                                           __bid64_to_int64_rninta
                                    BID64                uint64
                                           __bid64_to_uint64_rninta
                                    BID128               int32
                                           __bid128_to_int32_rninta
                                    BID128               uint32
                                           __bid128_to_uint32_rninta
                                    BID128               int64
                                           __bid128_to_int64_rninta
                                    BID128               uint64
                                           __bid128_to_uint64_rninta
convertToIntegerExactTiesToEven     BID64                int32
                                           __bid64_to_int32_xrnint
                                    BID64                uint32
                                           __bid64_to_uint32_xrnint
                                    BID64                int64
                                           __bid64_to_int64_xrnint
                                    BID64                uint64
                                           __bid64_to_uint64_xrnint
                                    BID128               int32
                                           __bid128_to_int32_xrnint
                                    BID128               uint32
                                           __bid128_to_uint32_xrnint
                                    BID128               int64
                                           __bid128_to_int64_xrnint
                                    BID128               uint64
                                           __bid128_to_uint64_xrnint
convertToIntegerExactTowardZero     BID64                int32
                                           __bid64_to_int32_xint
                                    BID64                uint32
                                           __bid64_to_uint32_xint
                                    BID64                int64
                                           __bid64_to_int64_xint
                                    BID64                uint64
                                           __bid64_to_uint64_xint
                                    BID128               int32
                                           __bid128_to_int32_xint
                                    BID128               uint32
                                           __bid128_to_uint32_xint
                                    BID128               int64
                                           __bid128_to_int64_xint
                                    BID128               uint64
                                           __bid128_to_uint64_xint
convertToIntegerExactTowardPositive BID64                int32
                                           __bid64_to_int32_xceil
                                    BID64                uint32
                                           __bid64_to_uint32_xceil
                                    BID64                int64
                                           __bid64_to_int64_xceil
                                    BID64                uint64
                                           __bid64_to_uint64_xceil
                                    BID128               int32
                                           __bid128_to_int32_xceil
                                    BID128               uint32
                                           __bid128_to_uint32_xceil
                                    BID128               int64
                                           __bid128_to_int64_xceil
                                    BID128               uint64
                                           __bid128_to_uint64_xceil
convertToIntegerExactTowardNegative BID64                int32
                                           __bid64_to_int32_xfloor
                                    BID64                uint32
                                           __bid64_to_uint32_xfloor
                                    BID64                int64
                                           __bid64_to_int64_xfloor
                                    BID64                uint64
                                           __bid64_to_uint64_xfloor
                                    BID128               int32
                                           __bid128_to_int32_xfloor
                                    BID128               uint32
                                           __bid128_to_uint32_xfloor
                                    BID128               int64
                                           __bid128_to_int64_xfloor
                                    BID128               uint64
                                           __bid128_to_uint64_xfloor
convertToIntegerExactTiesToAway     BID64                int32
                                           __bid64_to_int32_xrninta
                                    BID64                uint32
                                           __bid64_to_uint32_xrninta
                                    BID64                int64
                                           __bid64_to_int64_rninta
                                    BID64                uint64
                                           __bid64_to_uint64_xrninta
                                    BID128               int32
                                           __bid128_to_int32_xrninta
                                    BID128               uint32
                                           __bid128_to_uint32_xrninta
                                    BID128               int64
                                           __bid128_to_int64_xrninta
                                    BID128               uint64
                                           __bid128_to_uint64_xrninta
convert                             BID32                BID64
                                           __bid32_to_bid64
                                    BID32                BID128
                                           __bid32_to_bid128
                                    BID32                bin32
                                           __bid32_to_binary32
                                    BID32                bin64
                                           __bid32_to_binary64
                                    BID32                bin80
                                           __bid32_to_binary80
                                    BID32                bin128
                                           __bid32_to_binary128
                                    BID64                BID32
                                           __bid64_to_bid32
                                    BID64                BID128
                                           __bid64_to_bid128
                                    BID64                bin32
                                           __bid64_to_binary32
                                    BID64                bin64
                                           __bid64_to_binary64
                                    BID64                bin80
                                           __bid64_to_binary80
                                    BID64                bin128
                                           __bid64_to_binary128
                                    BID128               BID32
                                           __bid128_to_bid32
                                    BID128               BID64
                                           __bid128_to_bid64
                                    BID128               bin32
                                           __bid128_to_binary32
                                    BID128               bin64
                                           __bid128_to_binary64
                                    BID128               bin80
                                           __bid128_to_binary80
                                    BID128               bin128
                                           __bid128_to_binary128
                                    bin32                BID32
                                           __binary32_to_bid32
                                    bin32                BID64
                                           __binary32_to_bid64
                                    bin32                BID128
                                           __binary32_to_bid128
                                    bin64                BID32
                                           __binary64_to_bid32
                                    bin64                BID64
                                           __binary64_to_bid64
                                    bin64                BID128
                                           __binary64_to_bid128
                                    bin80                BID32
                                           __binary80_to_bid32
                                    bin80                BID64
                                           __binary80_to_bid64
                                    bin80                BID128
                                           __binary80_to_bid128
                                    bin128               BID32
                                           __binary128_to_bid32
                                    bin128               BID64
                                           __binary128_to_bid64
                                    bin128               BID128
                                           __binary128_to_bid128
convertFromDecimalCharacter         string               BID64
                                           __bid64_from_string
                                    string               BID128
                                           __bid128_from_string
convertToDecimalCharacter           BID64                string
                                           __bid64_to_string
                                    BID128               string
                                           __bid128_to_string
copy                                BID64                BID64
                                           __bid64_copy
                                    BID128               BID128
                                           __bid128_copy
negate                              BID64                BID64
                                           __bid64_negate
                                    BID128               BID128
                                           __bid128_negate
abs                                 BID64                BID64
                                           __bid64_abs
                                    BID128               BID128
                                           __bid128_abs
copySign                            BID64  BID64         BID64
                                           __bid64_copySign
                                    BID128 BID128        BID128
                                           __bid128_copySign
encodeDecimal                       BID32                DPD32
                                           __bid_to_dpd32
                                    BID64                DPD64
                                           __bid_to_dpd64
                                    BID128               DPD128
                                           __bid_to_dpd128
decodeDecimal                       DPD32                BID32
                                           __bid_dpd_to_bid32
                                    DPD64                BID64
                                           __bid_dpd_to_bid64
                                    DPD128               BID128
                                           __bid_dpd_to_bid128
compareQuietEqual                   BID64  BID64         boolean
                                           __bid64_quiet_equal
                                    BID128 BID128        boolean
                                           __bid128_quiet_equal
compareQuietGreater                 BID64  BID64         boolean
                                           __bid64_quiet_greater
                                    BID128 BID128        boolean
                                           __bid128_quiet_greater
compareQuietGreaterEqual            BID64  BID64         boolean
                                           __bid64_quiet_greater_equal
                                    BID128 BID128        boolean
                                           __bid128_quiet_greater_equal
compareQuietGreaterUnordered        BID64  BID64         boolean
                                           __bid64_quiet_greater_unordered
                                    BID128 BID128        boolean
                                           __bid128_quiet_greater_unordered
compareQuietLess                    BID64  BID64         boolean
                                           __bid64_quiet_less
                                    BID128 BID128        boolean
                                           __bid128_quiet_less
compareQuietLessEqual               BID64  BID64         boolean
                                           __bid64_quiet_less_equal
                                    BID128 BID128        boolean
                                           __bid128_quiet_less_equal
compareQuietLessUnordered           BID64  BID64         boolean
                                           __bid64_quiet_less_unordered
                                    BID128 BID128        boolean
                                           __bid128_quiet_less_unordered
compareQuietNotEqual                BID64  BID64         boolean
                                           __bid64_quiet_not_equal
                                    BID128 BID128        boolean
                                           __bid128_quiet_not_equal
compareQuietNotGreater              BID64  BID64         boolean
                                           __bid64_quiet_not_greater
                                    BID128 BID128        boolean
                                           __bid128_quiet_not_greater
compareQuietNotLess                 BID64  BID64         boolean
                                           __bid64_quiet_not_less
                                    BID128 BID128        boolean
                                           __bid128_quiet_not_less
compareQuietOrdered                 BID64  BID64         boolean
                                           __bid64_quiet_ordered
                                    BID128 BID128        boolean
                                           __bid128_quiet_ordered
compareQuietUnordered               BID64  BID64         boolean
                                           __bid64_quiet_unordered
                                    BID128 BID128        boolean
                                           __bid128_quiet_unordered
compareSignalingEqual               BID64  BID64         boolean
                                           __bid64_signaling_equal (not currently implemented)
                                    BID128 BID128        boolean
                                           __bid128_signaling_equal (not currently implemented)
compareSignalingGreater             BID64  BID64         boolean
                                           __bid64_signaling_greater
                                    BID128 BID128        boolean
                                           __bid128_signaling_greater
compareSignalingGreaterEqual        BID64  BID64         boolean
                                           __bid64_signaling_greater_equal
                                    BID128 BID128        boolean
                                           __bid128_signaling_greater_equal
compareSignalingGreaterUnordered    BID64  BID64         boolean
                                           __bid64_signaling_greater_unordered
                                    BID128 BID128        boolean
                                           __bid128_signaling_greater_unordered
compareSignalingLess                BID64  BID64         boolean
                                           __bid64_signaling_less
                                    BID128 BID128        boolean
                                           __bid128_signaling_less
compareSignalingLessEqual           BID64  BID64         boolean
                                           __bid64_signaling_less_equal
                                    BID128 BID128        boolean
                                           __bid128_signaling_less_equal
compareSignalingLessUnordered       BID64  BID64         boolean
                                           __bid64_signaling_less_unordered
                                    BID128 BID128        boolean
                                           __bid128_signaling_less_unordered
compareSignalingNotEqual            BID64  BID64         boolean
                                           __bid64_signaling_not_equal (not currently implemented)
                                    BID128 BID128        boolean
                                           __bid128_signaling_not_equal (not currently implemented)
compareSignalingNotGreater          BID64  BID64         boolean
                                           __bid64_signaling_not_greater
                                    BID128 BID128        boolean
                                           __bid128_signaling_not_greater
compareSignalingNotLess             BID64  BID64         boolean
                                           __bid64_signaling_not_less
                                    BID128 BID128        boolean
                                           __bid128_signaling_not_less
N/A                                 IDEC_flags *IDEC_flags
                                           __bid_signalException
is754version1985                                         int
                                           __bid_is754
is754version2008                                         int
                                           __bid_is754R
isSignMinus                         BID64                boolean
                                           __bid64_isSigned
                                    BID128               boolean
                                           __bid128_isSigned
isNormal                            BID64                boolean
                                           __bid64_isNormal
                                    BID128               boolean
                                           __bid128_isNormal
isFinite                            BID64                boolean
                                           __bid64_isFinite
                                    BID128               boolean
                                           __bid128_isFinite
isZero                              BID64                boolean
                                           __bid64_isZero
                                    BID128               boolean
                                           __bid128_isZero
isSubnormal                         BID64                boolean
                                           __bid64_isSubnormal
                                    BID128               boolean
                                           __bid128_isSubnormal
isInfinite                          BID64                boolean
                                           __bid64_isInf
                                    BID128               boolean
                                           __bid128_isInf
isNaN                               BID64                boolean
                                           __bid64_isNaN
                                    BID128               boolean
                                           __bid128_isNaN
isSignaling                         BID64                boolean
                                           __bid64_isSignaling
                                    BID128               boolean
                                           __bid128_isSignaling
isCanonical                         BID64                boolean
                                           __bid64_isCanonical
                                    BID128               boolean
                                           __bid128_isCanonical
radix                               BID64                boolean
                                           __bid64_radix
                                    BID128               boolean
                                           __bid128_radix
class                               BID64                enum
                                           __bid64_class
                                    BID128               enum
                                           __bid128_class
totalOrder                          BID64  BID64         boolean
                                           __bid64_totalOrder
                                    BID128 BID128        boolean
                                           __bid128_totalOrder
totalOrderMag                       BID64  BID64         boolean
                                           __bid64_totalOrderMag
                                    BID128 BID128        boolean
                                           __bid128_totalOrderMag
sameQuantum                         BID64  BID64         boolean
                                           __bid64_sameQuantum
                                    BID128 BID128        boolean
                                           __bid128_sameQuantum
lowerFlags                          _IDEC_flags
                                           __bid_lowerFlags
testFlags                           _IDEC_flags          boolean
                                           __bid_testFlags
testSavedFlags                      _IDEC_flags _IDEC_flags boolean
                                           __bid_testSavedFlags
restoreFlags                        _IDEC_flags _IDEC_flags
                                           __bid_restoreFlags
saveAllFlags                        _IDEC_flags         IDEC_flags
                                           __bid_saveFlags
getDecimalRoundingDirection                             _IDEC_round
                                           __bid_getDecimalRoundingDirection
setDecimalRoundingDirection         _IDEC_round
                                           __bid_setDecimalRoundingDirection


  10. DESCRIPTION OF THE INTEL(R) DECIMAL FP MATH LIBRARY FUNCTIONS

This section gives brief descriptions of the functions available in the
Intel(R) Decimal Floating-Point Math Library v2.2. The prototypes are
shown assuming all arguments are passed by value; the rounding mode variable is
passed as an argument to each function that requires it; a pointer to a
variable containing the status flags is passed to each function that requires
it; alternate exception handling is not supported. The function prototypes
for other variants allowed for building the library can be determined from
header files bid_functions.h and bid_conf.h, which contain also all the type
definitions used in the following description, as well as the possible values of
the rounding mode variable rnd_mode and the positions of the individual status
flags in the status word *pfpsf.

Notes:
 1. Three decimal floating-point formats are supported, as defined in
    IEEE Standard 754-2008: 32-bit, 64-bit, and 128-bit.
    The data types used in the library for entities in the three formats are
    UINT32, UINT64, and UINT128 which can be mapped externally to types of
    appropriate sizes and alignments but having different names, for example
    _Decimal32, _Decimal64, and _Decimal128.
    The maximum number of decimal digits in the significand of numerical
    values represented in these three formats are:
      P = 7 decimal digits for the 32-bit decimal floating-point format
      P = 16 decimal digits for the 64-bit decimal floating-point format
      P = 34 decimal digits for the 128-bit decimal floating-point format
    The ranges for normal decimal floating-point numbers are (in magnitude):
      1.000000 * 10^(-95) <= x <= 9.999999 * 10^96 for 32-bit format
      1.0...0 * 10^(-383) <= x <= 9.9...9 * 10^384 for 64-bit format
            (15 decimal digits in the fractional part of the significand)
      1.0...0 * 10^(-6143) <= x <= 9.9...9 * 10^6144 for 128-bit format
            (33 decimal digits in the fractional part of the significand)
    The ranges for subnormal decimal floating-point numbers are (in magnitude):
      0.000001 * 10^(-95) <= x <= 0.999999 * 10^(-95) for 32-bit format
      0.0...01 * 10^(-383) <= x <= 0.9...9 * 10^(-383) for 64-bit format
            (15 decimal digits in the fractional part of the significand)
      0.0...01 * 10^(-6144) <= x <= 0.9...9 * 10^(-6144) for 128-bit format
            (33 decimal digits in the fractional part of the significand)
    Operations with decimal floating-point results usually choose one
    representation of the result from among several possible that have the
    same numerical value (constituting a 'cohort'). The chosen representation
    must have the 'preferred exponent' specified in the IEEE Standard 754-2008.
    (For example 1.0 * 10^(-2) + 10.0 * 10^(-3) = 20.0 * 10^(-3), and not
    2.0 * 10^(-2).)
    The encoding methods for decimal floating-point values are not explained
    here. Decimal floating-point values can be encoded using the
    string-to-decimal conversion functions (__bid64_from_string and
    __bid128_from_string), or decoded using the decimal-to-string conversion
    functions (__bid64_to_string and __bid128_to_string).
 2. The acronym 'dpd' or 'DPD' is used to identify the decimal encoding method
    for decimal floating-point values, defined in the IEEE Standard 754-2008.
    The acronym 'bid' or 'BID' is used to identify the binary encoding method
    for decimal floating-point values, defined in the IEEE Standard 754-2008.
 3. The library functions that operate on decimal floating-point values do so on
    values encoded in BID format.
 4. The floating-point status flags for inexact result, underflow, overflow,
    division by zero and invalid operation are denoted by P, U, O, Z, I
    respectively

Note that the function names can be changed by editing #define statements in
bid_conf.h.

===============================================================================
FUNCTION: Convert a 32-bit decimal floating-point value encoded in BID format to
  the same value encoded in DPD format
PROTOTYPE:
  UINT32 __bid_to_dpd32 (
    UINT32 px);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Convert a 64-bit decimal floating-point value encoded in BID format to
  the same value encoded in DPD format
PROTOTYPE:
  UINT64 __bid_to_dpd64 (
    UINT64 px);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Convert a 128-bit decimal floating-point value encoded in BID format
  to the  same value encoded in DPD format
PROTOTYPE:
  UINT128 __bid_to_dpd128 (
    UINT128 px);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Convert a 32-bit decimal floating-point value encoded in DPD format
  to the  same value encoded in BID format
PROTOTYPE:
  UINT32 __bid_dpd_to_bid32 (
    UINT32 px);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Convert a 64-bit decimal floating-point value encoded in DPD format
  to the same value encoded in BID format
PROTOTYPE:
  UINT64 __bid_dpd_to_bid64 (
    UINT64 px);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Convert a 128-bit decimal floating-point value encoded in DPD format
  to the same value encoded in BID format
PROTOTYPE:
  UINT128 __bid_dpd_to_bid128 (
    UINT128 px);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Decimal floating-point addition, UINT64 + UINT64 -> UINT128
PROTOTYPE:
  UINT128 __bid128dd_add (
    UINT64 x, UINT64 y, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Decimal floating-point addition, UINT64 + UINT128 -> UINT128
PROTOTYPE:
  UINT128 __bid128dq_add (
    UINT64 x, UINT128 y, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Decimal floating-point addition, UINT128 + UINT64 -> UINT128
PROTOTYPE:
  UINT128 __bid128qd_add (
    UINT128 x, UINT64 y, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Decimal floating-point addition, UINT128 + UINT128 -> UINT128
PROTOTYPE:
  UINT128 __bid128_add (
    UINT128 x, UINT128 y, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Decimal floating-point subtraction, UINT64 - UINT64 -> UINT128
PROTOTYPE:
  UINT128 __bid128dd_sub (
    UINT64 x, UINT64 y, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Decimal floating-point subtraction, UINT64 - UINT128 -> UINT128
PROTOTYPE:
  UINT128 __bid128dq_sub (
    UINT64 x, UINT128 y, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Decimal floating-point subtraction, UINT128 - UINT64 -> UINT128
PROTOTYPE:
  UINT128 __bid128qd_sub (
    UINT128 x, UINT64 y, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Decimal floating-point subtraction, UINT128 - UINT128 -> UINT128
PROTOTYPE:
  UINT128 __bid128_sub (
    UINT128 x, UINT128 y, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Decimal floating-point multiplication, UINT64 * UINT64 -> UINT128
PROTOTYPE:
  UINT128 __bid128dd_mul (
    UINT64 x, UINT64 y, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Decimal floating-point multiplication, UINT64 * UINT128 -> UINT128
PROTOTYPE:
  UINT128 __bid128dq_mul (
    UINT64 x, UINT128 y, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Decimal floating-point multiplication, UINT128 * UINT64 -> UINT128
PROTOTYPE:
  UINT128 __bid128qd_mul (
    UINT128 x, UINT64 y, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Decimal floating-point multiplication, UINT128 * UINT128 -> UINT128
PROTOTYPE:
  UINT128 __bid128_mul (
    UINT128 x, UINT128 y, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Decimal floating-point division, UINT128 / UINT128 -> UINT128
PROTOTYPE:
  UINT128 __bid128_div (
    UINT128 x, UINT128 y, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Decimal floating-point division, UINT64 / UINT64 -> UINT128
PROTOTYPE:
  UINT128 __bid128dd_div (
    UINT64 x, UINT64 y, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, Z, I

FUNCTION: Decimal floating-point division, UINT64 / UINT128 -> UINT128
PROTOTYPE:
  UINT128 __bid128dq_div (
    UINT64 x, UINT128 y, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, Z, I

FUNCTION: Decimal floating-point division, UINT128 / UINT64 -> UINT128
PROTOTYPE:
  UINT128 __bid128qd_div (
    UINT128 x, UINT64 y, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, Z, I

FUNCTION: Decimal floating-point fused multiply-add,
    UINT128 * UINT128 + UINT128 -> UINT128
PROTOTYPE:
  UINT128 __bid128_fma (
    UINT128 x, UINT128 y, UINT128 z, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Decimal floating-point fused multiply-add,
    UINT64 * UINT64 + UINT64 -> UINT128
PROTOTYPE:
  UINT128 __bid128ddd_fma (
    UINT64 x, UINT64 y, UINT64 z, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Decimal floating-point fused multiply-add,
    UINT64 * UINT64 + UINT128 -> UINT128
PROTOTYPE:
  UINT128 __bid128ddq_fma (
    UINT64 x, UINT64 y, UINT128 z, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Decimal floating-point fused multiply-add,
    UINT64 * UINT128 + UINT64 -> UINT128
PROTOTYPE:
  UINT128 __bid128dqd_fma (
    UINT64 x, UINT128 y, UINT64 z, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Decimal floating-point fused multiply-add,
    UINT64 * UINT128 + UINT128 -> UINT128
PROTOTYPE:
  UINT128 __bid128dqq_fma (
    UINT64 x, UINT128 y, UINT128 z, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Decimal floating-point fused multiply-add,
    UINT128 * UINT64 + UINT64 -> UINT128
PROTOTYPE:
  UINT128 __bid128qdd_fma (
    UINT128 x, UINT64 y, UINT64 z, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Decimal floating-point fused multiply-add,
    UINT128 * UINT64 + UINT128 -> UINT128
PROTOTYPE:
  UINT128 __bid128qdq_fma (
    UINT128 x, UINT64 y, UINT128 z, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Decimal floating-point fused multiply-add,
    UINT128 * UINT128 + UINT64 -> UINT128
PROTOTYPE:
  UINT128 __bid128qqd_fma (
    UINT128 x, UINT128 y, UINT64 z, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Decimal floating-point fused multiply-add,
    UINT64 * UINT64 + UINT64 -> UINT64
PROTOTYPE:
  UINT64 __bid64_fma (
    UINT64 x, UINT64 y, UINT64 z, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Decimal floating-point fused multiply-add,
    UINT64 * UINT64 + UINT128 -> UINT64
PROTOTYPE:
  UINT64 __bid64ddq_fma (
    UINT64 x, UINT64 y, UINT128 z, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Decimal floating-point fused multiply-add,
    UINT64 * UINT128 + UINT64 -> UINT64
PROTOTYPE:
  UINT64 __bid64dqd_fma (
    UINT64 x, UINT128 y, UINT64 z, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Decimal floating-point fused multiply-add,
    UINT64 * UINT128 + UINT128 -> UINT64
PROTOTYPE:
  UINT64 __bid64dqq_fma (
    UINT64 x, UINT128 y, UINT128 z, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Decimal floating-point fused multiply-add,
    UINT128 * UINT64 + UINT64 -> UINT64
PROTOTYPE:
  UINT64 __bid64qdd_fma (
    UINT128 x, UINT64 y, UINT64 z, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Decimal floating-point fused multiply-add,
    UINT128 * UINT64 + UINT128 -> UINT64
PROTOTYPE:
  UINT64 __bid64qdq_fma (
    UINT128 x, UINT64 y, UINT128 z, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Decimal floating-point fused multiply-add,
    UINT128 * UINT128 + UINT64 -> UINT64
PROTOTYPE:
  UINT64 __bid64qqd_fma (
    UINT128 x, UINT128 y, UINT64 z, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Decimal floating-point fused multiply-add,
    UINT128 * UINT128 + UINT128 -> UINT64
PROTOTYPE:
  UINT64 __bid64qqq_fma (
    UINT128 x, UINT128 y, UINT128 z, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Decimal floating-point square root, UINT128 -> UINT128
PROTOTYPE:
  UINT128 __bid128_sqrt (
    UINT128 x, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, I

FUNCTION: Decimal floating-point square root, UINT64 -> UINT128
PROTOTYPE:
  UINT128 __bid128d_sqrt (
    UINT64 x, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, I

FUNCTION: Decimal floating-point addition, UINT64 + UINT64 -> UINT64
PROTOTYPE:
  UINT64 __bid64_add (
    UINT64 x, UINT64 y, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, O , I

FUNCTION: Decimal floating-point addition, UINT64 + UINT128 -> UINT64
PROTOTYPE:
  UINT64 __bid64dq_add (
    UINT64 x, UINT128 y, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Decimal floating-point addition, UINT128 + UINT64 -> UINT64
PROTOTYPE:
  UINT64 __bid64qd_add (
    UINT128 x, UINT64 y, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Decimal floating-point addition, UINT128 + UINT128 -> UINT64
PROTOTYPE:
  UINT64 __bid64qq_add (
    UINT128 x, UINT128 y, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Decimal floating-point subtraction, UINT64 - UINT64 -> UINT64
PROTOTYPE:
  UINT64 __bid64_sub (
    UINT64 x, UINT64 y, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, O, I

FUNCTION: Decimal floating-point subtraction, UINT64 - UINT128 -> UINT64
PROTOTYPE:
  UINT64 __bid64dq_sub (
    UINT64 x, UINT128 y, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Decimal floating-point subtraction, UINT128 - UINT64 -> UINT64
PROTOTYPE:
  UINT64 __bid64qd_sub (
    UINT128 x, UINT64 y, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Decimal floating-point subtraction, UINT128 - UINT128 -> UINT64
PROTOTYPE:
  UINT64 __bid64qq_sub (
    UINT128 x, UINT128 y, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Decimal floating-point multiplication, UINT64 * UINT64 -> UINT64
PROTOTYPE:
  UINT64 __bid64_mul (
    UINT64 x, UINT64 y, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Decimal floating-point multiplication, UINT64 * UINT128 -> UINT64
PROTOTYPE:
  UINT64 __bid64dq_mul (
    UINT64 x, UINT128 y, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Decimal floating-point multiplication, UINT128 * UINT64 -> UINT64
PROTOTYPE:
  UINT64 __bid64qd_mul (
    UINT128 x, UINT64 y, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Decimal floating-point multiplication, UINT128 * UINT128 -> UINT64
PROTOTYPE:
  UINT64 __bid64qq_mul (
    UINT128 x, UINT128 y, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Decimal floating-point division, UINT64 / UINT64 -> UINT64
PROTOTYPE:
  UINT64 __bid64_div (
    UINT64 x, UINT64 y, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, Z, I

FUNCTION: Decimal floating-point division, UINT64 / UINT128 -> UINT64
PROTOTYPE:
  UINT64 __bid64dq_div (
    UINT64 x, UINT128 y, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, Z, I

FUNCTION: Decimal floating-point division, UINT128 / UINT64 -> UINT64
PROTOTYPE:
  UINT64 __bid64qd_div (
    UINT128 x, UINT64 y, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, Z, I

FUNCTION: Decimal floating-point division, UINT128 / UINT128 -> UINT64
PROTOTYPE:
  UINT64 __bid64qq_div (
    UINT128 x, UINT128 y, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, Z, I

FUNCTION: Decimal floating-point square root, UINT64 -> UINT64
PROTOTYPE:
  UINT64 __bid64_sqrt (
    UINT64 x, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, I

FUNCTION: Decimal floating-point square root, UINT128 -> UINT64
PROTOTYPE:
  UINT64 __bid64q_sqrt (
    UINT128 x, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, I

FUNCTION: Convert 128-bit decimal floating-point value to 8-bit signed integer
  in rounding-to-nearest even mode; inexact exceptions not signaled
PROTOTYPE:
  char __bid128_to_int8_rnint (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 128-bit decimal floating-point value to 8-bit signed integer
  in rounding-to-nearest even mode; inexact exceptions signaled
PROTOTYPE:
  char __bid128_to_int8_xrnint (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 128-bit decimal floating-point value to 8-bit signed integer
  in rounding-to-nearest away mode; inexact exceptions not signaled
PROTOTYPE:
  char __bid128_to_int8_rninta (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 128-bit decimal floating-point value to 8-bit signed integer
  in rounding-to-nearest away mode; inexact exceptions signaled
PROTOTYPE:
  char __bid128_to_int8_xrninta (
    UINT128 x, _IDEC_flags *pfpsf);
  FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 128-bit decimal floating-point value to 8-bit signed integer
  in rounding-to-zero mode; inexact exceptions not signaled
PROTOTYPE:
  char __bid128_to_int8_int (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 128-bit decimal floating-point value to 8-bit signed integer
  in rounding-to-zero mode; inexact exceptions signaled
PROTOTYPE:
  char __bid128_to_int8_xint (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 128-bit decimal floating-point value to 8-bit signed integer
  in rounding-down mode; inexact exceptions not signaled
PROTOTYPE:
  char __bid128_to_int8_floor (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 128-bit decimal floating-point value to 8-bit signed integer
  in rounding-down mode; inexact exceptions signaled
PROTOTYPE:
  char __bid128_to_int8_xfloor (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 128-bit decimal floating-point value to 8-bit signed integer
  in rounding-up mode; inexact exceptions not signaled
PROTOTYPE:
  char __bid128_to_int8_ceil (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 128-bit decimal floating-point value to 8-bit signed integer
  in rounding-up mode; inexact exceptions signaled
PROTOTYPE:
  char __bid128_to_int8_xceil (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 128-bit decimal floating-point value to 16-bit signed integer
  in rounding-to-zero mode; inexact exceptions not signaled
PROTOTYPE:
  short __bid128_to_int16_rnint (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 128-bit decimal floating-point value to 16-bit signed integer
  in rounding-to-nearest-even mode; inexact exceptions signaled
PROTOTYPE:
  short __bid128_to_int16_xrnint (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 128-bit decimal floating-point value to 16-bit signed integer
  in rounding-to-nearest-even mode; inexact exceptions not signaled
PROTOTYPE:
  short __bid128_to_int16_rninta (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 128-bit decimal floating-point value to 16-bit signed integer
  in rounding-to-nearest-away; inexact exceptions signaled
PROTOTYPE:
  short __bid128_to_int16_xrninta (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 128-bit decimal floating-point value to 16-bit signed integer
  in rounding-to-zero; inexact exceptions not signaled
PROTOTYPE:
  short __bid128_to_int16_int (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 128-bit decimal floating-point value to 16-bit signed integer
  in rounding-to-zero mode; inexact exceptions signaled
PROTOTYPE:
  short __bid128_to_int16_xint (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 128-bit decimal floating-point value to 16-bit signed integer
  in rounding-down mode; inexact exceptions not signaled
PROTOTYPE:
  short __bid128_to_int16_floor (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 128-bit decimal floating-point value to 16-bit signed integer
  in rounding-down mode; inexact exceptions signaled
PROTOTYPE:
  short __bid128_to_int16_xfloor (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 128-bit decimal floating-point value to 16-bit signed integer
  in rounding-up mode; inexact exceptions not signaled
PROTOTYPE:
  short __bid128_to_int16_ceil (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 128-bit decimal floating-point value to 16-bit signed integer
  in rounding-up mode; inexact exceptions signaled
PROTOTYPE:
  short __bid128_to_int16_xceil (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 128-bit decimal floating-point value to 8-bit signed integer
  in rounding-to-nearest-even mode; inexact exceptions P, signaled
PROTOTYPE:
  unsigned char __bid128_to_uint8_rnint (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 128-bit decimal floating-point value to 8-bit signed integer
  in rounding-to-nearest-even mode; inexact exceptions signaled
PROTOTYPE:
  unsigned char __bid128_to_uint8_xrnint (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 128-bit decimal floating-point value to 8-bit signed integer
  in rounding-to-nearest-away; inexact exceptions not signaled
PROTOTYPE:
  unsigned char __bid128_to_uint8_rninta (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 128-bit decimal floating-point value to 8-bit signed integer
  in rounding-to-nearest-away; inexact exceptions signaled
PROTOTYPE:
  unsigned char __bid128_to_uint8_xrninta (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 128-bit decimal floating-point value to 8-bit signed integer
  in rounding-to-zero; inexact exceptions not signaled
PROTOTYPE:
  unsigned char __bid128_to_uint8_int (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 128-bit decimal floating-point value to 8-bit unsigned integer
  in rounding-to-zero; inexact exceptions signaled
PROTOTYPE:
  unsigned char __bid128_to_uint8_xint (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 128-bit decimal floating-point value to 8-bit unsigned integer
  in rounding-down mode; inexact exceptions not signaled
PROTOTYPE:
  unsigned char __bid128_to_uint8_floor (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 128-bit decimal floating-point value to 8-bit unsigned integer
  in rounding-down mode; inexact exceptions signaled
PROTOTYPE:
  unsigned char __bid128_to_uint8_xfloor (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 128-bit decimal floating-point value to 8-bit unsigned integer
  in rounding-up mode; inexact exceptions not signaled
PROTOTYPE:
  unsigned char __bid128_to_uint8_ceil (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 128-bit decimal floating-point value to 8-bit unsigned integer
  in rounding-up mode; inexact exceptions signaled
PROTOTYPE:
  unsigned char __bid128_to_uint8_xceil (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 128-bit decimal floating-point value to 16-bit unsigned
  integer in rounding-to-nearest-even mode; inexact exceptions not signaled
PROTOTYPE:
  unsigned short __bid128_to_uint16_rnint (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 128-bit decimal floating-point value to 16-bit unsigned
  integer in rounding-to-nearest-even mode; inexact exceptions signaled
PROTOTYPE:
  unsigned short __bid128_to_uint16_xrnint (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 128-bit decimal floating-point value to 16-bit unsigned
  integer in rounding-to-nearest-away; inexact exceptions not signaled
PROTOTYPE:
  unsigned short __bid128_to_uint16_rninta (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 128-bit decimal floating-point value to 16-bit unsigned
  integer in rounding-to-nearest-away; inexact exceptions signaled
PROTOTYPE:
  unsigned short __bid128_to_uint16_xrninta (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 128-bit decimal floating-point value to 16-bit unsigned
  integer in rounding-to-zero; inexact exceptions not signaled
PROTOTYPE:
  unsigned short __bid128_to_uint16_int (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 128-bit decimal floating-point value to 16-bit unsigned
  integer in rounding-to-zero; inexact exceptions signaled
PROTOTYPE:
  unsigned short __bid128_to_uint16_xint (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 128-bit decimal floating-point value to 16-bit unsigned
  integer in rounding-down mode; inexact exceptions not signaled
PROTOTYPE:
  unsigned short __bid128_to_uint16_floor (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 128-bit decimal floating-point value to 16-bit unsigned
  integer in rounding-down mode; inexact exceptions signaled
PROTOTYPE:
  unsigned short __bid128_to_uint16_xfloor (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 128-bit decimal floating-point value to 16-bit unsigned
  integer in rounding-up mode; inexact exceptions not signaled
PROTOTYPE:
  unsigned short __bid128_to_uint16_ceil (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 128-bit decimal floating-point value to 16-bit unsigned
  integer in rounding-up mode; inexact exceptions signaled
PROTOTYPE:
  unsigned short __bid128_to_uint16_xceil (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 128-bit decimal floating-point value to 32-bit signed
  integer in rounding-to-nearest-even mode; inexact exceptions not signaled
PROTOTYPE:
  int __bid128_to_int32_rnint (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 128-bit decimal floating-point value to 32-bit signed integer
  in rounding-to-nearest-even mode; inexact exceptions signaled
PROTOTYPE:
  int __bid128_to_int32_xrnint (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 128-bit decimal floating-point value to 32-bit signed integer
  in rounding-to-nearest-away; inexact exceptions not signaled
PROTOTYPE:
  int __bid128_to_int32_rninta (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 128-bit decimal floating-point value to 32-bit signed integer
  in rounding-to-nearest-away; inexact exceptions signaled
PROTOTYPE:
  int __bid128_to_int32_xrninta (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 128-bit decimal floating-point value to 32-bit signed integer
  in rounding-to-zero; inexact exceptions not signaled
PROTOTYPE:
  int __bid128_to_int32_int (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 128-bit decimal floating-point value to 32-bit signed integer
  in rounding-to-zero; inexact exceptions signaled
PROTOTYPE:
  int __bid128_to_int32_xint (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 128-bit decimal floating-point value to 32-bit signed integer
  in rounding-down mode; inexact exceptions not signaled
PROTOTYPE:
  int __bid128_to_int32_floor (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 128-bit decimal floating-point value to 32-bit signed integer
  in rounding-down mode; inexact exceptions signaled
PROTOTYPE:
  int __bid128_to_int32_xfloor (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 128-bit decimal floating-point value to 32-bit signed integer
  in rounding-up mode; inexact exceptions not signaled
PROTOTYPE:
  int __bid128_to_int32_ceil (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 128-bit decimal floating-point value to 32-bit signed integer
  in rounding-up mode; inexact exceptions signaled
PROTOTYPE:
  int __bid128_to_int32_xceil (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 128-bit decimal floating-point value to 32-bit unsigned
  integer in rounding-to-nearest-even mode; inexact exceptions not signaled
PROTOTYPE:
  unsigned int __bid128_to_uint32_rnint (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 128-bit decimal floating-point value to 32-bit unsigned
  integer in rounding-to-nearest-even mode; inexact exceptions signaled
PROTOTYPE:
  unsigned int __bid128_to_uint32_xrnint (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 128-bit decimal floating-point value to 32-bit unsigned
  integer in rounding-to-nearest-away; inexact exceptions not signaled
PROTOTYPE:
  unsigned int __bid128_to_uint32_rninta (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 128-bit decimal floating-point value to 32-bit unsigned
  integer in rounding-to-nearest-away; inexact exceptions signaled
PROTOTYPE:
  unsigned int __bid128_to_uint32_xrninta (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 128-bit decimal floating-point value to 32-bit unsigned
  integer in rounding-to-zero; inexact exceptions not signaled
PROTOTYPE:
  unsigned int __bid128_to_uint32_int (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 128-bit decimal floating-point value to 32-bit unsigned
  integer in rounding-to-zero; inexact exceptions signaled
PROTOTYPE:
  unsigned int __bid128_to_uint32_xint (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 128-bit decimal floating-point value to 32-bit unsigned
  integer in rounding-down mode; inexact exceptions not signaled
PROTOTYPE:
  unsigned int __bid128_to_uint32_floor (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 128-bit decimal floating-point value to 32-bit unsigned
  integer in rounding-down mode; inexact exceptions signaled
PROTOTYPE:
  unsigned int __bid128_to_uint32_xfloor (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 128-bit decimal floating-point value to 32-bit unsigned
  integer in rounding-up mode; inexact exceptions not signaled
PROTOTYPE:
  unsigned int __bid128_to_uint32_ceil (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 128-bit decimal floating-point value to 32-bit unsigned
  integer in rounding-up mode; inexact exceptions signaled
PROTOTYPE:
  unsigned int __bid128_to_uint32_xceil (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 128-bit decimal floating-point value to 64-bit signed
  integer in rounding-to-nearest-even mode; inexact exceptions not signaled
PROTOTYPE:
  SINT64 __bid128_to_int64_rnint (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 128-bit decimal floating-point value to 64-bit signed
  integer in rounding-to-nearest-even mode; inexact exceptions signaled
PROTOTYPE:
  SINT64 __bid128_to_int64_xrnint (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 128-bit decimal floating-point value to 64-bit signed
  integer in rounding-to-nearest-away; inexact exceptions not signaled
PROTOTYPE:
  SINT64 __bid128_to_int64_rninta (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 128-bit decimal floating-point value to 64-bit signed
  integer in rounding-to-nearest-away; inexact exceptions signaled
PROTOTYPE:
  SINT64 __bid128_to_int64_xrninta (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 128-bit decimal floating-point value to 64-bit signed integer
  in rounding-to-zero; inexact exceptions not signaled
PROTOTYPE:
  SINT64 __bid128_to_int64_int (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 128-bit decimal floating-point value to 64-bit signed integer
  in rounding-to-zero; inexact exceptions signaled
PROTOTYPE:
  SINT64 __bid128_to_int64_xint (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 128-bit decimal floating-point value to 64-bit signed integer
  in rounding-down mode; inexact exceptions not signaled
PROTOTYPE:
  SINT64 __bid128_to_int64_floor (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 128-bit decimal floating-point value to 64-bit signed integer
  in rounding-down mode; inexact exceptions signaled
PROTOTYPE:
  SINT64 __bid128_to_int64_xfloor (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 128-bit decimal floating-point value to 64-bit signed integer
  in rounding-up mode; inexact exceptions not signaled
PROTOTYPE:
  SINT64 __bid128_to_int64_ceil (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 128-bit decimal floating-point value to 64-bit signed integer
  in rounding-up mode; inexact exceptions signaled
PROTOTYPE:
  SINT64 __bid128_to_int64_xceil (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 128-bit decimal floating-point value to 64-bit unsigned
  integer in rounding-to-nearest-even mode; inexact exceptions not signaled
PROTOTYPE:
  UINT64 __bid128_to_uint64_rnint (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 128-bit decimal floating-point value to 64-bit unsigned
  integer in rounding-to-nearest-even mode; inexact exceptions signaled
PROTOTYPE:
  UINT64 __bid128_to_uint64_xrnint (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 128-bit decimal floating-point value to 64-bit unsigned
  integer in rounding-to-nearest-away; inexact exceptions not signaled
PROTOTYPE:
  UINT64 __bid128_to_uint64_rninta (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 128-bit decimal floating-point value to 64-bit unsigned
  integer in rounding-to-nearest-away; inexact exceptions signaled
PROTOTYPE:
  UINT64 __bid128_to_uint64_xrninta (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 128-bit decimal floating-point value to 64-bit unsigned
  integer in rounding-to-zero; inexact exceptions not signaled
PROTOTYPE:
  UINT64 __bid128_to_uint64_int (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 128-bit decimal floating-point value to 64-bit unsigned
  integer in rounding-to-zero; inexact exceptions signaled
PROTOTYPE:
  UINT64 __bid128_to_uint64_xint (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 128-bit decimal floating-point value to 64-bit unsigned
  integer in rounding-down mode; inexact exceptions not signaled
PROTOTYPE:
  UINT64 __bid128_to_uint64_floor (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 128-bit decimal floating-point value to 64-bit unsigned
  integer in rounding-down mode; inexact exceptions signaled
PROTOTYPE:
  UINT64 __bid128_to_uint64_xfloor (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 128-bit decimal floating-point value to 64-bit unsigned
  integer in rounding-up mode; inexact exceptions not signaled
PROTOTYPE:
  UINT64 __bid128_to_uint64_ceil (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 128-bit decimal floating-point value to 64-bit unsigned
  integer in rounding-up mode; inexact exceptions signaled
PROTOTYPE:
  UINT64 __bid128_to_uint64_xceil (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 64-bit decimal floating-point value to 32-bit signed
  integer in rounding-to-nearest-even mode; inexact exceptions not signaled
PROTOTYPE:
  int __bid64_to_int32_rnint (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 64-bit decimal floating-point value to 32-bit signed integer
  in rounding-to-nearest-even mode; inexact exceptions signaled
PROTOTYPE:
  int __bid64_to_int32_xrnint (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 64-bit decimal floating-point value to 32-bit signed integer
  in rounding-to-nearest-away; inexact exceptions not signaled
PROTOTYPE:
  int __bid64_to_int32_rninta (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 64-bit decimal floating-point value to 32-bit signed integer
  in rounding-to-nearest-away; inexact exceptions signaled
PROTOTYPE:
  int __bid64_to_int32_xrninta (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 64-bit decimal floating-point value to 32-bit signed integer
  in rounding-to-zero; inexact exceptions not signaled
PROTOTYPE:
  int __bid64_to_int32_int (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 64-bit decimal floating-point value to 32-bit signed integer
  in rounding-to-zero; inexact exceptions signaled
PROTOTYPE:
  int __bid64_to_int32_xint (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 64-bit decimal floating-point value to 32-bit signed integer
  in rounding-down mode; inexact exceptions not signaled
PROTOTYPE:
  int __bid64_to_int32_floor (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 64-bit decimal floating-point value to 32-bit signed integer
  in rounding-down mode; inexact exceptions signaled
PROTOTYPE:
  int __bid64_to_int32_xfloor (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 64-bit decimal floating-point value to 32-bit signed integer
  in rounding-up mode; inexact exceptions not signaled
PROTOTYPE:
  int __bid64_to_int32_ceil (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 64-bit decimal floating-point value to 32-bit signed integer
  in rounding-up mode; inexact exceptions signaled
PROTOTYPE:
  int __bid64_to_int32_xceil (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 64-bit decimal floating-point value to 8-bit signed integer
  in rounding-to-nearest-even mode; inexact exceptions not signaled
PROTOTYPE:
  char __bid64_to_int8_rnint (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 64-bit decimal floating-point value to 8-bit signed integer
  in rounding-to-nearest-even mode; inexact exceptions signaled
PROTOTYPE:
  char __bid64_to_int8_xrnint (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 64-bit decimal floating-point value to 8-bit signed integer
  in rounding-to-nearest-away; inexact exceptions not signaled
PROTOTYPE:
  char __bid64_to_int8_rninta (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 64-bit decimal floating-point value to 8-bit signed integer
  in rounding-to-nearest-away; inexact exceptions signaled
PROTOTYPE:
  char __bid64_to_int8_xrninta (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 64-bit decimal floating-point value to 8-bit signed integer
  in rounding-to-zero; inexact exceptions not signaled
PROTOTYPE:
  char __bid64_to_int8_int (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 64-bit decimal floating-point value to 8-bit signed integer
  in rounding-to-zero; inexact exceptions signaled
PROTOTYPE:
  char __bid64_to_int8_xint (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 64-bit decimal floating-point value to 8-bit signed integer
  in rounding-down mode; inexact exceptions not signaled
PROTOTYPE:
  char __bid64_to_int8_floor (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 64-bit decimal floating-point value to 8-bit signed integer
  in rounding-down mode; inexact exceptions signaled
PROTOTYPE:
  char __bid64_to_int8_xfloor (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 64-bit decimal floating-point value to 8-bit signed integer
  in rounding-up mode; inexact exceptions not signaled
PROTOTYPE:
  char __bid64_to_int8_ceil (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 64-bit decimal floating-point value to 8-bit signed integer
  in rounding-up mode; inexact exceptions signaled
PROTOTYPE:
  char __bid64_to_int8_xceil (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 64-bit decimal floating-point value to 16-bit signed integer
  in rounding-to-nearest-even mode; inexact exceptions not signaled
PROTOTYPE:
  short __bid64_to_int16_rnint (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 64-bit decimal floating-point value to 16-bit signed integer
  in rounding-to-nearest-even mode; inexact exceptions signaled
PROTOTYPE:
  short __bid64_to_int16_xrnint (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 64-bit decimal floating-point value to 16-bit signed integer
  in rounding-to-nearest-away; inexact exceptions not signaled
PROTOTYPE:
  short __bid64_to_int16_rninta (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 64-bit decimal floating-point value to 16-bit signed integer
  in rounding-to-nearest-away; inexact exceptions signaled
PROTOTYPE:
  short __bid64_to_int16_xrninta (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 64-bit decimal floating-point value to 16-bit signed integer
  in rounding-to-zero; inexact exceptions not signaled
PROTOTYPE:
  short __bid64_to_int16_int (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 64-bit decimal floating-point value to 16-bit signed integer
  in rounding-to-zero; inexact exceptions signaled
PROTOTYPE:
  short __bid64_to_int16_xint (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 64-bit decimal floating-point value to 16-bit signed integer
  in rounding-down mode; inexact exceptions not signaled
PROTOTYPE:
  short __bid64_to_int16_floor (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 64-bit decimal floating-point value to 16-bit signed integer
  in rounding-down mode; inexact exceptions signaled
PROTOTYPE:
  short __bid64_to_int16_xfloor (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 64-bit decimal floating-point value to 16-bit signed integer
  in rounding-up mode; inexact exceptions not signaled
PROTOTYPE:
  short __bid64_to_int16_ceil (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 64-bit decimal floating-point value to 16-bit signed integer
  in rounding-up mode; inexact exceptions signaled
PROTOTYPE:
  short __bid64_to_int16_xceil (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 64-bit decimal floating-point value to 8-bit unsigned integer
  in rounding-to-nearest-even mode; inexact exceptions not signaled
PROTOTYPE:
  unsigned char __bid64_to_uint8_rnint (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 64-bit decimal floating-point value to 8-bit unsigned integer
  in rounding-to-nearest-even mode; inexact exceptions signaled
PROTOTYPE:
  unsigned char __bid64_to_uint8_xrnint (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 64-bit decimal floating-point value to 8-bit unsigned integer
  in rounding-to-nearest-away; inexact exceptions not signaled
PROTOTYPE:
  unsigned char __bid64_to_uint8_rninta (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 64-bit decimal floating-point value to 8-bit unsigned integer
  in rounding-to-nearest-away; inexact exceptions signaled
PROTOTYPE:
  unsigned char __bid64_to_uint8_xrninta (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 64-bit decimal floating-point value to 8-bit unsigned integer
  in rounding-to-zero; inexact exceptions not signaled
PROTOTYPE:
  unsigned char __bid64_to_uint8_int (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 64-bit decimal floating-point value to 8-bit unsigned integer
  in rounding-to-zero; inexact exceptions signaled
PROTOTYPE:
  unsigned char __bid64_to_uint8_xint (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 64-bit decimal floating-point value to 8-bit unsigned integer
  in rounding-down mode; inexact exceptions not signaled
PROTOTYPE:
  unsigned char __bid64_to_uint8_floor (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 64-bit decimal floating-point value to 8-bit unsigned integer
  in rounding-down mode; inexact exceptions signaled
PROTOTYPE:
  unsigned char __bid64_to_uint8_xfloor (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 64-bit decimal floating-point value to 8-bit unsigned integer
  in rounding-up mode; inexact exceptions not signaled
PROTOTYPE:
  unsigned char __bid64_to_uint8_ceil (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 64-bit decimal floating-point value to 8-bit unsigned integer
  in rounding-up mode; inexact exceptions signaled
PROTOTYPE:
  unsigned char __bid64_to_uint8_xceil (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 64-bit decimal floating-point value to 16-bit unsigned integer
  in rounding-to-nearest-even mode; inexact exceptions not signaled
PROTOTYPE:
  unsigned short __bid64_to_uint16_rnint (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 64-bit decimal floating-point value to 16-bit unsigned integer
  in rounding-to-nearest-even mode; inexact exceptions signaled
PROTOTYPE:
  unsigned short __bid64_to_uint16_xrnint (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 64-bit decimal floating-point value to 16-bit unsigned integer
  in rounding-to-nearest-away; inexact exceptions not signaled
PROTOTYPE:
  unsigned short __bid64_to_uint16_rninta (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 64-bit decimal floating-point value to 16-bit unsigned integer
  in rounding-to-nearest-away; inexact exceptions signaled
PROTOTYPE:
  unsigned short __bid64_to_uint16_xrninta (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 64-bit decimal floating-point value to 16-bit unsigned integer
  in rounding-to-zero; inexact exceptions not signaled
PROTOTYPE:
  unsigned short __bid64_to_uint16_int (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 64-bit decimal floating-point value to 16-bit unsigned integer
  in rounding-to-zero; inexact exceptions signaled
PROTOTYPE:
  unsigned short __bid64_to_uint16_xint (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 64-bit decimal floating-point value to 16-bit unsigned integer
  in rounding-down mode; inexact exceptions not signaled
PROTOTYPE:
  unsigned short __bid64_to_uint16_floor (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 64-bit decimal floating-point value to 16-bit unsigned integer
  in rounding-down mode; inexact exceptions signaled
PROTOTYPE:
  unsigned short __bid64_to_uint16_xfloor (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 64-bit decimal floating-point value to 16-bit unsigned integer
  in rounding-up mode; inexact exceptions not signaled
PROTOTYPE:
  unsigned short __bid64_to_uint16_ceil (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 64-bit decimal floating-point value to 16-bit unsigned integer
  in rounding-up mode; inexact exceptions signaled
PROTOTYPE:
  unsigned short __bid64_to_uint16_xceil (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 64-bit decimal floating-point value to 32-bit unsigned integer
  in rounding-to-nearest-even mode; inexact exceptions not signaled
PROTOTYPE:
  unsigned int __bid64_to_uint32_rnint (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 64-bit decimal floating-point value to 32-bit unsigned integer
  in rounding-to-nearest-even mode; inexact exceptions signaled
PROTOTYPE:
  unsigned int __bid64_to_uint32_xrnint (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 64-bit decimal floating-point value to 32-bit unsigned integer
  in rounding-to-nearest-away; inexact exceptions not signaled
PROTOTYPE:
  unsigned int __bid64_to_uint32_rninta (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 64-bit decimal floating-point value to 32-bit unsigned integer
  in rounding-to-nearest-away; inexact exceptions signaled
PROTOTYPE:
  unsigned int __bid64_to_uint32_xrninta (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 64-bit decimal floating-point value to 32-bit unsigned integer
  in rounding-to-zero; inexact exceptions not signaled
PROTOTYPE:
  unsigned int __bid64_to_uint32_int (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 64-bit decimal floating-point value to 32-bit unsigned integer
  in rounding-to-zero; inexact exceptions signaled
PROTOTYPE:
  unsigned int __bid64_to_uint32_xint (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 64-bit decimal floating-point value to 32-bit unsigned integer
  in rounding-down mode; inexact exceptions not signaled
PROTOTYPE:
  unsigned int __bid64_to_uint32_floor (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 64-bit decimal floating-point value to 32-bit unsigned integer
  in rounding-down mode; inexact exceptions signaled
PROTOTYPE:
  unsigned int __bid64_to_uint32_xfloor (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 64-bit decimal floating-point value to 32-bit unsigned integer
  in rounding-up mode; inexact exceptions not signaled
PROTOTYPE:
  unsigned int __bid64_to_uint32_ceil (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 64-bit decimal floating-point value to 32-bit unsigned integer
  in rounding-up mode; inexact exceptions signaled
PROTOTYPE:
  unsigned int __bid64_to_uint32_xceil (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 64-bit decimal floating-point value to 64-bit signed integer
  in rounding-to-nearest-even mode; inexact exceptions not signaled
PROTOTYPE:
  SINT64 __bid64_to_int64_rnint (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 64-bit decimal floating-point value to 64-bit signed integer
  in rounding-to-nearest-even mode; inexact exceptions signaled
PROTOTYPE:
  SINT64 __bid64_to_int64_xrnint (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 64-bit decimal floating-point value to 64-bit signed integer
  in rounding-to-nearest-away; inexact exceptions not signaled
PROTOTYPE:
  SINT64 __bid64_to_int64_rninta (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 64-bit decimal floating-point value to 64-bit signed integer
  in rounding-to-nearest-away; inexact exceptions signaled
PROTOTYPE:
  SINT64 __bid64_to_int64_xrninta (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 64-bit decimal floating-point value to 64-bit signed integer
  in rounding-to-zero; inexact exceptions not signaled
PROTOTYPE:
  SINT64 __bid64_to_int64_int (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 64-bit decimal floating-point value to 64-bit signed integer
  in rounding-to-zero; inexact exceptions signaled
PROTOTYPE:
  SINT64 __bid64_to_int64_xint (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 64-bit decimal floating-point value to 64-bit signed integer
  in rounding-down mode; inexact exceptions not signaled
PROTOTYPE:
  SINT64 __bid64_to_int64_floor (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 64-bit decimal floating-point value to 64-bit signed integer
  in rounding-down mode; inexact exceptions signaled
PROTOTYPE:
  SINT64 __bid64_to_int64_xfloor (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 64-bit decimal floating-point value to 64-bit signed integer
  in rounding-up mode; inexact exceptions not signaled
PROTOTYPE:
  SINT64 __bid64_to_int64_ceil (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 64-bit decimal floating-point value to 64-bit signed integer
  in rounding-up mode; inexact exceptions signaled
PROTOTYPE:
  SINT64 __bid64_to_int64_xceil (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 64-bit decimal floating-point value to 64-bit unsigned integer
  in rounding-to-nearest-even mode; inexact exceptions not signaled
PROTOTYPE:
  UINT64 __bid64_to_uint64_rnint (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 64-bit decimal floating-point value to 64-bit unsigned integer
  in rounding-to-nearest-even mode; inexact exceptions signaled
PROTOTYPE:
  UINT64 __bid64_to_uint64_xrnint (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 64-bit decimal floating-point value to 64-bit unsigned integer
  in rounding-to-nearest-away; inexact exceptions not signaled
PROTOTYPE:
  UINT64 __bid64_to_uint64_rninta (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 64-bit decimal floating-point value to 64-bit unsigned integer
  in rounding-to-nearest-away; inexact exceptions signaled
PROTOTYPE:
  UINT64 __bid64_to_uint64_xrninta (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 64-bit decimal floating-point value to 64-bit unsigned integer
  in rounding-to-zero; inexact exceptions not signaled
PROTOTYPE:
  UINT64 __bid64_to_uint64_int (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 64-bit decimal floating-point value to 64-bit unsigned integer
  in rounding-to-zero mode; inexact exceptions signaled
PROTOTYPE:
  UINT64 __bid64_to_uint64_xint (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 64-bit decimal floating-point value to 64-bit unsigned integer
  in rounding-down mode; inexact exceptions not signaled
PROTOTYPE:
  UINT64 __bid64_to_uint64_floor (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 64-bit decimal floating-point value to 64-bit unsigned integer
  in rounding-down mode; inexact exceptions signaled
PROTOTYPE:
  UINT64 __bid64_to_uint64_xfloor (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 64-bit decimal floating-point value to 64-bit unsigned integer
  in rounding-up mode; inexact exceptions not signaled
PROTOTYPE:
  UINT64 __bid64_to_uint64_ceil (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 64-bit decimal floating-point value to 64-bit unsigned integer
  in rounding-up mode; inexact exceptions signaled
PROTOTYPE:
  UINT64 __bid64_to_uint64_xceil (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Compare 64-bit decimal floating-point numbers for specified relation;
  do not signal invalid exception for quiet NaNs
PROTOTYPE:
  int __bid64_quiet_equal (
    UINT64 x, UINT64 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Compare 64-bit decimal floating-point numbers for specified relation;
  do not signal invalid exception for quiet NaNs
PROTOTYPE:
  int __bid64_quiet_greater (
    UINT64 x, UINT64 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Compare 64-bit decimal floating-point numbers for specified relation;
  do not signal invalid exception for quiet NaNs
PROTOTYPE:
  int __bid64_quiet_greater_equal (
    UINT64 x, UINT64 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Compare 64-bit decimal floating-point numbers for specified relation;
  do not signal invalid exception for quiet NaNs
PROTOTYPE:
  int __bid64_quiet_greater_unordered (
    UINT64 x, UINT64 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Compare 64-bit decimal floating-point numbers for specified relation;
  do not signal invalid exception for quiet NaNs
PROTOTYPE:
  int __bid64_quiet_less (
    UINT64 x, UINT64 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Compare 64-bit decimal floating-point numbers for specified relation;
  do not signal invalid exception for quiet NaNs
PROTOTYPE:
  int __bid64_quiet_less_equal (
    UINT64 x, UINT64 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Compare 64-bit decimal floating-point numbers for specified relation;
  do not signal invalid exception for quiet NaNs
PROTOTYPE:
  int __bid64_quiet_less_unordered (
    UINT64 x, UINT64 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Compare 64-bit decimal floating-point numbers for specified relation;
  do not signal invalid exception for quiet NaNs
PROTOTYPE:
  int __bid64_quiet_not_equal (
    UINT64 x, UINT64 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Compare 64-bit decimal floating-point numbers for specified relation;
  do not signal invalid exception for quiet NaNs
PROTOTYPE:
  int __bid64_quiet_not_greater (
    UINT64 x, UINT64 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Compare 64-bit decimal floating-point numbers for specified relation;
  do not signal invalid exception for quiet NaNs
PROTOTYPE:
  int __bid64_quiet_not_less (
    UINT64 x, UINT64 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Compare 64-bit decimal floating-point numbers for specified relation;
  do not signal invalid exception for quiet NaNs
PROTOTYPE:
  int __bid64_quiet_ordered (
    UINT64 x, UINT64 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Compare 64-bit decimal floating-point numbers for specified relation;
  do not signal invalid exception for quiet NaNs
PROTOTYPE:
  int __bid64_quiet_unordered (
    UINT64 x, UINT64 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Compare 64-bit decimal floating-point numbers for specified relation;
  signal invalid exception for quiet NaNs
PROTOTYPE:
  int __bid64_signaling_greater (
    UINT64 x, UINT64 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Compare 64-bit decimal floating-point numbers for specified relation;
  signal invalid exception for quiet NaNs
PROTOTYPE:
  int __bid64_signaling_greater_equal (
    UINT64 x, UINT64 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Compare 64-bit decimal floating-point numbers for specified relation;
  signal invalid exception for quiet NaNs
PROTOTYPE:
  int __bid64_signaling_greater_unordered (
    UINT64 x, UINT64 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Compare 64-bit decimal floating-point numbers for specified relation;
  signal invalid exception for quiet NaNs
PROTOTYPE:
  int __bid64_signaling_less (
    UINT64 x, UINT64 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Compare 64-bit decimal floating-point numbers for specified relation;
  signal invalid exception for quiet NaNs
PROTOTYPE:
  int __bid64_signaling_less_equal (
    UINT64 x, UINT64 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Compare 64-bit decimal floating-point numbers for specified relation;
  signal invalid exception for quiet NaNs
PROTOTYPE:
  int __bid64_signaling_less_unordered (
    UINT64 x, UINT64 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Compare 64-bit decimal floating-point numbers for specified relation;
  signal invalid exception for quiet NaNs
PROTOTYPE:
  int __bid64_signaling_not_greater (
    UINT64 x, UINT64 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Compare 64-bit decimal floating-point numbers for specified relation;
  signal invalid exception for quiet NaNs
PROTOTYPE:
  int __bid64_signaling_not_less (
    UINT64 x, UINT64 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Compare 128-bit decimal floating-point numbers for specified relation;
  do not signal invalid exception for quiet NaNs
PROTOTYPE:
  int __bid128_quiet_equal (
    UINT128 x, UINT128 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Compare 128-bit decimal floating-point numbers for specified relation;
  do not signal invalid exception for quiet NaNs
PROTOTYPE:
  int __bid128_quiet_greater (
    UINT128 x, UINT128 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Compare 128-bit decimal floating-point numbers for specified relation;
  do not signal invalid exception for quiet NaNs
PROTOTYPE:
  int __bid128_quiet_greater_equal (
    UINT128 x, UINT128 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Compare 128-bit decimal floating-point numbers for specified relation;
  do not signal invalid exception for quiet NaNs
PROTOTYPE:
  int __bid128_quiet_greater_unordered (
    UINT128 x, UINT128 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Compare 128-bit decimal floating-point numbers for specified relation;
  do not signal invalid exception for quiet NaNs
PROTOTYPE:
  int __bid128_quiet_less (
    UINT128 x, UINT128 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Compare 128-bit decimal floating-point numbers for specified relation;
  do not signal invalid exception for quiet NaNs
PROTOTYPE:
  int __bid128_quiet_less_equal (
    UINT128 x, UINT128 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Compare 128-bit decimal floating-point numbers for specified relation;
  do not signal invalid exception for quiet NaNs
PROTOTYPE:
  int __bid128_quiet_less_unordered (
    UINT128 x, UINT128 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Compare 128-bit decimal floating-point numbers for specified relation;
  do not signal invalid exception for quiet NaNs
PROTOTYPE:
  int __bid128_quiet_not_equal (
    UINT128 x, UINT128 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Compare 128-bit decimal floating-point numbers for specified relation;
  do not signal invalid exception for quiet NaNs
PROTOTYPE:
  int __bid128_quiet_not_greater (
    UINT128 x, UINT128 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Compare 128-bit decimal floating-point numbers for specified relation;
  do not signal invalid exception for quiet NaNs
PROTOTYPE:
  int __bid128_quiet_not_less (
    UINT128 x, UINT128 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Compare 128-bit decimal floating-point numbers for specified relation;
  do not signal invalid exception for quiet NaNs
PROTOTYPE:
  int __bid128_quiet_ordered (
    UINT128 x, UINT128 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Compare 128-bit decimal floating-point numbers for specified relation;
  do not signal invalid exception for quiet NaNs
PROTOTYPE:
  int __bid128_quiet_unordered (
    UINT128 x, UINT128 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Compare 128-bit decimal floating-point numbers for specified relation;
  signal invalid exception for quiet NaNs
PROTOTYPE:
  int __bid128_signaling_greater (
    UINT128 x, UINT128 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Compare 128-bit decimal floating-point numbers for specified relation;
  signal invalid exception for quiet NaNs
PROTOTYPE:
  int __bid128_signaling_greater_equal (
    UINT128 x, UINT128 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Compare 128-bit decimal floating-point numbers for specified relation;
  signal invalid exception for quiet NaNs
PROTOTYPE:
  int __bid128_signaling_greater_unordered (
    UINT128 x, UINT128 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Compare 128-bit decimal floating-point numbers for specified relation;
  signal invalid exception for quiet NaNs
PROTOTYPE:
  int __bid128_signaling_less (
    UINT128 x, UINT128 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Compare 128-bit decimal floating-point numbers for specified relation;
  signal invalid exception for quiet NaNs
PROTOTYPE:
  int __bid128_signaling_less_equal (
    UINT128 x, UINT128 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Compare 128-bit decimal floating-point numbers for specified relation;
  signal invalid exception for quiet NaNs
PROTOTYPE:
  int __bid128_signaling_less_unordered (
    UINT128 x, UINT128 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Compare 128-bit decimal floating-point numbers for specified relation;
  signal invalid exception for quiet NaNs
PROTOTYPE:
  int __bid128_signaling_not_greater (
    UINT128 x, UINT128 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Compare 128-bit decimal floating-point numbers for specified relation;
  signal invalid exception for quiet NaNs
PROTOTYPE:
  int __bid128_signaling_not_less (
    UINT128 x, UINT128 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Round 64-bit decimal floating-point value to integral-valued decimal
  floating-point value in the same format, using the current rounding mode;
  signal inexact exceptions
PROTOTYPE:
  UINT64 __bid64_round_integral_exact (
    UINT64 x, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Round 64-bit decimal floating-point value to integral-valued decimal
  floating-point value in the same format, using the rounding-to-nearest-even
  mode; do not signal inexact exceptions
PROTOTYPE:
  UINT64 __bid64_round_integral_nearest_even (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Round 64-bit decimal floating-point value to integral-valued decimal
  floating-point value in the same format, using the rounding-down mode; do not
  signal inexact exceptions
PROTOTYPE:
  UINT64 __bid64_round_integral_negative (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Round 64-bit decimal floating-point value to integral-valued decimal
  floating-point value in the same format, using the rounding-up  mode; do not
  signal inexact exceptions
PROTOTYPE:
  UINT64 __bid64_round_integral_positive (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Round 64-bit decimal floating-point value to integral-valued decimal
  floating-point value in the same format, using the rounding-to-zero  mode;
  do not  signal inexact exceptions
PROTOTYPE:
  UINT64 __bid64_round_integral_zero (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Round 64-bit decimal floating-point value to integral-valued decimal
  floating-point value in the same format, using the rounding-to-nearest-away
  mode; do not signal inexact exceptions
PROTOTYPE:
  UINT64 __bid64_round_integral_nearest_away (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Round 128-bit decimal floating-point value to integral-valued decimal
  floating-point value in the same format, using the current rounding mode;
  signal inexact exceptions
PROTOTYPE:
  UINT128 __bid128_round_integral_exact (
    UINT128 x, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Round 128-bit decimal floating-point value to integral-valued decimal
  floating-point value in the same format, using the rounding-to-nearest-even   mode; do not signal inexact exceptions
PROTOTYPE:
  UINT128 __bid128_round_integral_nearest_even (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Round 128-bit decimal floating-point value to integral-valued decimal
  floating-point value in the same format, using the rounding-down mode; do not
  signal inexact exceptions
PROTOTYPE:
  UINT128 __bid128_round_integral_negative (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Round 128-bit decimal floating-point value to integral-valued decimal
  floating-point value in the same format, using the rounding-up  mode; do not
  signal inexact exceptions
PROTOTYPE:
  UINT128 __bid128_round_integral_positive (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Round 128-bit decimal floating-point value to integral-valued decimal
  floating-point value in the same format, using the rounding-to-zero  mode;
  do not  signal inexact exceptions
PROTOTYPE:
  UINT128 __bid128_round_integral_zero (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Round 128-bit decimal floating-point value to integral-valued decimal
  floating-point value in the same format, using the rounding-to-nearest-away
  mode; do not signal inexact exceptions
PROTOTYPE:
  UINT128 __bid128_round_integral_nearest_away (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Returns the least 64-bit decimal floating-point number that
  compares greater than the operand
PROTOTYPE:
  UINT64 __bid64_nextup (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Returns the greatest 64-bit decimal floating-point number that
  compares less than the operand
PROTOTYPE:
  UINT64 __bid64_nextdown (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Returns the next 64-bit decimal floating-point number that neighbors
  the first operand in the direction toward the second operand
PROTOTYPE:
  UINT64 __bid64_nextafter (
    UINT64 x, UINT64 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Returns the least 128-bit decimal floating-point number that
  compares greater than the operand
PROTOTYPE:
  UINT128 __bid128_nextup (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Returns the greatest 128-bit decimal floating-point number that
  compares less than the operand
PROTOTYPE:
  UINT128 __bid128_nextdown (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Returns the next 128-bit decimal floating-point number that neighbors
  the first operand in the direction toward the second operand
PROTOTYPE:
  UINT128 __bid128_nextafter (
    UINT128 x, UINT128 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Returns the canonicalized floating-point number x if x < y,
  y if y < x, the canonicalized floating-point number if one operand is
  a floating-point number and the other a quiet NaN. Otherwise it is
  either x or y, canonicalized.
PROTOTYPE:
  UINT64 __bid64_minnum (
    UINT64 x, UINT64 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Returns the canonicalized floating-point number x if |x| < |y|,
  y if |y| < |x|, otherwise this function is identical to __bid64_minnum
PROTOTYPE:
  UINT64 __bid64_minnum_mag (
    UINT64 x, UINT64 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Returns the canonicalized floating-point number y if x < y,
  x if y < x, the canonicalized floating-point number if one operand is a
  floating-point number and the other a quiet NaN.  Otherwise it is either x
  or y, canonicalized.
PROTOTYPE:
  UINT64 __bid64_maxnum (
    UINT64 x, UINT64 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Returns the canonicalized floating-point number x if |x| > |y|,
  y if |y| > |x|, otherwise this function is identical to __bid64_maxnum
PROTOTYPE:
  UINT64 __bid64_maxnum_mag (
    UINT64 x, UINT64 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Returns the canonicalized floating-point number x if x < y,
  y if y < x, the canonicalized floating-point number if one operand is
  a floating-point number and the other a quiet NaN. Otherwise it is    either x or y, canonicalized.
PROTOTYPE:
  UINT128 __bid128_minnum (
    UINT128 x, UINT128 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Returns the canonicalized floating-point number x if |x| < |y|,
  y if |y| < |x|, otherwise this function is identical to __bid128_minnum
PROTOTYPE:
  UINT128 __bid128_minnum_mag (
    UINT128 x, UINT128 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Returns the canonicalized floating-point number y if x < y,
  x if y < x, the canonicalized floating-point number if one operand is a
  floating-point number and the other a quiet NaN.  Otherwise it is either x
  or y, canonicalized.
PROTOTYPE:
  UINT128 __bid128_maxnum (
    UINT128 x, UINT128 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Returns the canonicalized floating-point number x if |x| > |y|,
  y if |y| > |x|, otherwise this function is identical to __bid128_maxnum
PROTOTYPE:
  UINT128 __bid128_maxnum_mag (
    UINT128 x, UINT128 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 32-bit signed integer to 64-bit decimal floating-point number
PROTOTYPE:
  UINT64 __bid64_from_int32 (
    int x);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Convert 32-bit unsigned integer to 64-bit decimal floating-point
  number
PROTOTYPE:
  UINT64 __bid64_from_uint32 (
    unsigned int x);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Convert 64-bit signed integer to 64-bit decimal floating-point number
PROTOTYPE:
  UINT64 __bid64_from_int64 (
    SINT64 x, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P

FUNCTION: Convert 64-bit unsigned integer to 64-bit decimal floating-point
  number
PROTOTYPE:
  UINT64 __bid64_from_uint64 (
    UINT64, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P

FUNCTION: Convert 32-bit signed integer to 128-bit decimal floating-point number
PROTOTYPE:
  UINT128 __bid128_from_int32 (
    int x);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Convert 32-bit unsigned integer to 128-bit decimal floating-point
  number
PROTOTYPE:
  UINT128 __bid128_from_uint32 (
    unsigned int x);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Convert 64-bit signed integer to 128-bit decimal floating-point number
PROTOTYPE:
  UINT128 __bid128_from_int64 (
    SINT64 x);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Convert 64-bit unsigned integer to 128-bit decimal floating-point
  number
PROTOTYPE:
  UINT128 __bid128_from_uint64 (
    UINT64 x);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Return true if and only if x has negative sign
PROTOTYPE:
  int __bid64_isSigned (
    UINT64 x);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Return true if and only if x is normal (not zero, subnormal,
  infinite, or NaN)
PROTOTYPE:
  int __bid64_isNormal (
    UINT64 x);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Return true if and only if x is subnormal
PROTOTYPE:
  int __bid64_isSubnormal (
    UINT64 x);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Return true if and only if x is zero, subnormal or normal
  (not infinite or NaN)
PROTOTYPE:
  int __bid64_isFinite (
    UINT64 x);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Return true if and only if x is +0 or -0
PROTOTYPE:
  int __bid64_isZero (
    UINT64 x);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Return true if and only if x is infinite
PROTOTYPE:
  int __bid64_isInf (
    UINT64 x);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Return true if and only if x is a signaling NaN
PROTOTYPE:
  int __bid64_isSignaling (
    UINT64 x);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Return true if and only if x is a finite number, infinity, or
  NaN that is canonical.
PROTOTYPE:
  int __bid64_isCanonical (
    UINT64 x);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Return true if and only if x is a NaN
PROTOTYPE:
  int __bid64_isNaN (
    UINT64 x);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Copies a decimal floating-point operand x to a destination in the
  same format, with no change
PROTOTYPE:
  UINT64 __bid64_copy (
    UINT64 x);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Copies a 64-bit decimal floating-point operand x to a destination
  in the same format, reversing the sign
PROTOTYPE:
  UINT64 __bid64_negate (
    UINT64 x);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Copies a 64-bit decimal floating-point operand x to a destination
  in the same format, changing the sign to positive
PROTOTYPE:
  UINT64 __bid64_abs (
    UINT64 x);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Copies a 64-bit decimal floating-point operand x to a destination
  in the same format as x, but with the sign of y
PROTOTYPE:
  UINT64 __bid64_copySign (
    UINT64 x, UINT64 y);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Tells which of the following ten classes x falls into (details in
  the IEEE Standard 754-2008): signalingNaN, quietNaN, negativeInfinity,
  negativeNormal, negativeSubnormal, negativeZero, positiveZero,
  positiveSubnormal, positiveNormal, positiveInfinity
PROTOTYPE:
  int __bid64_class (
    UINT64 x);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: sameQuantum(x, y) is true if the exponents of x and y are the same,
  and false otherwise; sameQuantum(NaN, NaN) and sameQuantum(inf, inf) are
  true; if exactly one operand is infinite or exactly one operand is NaN,
  sameQuantum is false
PROTOTYPE:
  int __bid64_sameQuantum (
    UINT64 x, UINT64 y);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Return true if x and y are ordered (see the IEEE Standard 754-2008)
PROTOTYPE:
  int __bid64_totalOrder (
    UINT64 x, UINT64 y);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Return true if the absolute values of x and y are ordered
  (see the IEEE Standard 754-2008)
PROTOTYPE:
  int __bid64_totalOrderMag (
    UINT64 x, UINT64 y);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Return the radix b of the format of x, 2 or 10
PROTOTYPE:
  int __bid64_radix (
    UINT64 x);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Return true if and only if x has negative sign
PROTOTYPE:
  int __bid128_isSigned (
    UINT128 x);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Return true if and only if x is normal (not zero, subnormal,
  infinite, or NaN)
PROTOTYPE:
  int __bid128_isNormal (
    UINT128 x);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Return true if and only if x is subnormal
PROTOTYPE:
  int __bid128_isSubnormal (
    UINT128 x);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Return true if and only if x is zero, subnormal or normal
  (not infinite or NaN)
PROTOTYPE:
  int __bid128_isFinite (
    UINT128 x);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Return true if and only if x is +0 or -0
PROTOTYPE:
  int __bid128_isZero (
    UINT128 x);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Return true if and only if x is infinite
PROTOTYPE:
  int __bid128_isInf (
    UINT128 x);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Return true if and only if x is a signaling NaN
PROTOTYPE:
  int __bid128_isSignaling (
    UINT128 x);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Return Return true if and only if x is a finite number, infinity, or    NaN that is canonical.
PROTOTYPE:
  int __bid128_isCanonical (
    UINT128 x);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Return true if and only if x is a NaN
PROTOTYPE:
  int __bid128_isNaN (
    UINT128 x);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Copies a decimal floating-point operand x to a destination in the    same format, with no change
PROTOTYPE:
  UINT128 __bid128_copy (
    UINT128 x);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Copies a 128-bit decimal floating-point operand x to a destination
  in the same format, reversing the sign
PROTOTYPE:
  UINT128 __bid128_negate (
    UINT128 x);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Copies a 128-bit decimal floating-point operand x to a destination
  in the same format, changing the sign to positive
PROTOTYPE:
  UINT128 __bid128_abs (
    UINT128 x);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Copies a 128-bit decimal floating-point operand x to a destination
  in the  same format as x, but with the sign of y
PROTOTYPE:
  UINT128 __bid128_copySign (
    UINT128 x, UINT128 y);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Tells which of the following ten classes x falls into (details in
  the IEEE Standard 754-2008): signalingNaN, quietNaN, negativeInfinity,
  negativeNormal, negativeSubnormal, negativeZero, positiveZero,
  positiveSubnormal, positiveNormal, positiveInfinity
PROTOTYPE:
  int __bid128_class (
    UINT128 x);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: sameQuantum(x, y) returns true if the exponents of x and y are the same,
  and false otherwise; sameQuantum(NaN, NaN) and sameQuantum(inf, inf) are
  true; if exactly one operand is infinite or exactly one operand is NaN,
  sameQuantum is false
PROTOTYPE:
  int __bid128_sameQuantum (
    UINT128 x, UINT128 y);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Return true if x and y are ordered (see the IEEE Standard 754-2008)
PROTOTYPE:
  int __bid128_totalOrder (
    UINT128 x, UINT128 y);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Return true if the absolute values of x and y are ordered
  (see the IEEE Standard 754-2008)
PROTOTYPE:
  int __bid128_totalOrderMag (
    UINT128 x, UINT128 y);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Return the radix b of the format of x, 2 or 10
PROTOTYPE:
  int __bid128_radix (
    UINT128 x);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Decimal floating-point remainder
PROTOTYPE:
  UINT64 __bid64_rem (
    UINT64 x, UINT64 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Returns the exponent e of x, a signed integral value, determined
  as though x were represented with infinite range and minimum exponent
PROTOTYPE:
  UINT64 __bid64_ilogb (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: Z, I

FUNCTION: Returns x * 10^N
PROTOTYPE:
  UINT64 __bid64_scalbn (
    UINT64 x, int n, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Decimal floating-point remainder
PROTOTYPE:
  UINT128 __bid128_rem (
    UINT128 x, UINT128 y, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Returns the exponent e of x, a signed integral value, determined
  as though x were represented with infinite range and minimum exponent
PROTOTYPE:
  UINT128 __bid128_ilogb (
    UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: Z, I

FUNCTION: Returns x * 10^N
PROTOTYPE:
  UINT128 __bid128_scalbn (
    UINT128 x, int n, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 32-bit decimal floating-point value to 64-bit decimal
  floating-point format (binary encoding)
PROTOTYPE:
  UINT64 __bid32_to_bid64 (
    UINT32 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 32-bit decimal floating-point value to 128-bit decimal
  floating-point format (binary encoding)
PROTOTYPE:
  UINT128 __bid32_to_bid128 (
    UINT32 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 64-bit decimal floating-point value to 128-bit decimal
  floating-point format (binary encoding)
PROTOTYPE:
  UINT128 __bid64_to_bid128 (
    UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: I

FUNCTION: Convert 64-bit decimal floating-point value to 32-bit decimal
  floating-point format (binary encoding)
PROTOTYPE:
  UINT32 __bid64_to_bid32 (
    UINT64 x, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, I

FUNCTION: Convert 128-bit decimal floating-point value to 32-bit decimal
  floating-point format (binary encoding)
PROTOTYPE:
  UINT32 __bid128_to_bid32 (
    UINT128 x, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, I

FUNCTION: Convert 128-bit decimal floating-point value to 64-bit decimal
  floating-point format (binary encoding)
PROTOTYPE:
  UINT64 __bid128_to_bid64 (
    UINT128 x, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, I

FUNCTION: Convert 64-bit decimal floating-point value (binary encoding)
  to string format (decimal character sequence)
PROTOTYPE:
  void __bid64_to_string (
    char *ps, UINT64 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Convert a decimal floating-point value represented in string format
  (decimal character sequence) to 64-bit decimal floating-point format
  (binary encoding)
PROTOTYPE:
  UINT64 __bid64_from_string (
    char *ps, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O

FUNCTION: Convert 128-bit decimal floating-point value (binary encoding)
  to string format (decimal character sequence)
PROTOTYPE:
  void __bid128_to_string (
    char *str, UINT128 x, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Convert a decimal floating-point value represented in string format
  (decimal character sequence) to 128-bit decimal floating-point format
  (binary encoding)
PROTOTYPE:
  UINT128 __bid128_from_string (
    char *ps, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O

FUNCTION: Quantize(x, y) is a floating-point number in the same format that
  has, if possible, the same numerical value as x and the same quantum
  (unit-in-the-last-place) as y. If the exponent is being increased, rounding
  according to the prevailing rounding-direction mode might occur: the result
  is a different floating-point representation and inexact is signaled if the
  result does not have the same numerical value as x. If the exponent is being
  decreased and the significand of the result would have more than 16 digits,
  invalid is signaled and the result is NaN. If one or both operands are NaN
  the rules for NaNs are followed. Otherwise if only one operand is
  infinite then invalid is signaled and the result is NaN. If both operands
  are infinite then the result is canonical infinity with the sign of x
PROTOTYPE:
  UINT64 __bid64_quantize (
    UINT64 x, UINT64 y, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Quantize(x, y) is a floating-point number in the same format that
  has, if possible, the same numerical value as x and the same quantum
  (unit-in-the-last-place) as y. If the exponent is being increased, rounding
  according to the prevailing rounding-direction mode might occur: the result
  is a different floating-point representation and inexact is signaled if the
  result does not have the same numerical value as x. If the exponent is being
  decreased and the significand of the result would have more than 34 digits,
  invalid is signaled and the result is NaN. If one or both operands are NaN
  the rules for NaNs are followed. Otherwise if only one operand is
  infinite then invalid is signaled and the result is NaN. If both operands
  are infinite then the result is canonical infinity with the sign of x
PROTOTYPE:
  UINT128 __bid128_quantize (
    UINT128 x, UINT128 y, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 128-bit binary floating-point value to 32-bit decimal
  floating-point format (binary encoding)
PROTOTYPE:
  UINT32 __binary128_to_bid32 (
    BINARY128 x, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Convert 128-bit binary floating-point value to 64-bit decimal
  floating-point format (binary encoding)
PROTOTYPE:
  UINT64 __binary128_to_bid64 (
                  BINARY128 x, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Convert 128-bit binary floating-point value to 128-bit decimal
  floating-point format (binary encoding)
PROTOTYPE:
  UINT128 __binary128_to_bid128 (
    BINARY128 x, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 64-bit binary floating-point value to 32-bit decimal
  floating-point format (binary encoding)
PROTOTYPE:
  UINT32 __binary64_to_bid32 (
    double x, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Convert 64-bit binary floating-point value to 64-bit decimal
  floating-point format (binary encoding)
PROTOTYPE:
  UINT64 __binary64_to_bid64 (
    double x, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 64-bit binary floating-point value to 128-bit decimal
  floating-point format (binary encoding)
PROTOTYPE:
  UINT128 __binary64_to_bid128 (
    double x, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 80-bit binary floating-point value to 32-bit decimal
  floating-point format (binary encoding)
PROTOTYPE:
  UINT32 __binary80_to_bid32 (
    BINARY80 x, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Convert 80-bit binary floating-point value to 64-bit decimal
  floating-point format (binary encoding)
PROTOTYPE:
  UINT64 __binary80_to_bid64 (
    BINARY80 x, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Convert 80-bit binary floating-point value to 128-bit decimal
  floating-point format (binary encoding)
PROTOTYPE:
  UINT128 __binary80_to_bid128 (
    BINARY80 x, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 32-bit binary floating-point value to 32-bit decimal
  floating-point format (binary encoding)
PROTOTYPE:
  UINT32 __binary32_to_bid32 (
    float x, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 32-bit binary floating-point value to 64-bit decimal
  floating-point format (binary encoding)
PROTOTYPE:
  UINT64 __binary32_to_bid64 (
    float x, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 32-bit binary floating-point value to 128-bit decimal
  floating-point format (binary encoding)
PROTOTYPE:
  UINT128 __binary32_to_bid128 (
    float x, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 128-bit decimal floating-point value (binary encoding)
  to 32-bit binary floating-point format
PROTOTYPE:
  float __bid128_to_binary32 (
    UINT128 x, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Convert 128-bit decimal floating-point value (binary encoding)
  to 64-bit binary floating-point format
PROTOTYPE:
  double __bid128_to_binary64 (
    UINT128 x, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Convert 128-bit decimal floating-point value (binary encoding)
  to 80-bit binary floating-point format
PROTOTYPE:
  BINARY80 __bid128_to_binary80 (
    UINT128 x, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Convert 128-bit decimal floating-point value (binary encoding)
  to 128-bit binary floating-point format
PROTOTYPE:
  BINARY128 __bid128_to_binary128 (
    UINT128 x, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Convert 64-bit decimal floating-point value (binary encoding)
  to 32-bit binary floating-point format
PROTOTYPE:
  float __bid64_to_binary32 (
    UINT64 x, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Convert 64-bit decimal floating-point value (binary encoding)
  to 64-bit binary floating-point format
PROTOTYPE:
  double __bid64_to_binary64 (
    UINT64 x, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Convert 64-bit decimal floating-point value (binary encoding)
  to 80-bit binary floating-point format
PROTOTYPE:
  BINARY80 __bid64_to_binary80 (
    UINT64 x, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 64-bit decimal floating-point value (binary encoding)
  to 128-bit binary floating-point format
PROTOTYPE:
  BINARY128 __bid64_to_binary128 (
    UINT64 x, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 32-bit decimal floating-point value (binary encoding)
  to 32-bit binary floating-point format
PROTOTYPE:
  float __bid32_to_binary32 (
    UINT32 x, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, I

FUNCTION: Convert 32-bit decimal floating-point value (binary encoding)
  to 64-bit binary floating-point format
PROTOTYPE:
  double __bid32_to_binary64 (
    UINT32 x, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 32-bit decimal floating-point value (binary encoding)
  to 80-bit binary floating-point format
PROTOTYPE:
  BINARY80 __bid32_to_binary80 (
    UINT32 x, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Convert 32-bit decimal floating-point value (binary encoding)
  to 128-bit binary floating-point format
PROTOTYPE:
  BINARY128 __bid32_to_binary128 (
    UINT32 x, _IDEC_round rnd_mode, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, I

FUNCTION: Return true if and only if this programming environment conforms
  to the 1985 version of the standard
PROTOTYPE:
  int __bid_is754 (
    void);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Return true if and only if this programming environment conforms
  to the revised version of the standard
PROTOTYPE:
  int __bid_is754R (
    void);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Signals the exceptions specified in the flagmask operand, which
  can represent any subset of the exceptions
PROTOTYPE:
  void __bid_signalException (
    _IDEC_flags flagsmask, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: P, U, O, Z, I

FUNCTION: Lowers (clears) the flags corresponding to the exceptions specified
  in the flagmask operand, which can represent any subset of the exceptions
PROTOTYPE:
  void __bid_lowerFlags (
    _IDEC_flags flagsmask, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Queries whether any of the flags corresponding to the exceptions
  specified in the flagsmask operand, which can represent any subset of the
  exceptions, are raised
PROTOTYPE:
  _IDEC_flags __bid_testFlags (
    _IDEC_flags flagsmask, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Queries whether any of the flags in the savedflags operand
  corresponding to the exceptions specified in the flagmask operand, which
  can represent any subset of the exceptions, are raised
PROTOTYPE:
  _IDEC_flags __bid_testSavedFlags (
    _IDEC_flags savedflags, _IDEC_flags flagsmask);
FLOATING-POINT EXCEPTIONS: none
FUNCTION: Restores the flags corresponding to the exceptions specified in the
  flagsmask operand, which can represent any subset of the exceptions, to
  their state represented in the flagsvalues operand
PROTOTYPE:
  void __bid_restoreFlags (
    _IDEC_flags flagsvalues, _IDEC_flags flagsmask, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Returns a representation of the state of those flags corresponding
  to the exceptions specified in the flagmask operand
PROTOTYPE:
  _IDEC_flags __bid_saveFlags (
    _IDEC_flags flagsmask, _IDEC_flags *pfpsf);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Gets the prevailing value of the decimal floating-point rounding
  mode. Under constant specification for the rounding mode, it returns
  the constant value. Under dynamic specification for the rounding mode, it
   returns the current value of the dynamic rounding mode variable. Elsewhere,
  the return value is language-defined (and may be unspecified)
PROTOTYPE:
  _IDEC_round __bid_getDecimalRoundingDirection (
    _IDEC_round rnd_mode);
FLOATING-POINT EXCEPTIONS: none

FUNCTION: Sets the value of the dynamic rounding mode variable. The operand
  may be any of the language-defined representations for the default and
  each specific value of the rounding mode. The effect of this operation if
  used outside the static scope of a dynamic specification for the rounding
  mode is language-defined (and may be unspecified)
PROTOTYPE:
  _IDEC_round __bid_setDecimalRoundingDirection (
    _IDEC_round rounding_mode, _IDEC_round rnd_mode);
FLOATING-POINT EXCEPTIONS: none



## Functions Added in Release 2.0

The functions listed below are described in ISO/IEC TR 24732 (a proposed 
extension to the ISO C99 standard).  They are also implemented in the Intel(R) 
Decimal Floating-Point Math Library.  

In general, the maximum ulp error was estimated mathematically to be:

- below 1 ulp for 32-bit functions
- below 2 ulps for 64-bit functions
- below 8 ulps for 128-bit functions

Testing for hundreds of millions of points did not contradict these values. 

A few exceptions exist (notably for the gamma function family):

- the 32-bit atanh, with measured errors of less than 6 ulps 
- the 64-bit acosh and atanh, with measured errors of less than 6 ulps
- the 128-bit sinh, cosh, and erfc, with measured errors of less than 9 ulps
- the 128-bit tgamma and lgamma functions, whose last 5 digits of the 
  34-digit significand in the result are not reliable

As with the rest of the library, these functions can be built to use a global 
rounding mode and/or global decimal status flags.  Alternatively, they can be 
built to take the rounding mode and a pointer to the status flags as arguments 
(as seen for the functions described in the previous section).
Depending on how the library is built, the arguments and result can be passed 
by value or by reference.

The names used by the implementation differ from the names listed in ISO/IEC 
TR 24732; they are given below (as C comments).   However, the user may easily 
redefine these names by editing the #define statements at the beginning of 
bid_conf.h. 

Below, _Decimal64 stands for BID64;  _Decimal32 stands for BID32 and _Decimal128 
stands for BID128. 

```
_Decimal64 acosd64(_Decimal64 x);  // library name: __bid64_acos    
_Decimal32 acosd32(_Decimal32 x);  // library name: __bid32_acos
_Decimal128 acosd128(_Decimal128 x);  // library name: __bid128_acos
_Decimal64 asind64(_Decimal64 x);  // library name: __bid64_asin
_Decimal32 asind32(_Decimal32 x);  // library name: __bid32_asin
_Decimal128 asind128(_Decimal128 x);  // library name: __bid128_asin
_Decimal64 atand64(_Decimal64 x);  // library name: __bid64_atan
_Decimal32 atand32(_Decimal32 x);  // library name: __bid32_atan
_Decimal128 atand128(_Decimal128 x);  // library name: __bid128_atan
_Decimal64 atan2d64(_Decimal64 y, _Decimal64 x);  // library name: __bid64_atan2
_Decimal32 atan2d32(_Decimal32 y, _Decimal32 x);  // library name: __bid32_atan2
_Decimal128 atan2d128(_Decimal128 y, _Decimal128 x);  // library name: __bid128_atan2
_Decimal64 cosd64(_Decimal64 x);  // library name: __bid64_cos
_Decimal32 cosd32(_Decimal32 x);  // library name: __bid32_cos
_Decimal128 cosd128(_Decimal128 x);  // library name: __bid128_cos
_Decimal64 sind64(_Decimal64 x);  // library name: __bid64_sin
_Decimal32 sind32(_Decimal32 x);  // library name: __bid32_sin
_Decimal128 sind128(_Decimal128 x);  // library name: __bid128_sin
_Decimal64 tand64(_Decimal64 x);  // library name: __bid64_tan
_Decimal32 tand32(_Decimal32 x);  // library name: __bid32_tan
_Decimal128 tand128(_Decimal128 x);  // library name: __bid128_tan
_Decimal64 acoshd64(_Decimal64 x);  // library name: __bid64_acosh
_Decimal32 acoshd32(_Decimal32 x);  // library name: __bid32_acosh
_Decimal128 acoshd128(_Decimal128 x);  // library name: __bid128_acosh
_Decimal64 asinhd64(_Decimal64 x);  // library name: __bid64_asinh
_Decimal32 asinhd32(_Decimal32 x);  // library name: __bid32_asinh
_Decimal128 asinhd128(_Decimal128 x);  // library name: __bid128_asinh
_Decimal64 atanhd64(_Decimal64 x);  // library name: __bid64_atanh
_Decimal32 atanhd32(_Decimal32 x);  // library name: __bid32_atanh
_Decimal128 atanhd128(_Decimal128 x);  // library name: __bid128_atanh
_Decimal64 coshd64(_Decimal64 x);  // library name: __bid64_cosh
_Decimal32 coshd32(_Decimal32 x);  // library name: __bid32_cosh
_Decimal128 coshd128(_Decimal128 x);  // library name: __bid128_cosh
_Decimal64 sinhd64(_Decimal64 x);  // library name: __bid64_sinh
_Decimal32 sinhd32(_Decimal32 x);  // library name: __bid32_sinh
_Decimal128 sinhd128(_Decimal128 x);  // library name: __bid128_sinh
_Decimal64 tanhd64(_Decimal64 x);  // library name: __bid64_tanh
_Decimal32 tanhd32(_Decimal32 x);  // library name: __bid32_tanh
_Decimal128 tanhd128(_Decimal128 x);  // library name: __bid128_tanh
_Decimal64 expd64(_Decimal64 x);  // library name: __bid64_exp
_Decimal32 expd32(_Decimal32 x);  // library name: __bid32_exp
_Decimal128 expd128(_Decimal128 x);  // library name: __bid128_exp
_Decimal64 exp2d64(_Decimal64 x);  // library name: __bid64_exp2
_Decimal32 exp2d32(_Decimal32 x);  // library name: __bid32_exp2
_Decimal128 exp2d128(_Decimal128 x);  // library name: __bid128_exp2
_Decimal64 expm1d64(_Decimal64 x);  // library name: __bid64_expm1
_Decimal32 expm1d32(_Decimal32 x);  // library name: __bid32_expm1
_Decimal128 expm1d128(_Decimal128 x);  // library name: __bid128_expm1
_Decimal64 frexpd64(_Decimal64 value, int *exp);  // library name: __bid64_frexp
_Decimal32 frexpd32(_Decimal32 value, int *exp);  // library name: __bid32_frexp
_Decimal128 frexpd128(_Decimal128 value, int *exp);  // library name: __bid128_frexp
_Decimal64 ldexpd64(_Decimal64 x, int exp);  // library name: __bid64_ldexp
_Decimal32 ldexpd32(_Decimal32 x, int exp);  // library name: __bid32_ldexp
_Decimal128 ldexpd128(_Decimal128 x, int exp);  // library name: __bid128_ldexp
_Decimal64 logd64(_Decimal64 x);  // library name: __bid64_log
_Decimal32 logd32(_Decimal32 x);  // library name: __bid32_log
_Decimal128 logd128(_Decimal128 x);  // library name: __bid128_log
_Decimal64 log10d64(_Decimal64 x);  // library name: __bid64_log10
_Decimal32 log10d32(_Decimal32 x);  // library name: __bid32_log10
_Decimal128 log10d128(_Decimal128 x);  // library name: __bid128_log10
_Decimal64 log1pd64(_Decimal64 x);  // library name: __bid64_log1p
_Decimal32 log1pd32(_Decimal32 x);  // library name: __bid32_log1p
_Decimal128 log1pd128(_Decimal128 x);  // library name: __bid128_log1p
_Decimal64 log2d64(_Decimal64 x);  // library name: __bid64_log2
_Decimal32 log2d32(_Decimal32 x);  // library name: __bid32_log2
_Decimal128 log2d128(_Decimal128 x);  // library name: __bid128_log2
_Decimal64 modfd64(_Decimal64 value, _Decimal64 *iptr);  // library name: __bid64_modf
_Decimal32 modfd32(_Decimal32 value, _Decimal32 *iptr);  // library name: __bid32_modf
_Decimal128 modfd128(_Decimal128 value, _Decimal128 *iptr);  // library name: __bid128_modf
_Decimal64 scalblnd64(_Decimal64 x, long int n);  // library name: __bid64_scalbln
_Decimal32 scalblnd32(_Decimal32 x, long int n);  // library name: __bid32_scalbln
_Decimal128 scalblnd128(_Decimal128 x, long int n);  // library name: __bid128_scalbln
_Decimal64 cbrtd64(_Decimal64 x);  // library name: __bid64_cbrt
_Decimal32 cbrtd32(_Decimal32 x);  // library name: __bid32_cbrt
_Decimal128 cbrtd128(_Decimal128 x);  // library name: __bid128_cbrt
_Decimal64 fabsd64(_Decimal64 x);  // library name: __bid64_abs
_Decimal32 fabsd32(_Decimal32 x);  // library name: __bid32_abs
_Decimal128 fabsd128(_Decimal128 x);  // library name: __bid128_abs
_Decimal64 hypotd64(_Decimal64 x, _Decimal64 y);  // library name: __bid64_hypot
_Decimal32 hypotd32(_Decimal32 x, _Decimal32 y);  // library name: __bid32_hypot
_Decimal128 hypotd128(_Decimal128 x, _Decimal128 y);  // library name: __bid128_hypot
_Decimal64 powd64(_Decimal64 x, _Decimal64 y);  // library name: __bid64_pow
_Decimal32 powd32(_Decimal32 x, _Decimal32 y);  // library name: __bid32_pow
_Decimal128 powd128(_Decimal128 x, _Decimal128 y);  // library name: __bid128_pow
_Decimal64 erfd64(_Decimal64 x);  // library name: __bid64_erf
_Decimal32 erfd32(_Decimal32 x);  // library name: __bid32_erf
_Decimal128 erfd128(_Decimal128 x);  // library name: __bid128_erf
_Decimal64 erfcd64(_Decimal64 x);  // library name: __bid64_erfc
_Decimal32 erfcd32(_Decimal32 x);  // library name: __bid32_erfc
_Decimal128 erfcd128(_Decimal128 x);  // library name: __bid128_erfc
_Decimal64 lgammad64(_Decimal64 x);  // library name: __bid64_lgamma
_Decimal32 lgammad32(_Decimal32 x);  // library name: __bid32_lgamma
_Decimal128 lgammad128(_Decimal128 x);  // library name: __bid128_lgamma
_Decimal64 tgammad64(_Decimal64 x);  // library name: __bid64_tgamma
_Decimal32 tgammad32(_Decimal32 x);  // library name: __bid32_tgamma
_Decimal128 tgammad128(_Decimal128 x);  // library name: __bid128_tgamma
_Decimal64 ceild64(_Decimal64 x);  // library name: __bid64_ceil
_Decimal32 ceild32(_Decimal32 x);  // library name: __bid32_ceil
_Decimal128 ceild128(_Decimal128 x);  // library name: __bid128_ceil
_Decimal64 floord64(_Decimal64 x);  // library name: __bid64_floor
_Decimal32 floord32(_Decimal32 x);  // library name: __bid32_floor
_Decimal128 floord128(_Decimal128 x);  // library name: __bid128_floor
_Decimal64 nearbyintd64(_Decimal64 x);  // library name: __bid64_nearbyint
_Decimal32 nearbyintd32(_Decimal32 x);  // library name: __bid32_nearbyint
_Decimal128 nearbyintd128(_Decimal128 x);  // library name: __bid128_nearbyint
long int lrintd64(_Decimal64 x);  // library name: __bid64_lrint
long int lrintd32(_Decimal32 x);  // library name: __bid32_lrint
long int lrintd128(_Decimal128 x);  // library name: __bid128_lrint
long long int llrintd64(_Decimal64 x);  // library name: __bid64_llrint
long long int llrintd32(_Decimal32 x);  // library name: __bid32_llrint
long long int llrintd128(_Decimal128 x);  // library name: __bid128_llrint
long int lroundd64(_Decimal64 x);  // library name: __bid64_lround
long int lroundd32(_Decimal32 x);  // library name: __bid32_lround
long int lroundd128(_Decimal128 x);  // library name: __bid128_lround
long long int llroundd64(_Decimal64 x);  // library name: __bid64_llround
long long int llroundd32(_Decimal32 x);  // library name: __bid32_llround
long long int llroundd128(_Decimal128 x);  // library name: __bid128_llround
_Decimal64 nexttowardd64(_Decimal64 x, _Decimal128 y);  // library name: __bid64_nexttoward
_Decimal32 nexttowardd32(_Decimal32 x, _Decimal128 y);  // library name: __bid32_nexttoward
_Decimal128 nexttowardd128(_Decimal128 x, _Decimal128 y);  // library name: __bid128_nexttoward
_Decimal64 fdimd64(_Decimal64 x, _Decimal64 y);  // library name: __bid64_fdim
_Decimal32 fdimd32(_Decimal32 x, _Decimal32 y);  // library name: __bid32_fdim
_Decimal128 fdimd128(_Decimal128 x, _Decimal128 y);  // library name: __bid128_fdim
int samequantumd64 (_Decimal64 __x, _Decimal64 __y); // library name: __bid64_sameQuantum
int samequantumd32 (_Decimal32 __x, _Decimal32 __y); // library name: __bid32_sameQuantum
int samequantumd128 (_Decimal128 __x, _Decimal128 __y); // library name: __bid128_sameQuantum
```

Calling Transcendental Functions
_________________________________

Transcendental function prototypes are similar to those of the basic decimal
functions, so they are called in a similar manner.
All function prototypes can be found in bid_functions.h.

As an example, consider the following code from bid_functions.h (also consult
bid_conf.h, where the user can rename functions via #define directives):

```
#if DECIMAL_CALL_BY_REFERENCE
....
     BID_EXTERN_C void bid64_exp (BID_UINT64 * pres, BID_UINT64 * px
                            _RND_MODE_PARAM _EXC_FLAGS_PARAM
                            _EXC_MASKS_PARAM _EXC_INFO_PARAM);
....
     BID_EXTERN_C void bid128_pow (BID_UINT128 * pres, BID_UINT128 * px, BID_UINT128 * py
                            _RND_MODE_PARAM _EXC_FLAGS_PARAM
                            _EXC_MASKS_PARAM _EXC_INFO_PARAM);
....
#else
....
     BID_EXTERN_C BID_UINT64 bid64_exp (BID_UINT64 x
                              _RND_MODE_PARAM _EXC_FLAGS_PARAM
                              _EXC_MASKS_PARAM _EXC_INFO_PARAM);
....
     BID_EXTERN_C BID_UINT128 bid128_pow (BID_UINT128 x, BID_UINT128 y
                              _RND_MODE_PARAM _EXC_FLAGS_PARAM
                              _EXC_MASKS_PARAM _EXC_INFO_PARAM);
....
#endif
```

Consider also the following code from bid_conf.h:

```
....  #define bid64_exp         __bid64_exp
....
#define bid128_pow        __bid128_pow
...
```

The code examples provided with this package (in the EXAMPLES directory) 
show how to call __bid128_mul(), and can be followed for transcendental 
functions as well.

Here are two simple examples, to compute z=bid128_pow(x,y) and b=bid64_exp(a):

  Decimal128 x, y, z;
  Decimal64 a, b;

  // The user must initialize input arguments x, y, a

  // Call sequences, assuming the library is built using:
  //  DECIMAL_CALL_BY_REFERENCE=1 (arguments passed by reference) 
  //  DECIMAL_GLOBAL_ROUNDING=0 (rounding mode passed as argument to function)
  //  DECIMAL_GLOBAL_EXCEPTION_FLAGS=0 (status flags passed as argument)
  // Need to declare and initialize rounding mode and status flags variables
  _IDEC_round my_rnd_mode = _IDEC_dflround;
  _IDEC_flags my_fpsf = _IDEC_allflagsclear;
  .....
  __bid128_pow (&z, &x, &y, &my_rnd_mode, &my_fpsf);
  __bid64_exp (&b, &a, &my_rnd_mode, &my_fpsf);

  // Call sequences, assuming the library is built using:
  //  DECIMAL_CALL_BY_REFERENCE=0 (arguments passed by value) 
  //  DECIMAL_GLOBAL_ROUNDING=1 (rounding mode stored in global variable)
  //  DECIMAL_GLOBAL_EXCEPTION_FLAGS=1 (status flags stored in global variable)
  z = __bid128_pow (x, y);
  b = __bid64_exp (a);

  For more details, including how to access the global rounding mode and 
status flags variables and how to call decimal functions when the library is 
built using any of the 8 combinations specified by DECIMAL_CALL_BY_REFERENCE, 
DECIMAL_GLOBAL_ROUNDING, and DECIMAL_GLOBAL_EXCEPTION_FLAGS, please see the 
EXAMPLES directory.


## Footnotes:

* BID stands for Binary Integer Decimal, which is an informal name for
the binary encoding format of decimal floating-point values, described in the
IEEE Standard 754-2008.

** Other names and brands may be claimed as the property of others.

*** Microsoft, Windows, and the Windows logo are trademarks, or registered
trademarks of Microsoft Corporation in the United States and/or other countries


## Note 1:

Functions operating on the BID32 format are implemented, but some are not 
listed here (for example bid32_add). See the prototypes in 
LIBRARY/src/bid_functions.h for a complete list of the library functions.

## Note 2:

UNCHANGED_BINARY_STATUS_FLAGS allows for prevention of binary flag pollution 


