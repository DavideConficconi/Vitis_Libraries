# AMD/Xilinx Vitis Library for Regular Expression Matching

This repository is intended to provide a custom version of the Vitis Library for Regex Matching procedure only.
All the content is property of AMD/Xilinx licenses.
Most of this material comes from AMD/Xilinx Documentation.

Useful links: 
[Vitis&trade; Unified Software Platform](https://www.xilinx.com/products/design-tools/vitis/vitis-platform.html) 
[Vitis Libraries documentation](https://xilinx.github.io/Vitis_Libraries/)

## L1 Kernel

#### regexVM
Implementation for regular expression VM (1 instruction per iteration).

`#include "xf_data_analytics/text/regexVM.hpp"`

```
#include "xf_data_analytics/text/regexVM.hpp"
template <int STACK_SIZE>
void regexVM (
    ap_uint <32>* bitSetBuff,
    ap_uint <64>* instrBuff,
    ap_uint <32>* msgBuff,
    unsigned int lenMsg,
    ap_uint <2>& match,
    ap_uint <16>* offsetBuff
    )
```

| **Parameter** |                                                 **Description**                                                 |
|:-------------:|:---------------------------------------------------------------------------------------------------------------:|
| _STACK_SIZE_  |                                             Size of internal stack.                                             |
| _bitSetBuff_  |                                         Bit set map for character class.                                        |
| _instrBuff_   |                                               Instruction buffer.                                               |
| _msgBuff_     |                                         Message buffer as input string.                                         |
| _lenMsg_      |                                             Length of input string.                                             |
| _match_       | Flag to indicate whether the input string is matched or not, 0 for mismatch, 1 for match, 2 for stack overflow. |
| _offsetBuff_  |                                     Offset address for each capturing group.                                    |

#### regexVM Optimized
Implementation for regular expression VM (2 instructions per iteration).

`#include "xf_data_analytics/text/regexVM.hpp"`

```
template <int STACK_SIZE>
void regexVM_opt (
    ap_uint <32>* bitSetBuff,
    ap_uint <64>* instrBuff,
    ap_uint <32>* msgBuff,
    unsigned int lenMsg,
    ap_uint <2>& match,
    ap_uint <16>* offsetBuff
    )

```

| **Parameter** |  **Description**  |
|:-------------:|:---------:|
| _STACK_SIZE_  |                                             Size of internal stack.                                             |
| _bitSetBuff_  |                                         Bit set map for character class.                                        |
| _instrBuff_   |                                               Instruction buffer.                                               |
| _msgBuff_     |                                         Message buffer as input string.                                         |
| _lenMsg_      |                                             Length of input string.                                             |
| _match_       | Flag to indicate whether the input string is matched or not, 0 for mismatch, 1 for match, 2 for stack overflow. |
| _offsetBuff_  |                                     Offset address for each capturing group.                                    |

### Design Internals regex-VM
Extended ASCII excluded, and Nested repetition is not supported.

Compilation and ISA based on instructions and compiler from popular [Oniguruma](https://github.com/kkos/oniguruma.git) library and the VM implementation based on their [Ruby regex implementation](https://github.com/k-takata/Onigmo)

#### Software Compiler (C)
Compiles any regular expression given by user into an instruction list along with the corresponding bit-set map, number of instructions/character-classes/capturing-groups, and the name of each capturing group (if specified in input pattern).

#### Hardware VM (C++)
which takes the outputs from the compiler mentioned above to construct a practical matcher to match the string given in message buffer, and emit a 2-bit match flag indicating whether the input string is matched with the pattern or an internal stack overflow is happened. Futhermore, if the input string is matched, the offset addresses for each capturing group is provided in the output offset buffer, users can find the sub-strings in interest by picking them out from the whole input string according to the information given in that buffer.

#### Supported OPs

| **OP list 1** | **OP list 2** |
|:-------------:|:-------------:|
| END                         | STR_1   |
| STR_2                       | STR_3   |
| STR_4                       | STR_5 |
| STR_N                       | CCLASS    |
| CCLASS_NOT                  |  ANYCHAR  |
| ANYCHAR_STAR                | BEGIN_BUF  |
| END_BUF                     |BEGIN_LINE   |
| END_LINE                    |  MEM_START   |
| MEM_START_PUSH              | MEM_END     |
| FAIL                        |JUMP      |
| PUSH                        | POP   |
| POP_TO_MARK                 | PUSH_OR_JUMP_EXACT1   |
| REPEAT                      |  REPEAT_INC      |
| STEP_BACK_START             | MARK       |

Therefore, the supported atomic regular expressions and their corresponding descriptions should be:

| **Regex** | **Description**|
|:-------------:|:----------------------------------------:|
| ``^``             | asserts position at start of a line                                                                  |
| ``$``             | asserts position at the end of a line                                                                |
| ``\A``            | asserts position at start of the string                                                              |
| ``\z``            | asserts position at the end of the string                                                            |
| ``\ca``           | matches the control sequence ``CTRL+a``                                                              |
| ``\C``            | matches one data unit, even in UTF mode (best avoided)                                               |
| ``\c\\``          | matches the control sequence ``CTRL+\``                                                              |
| ``\s``            | matches any whitespace character (equal to ``[\r\n\t\f\v ]``)                                        |
| ``\S``            | matches any non-whitespace character (equal to ``[^\r\n\t\f\v ]``)                                   |
| ``\d``            | matches a digit (equal to ``[0-9]``)                                                                 |
| ``\D``            | matches any character that's not a digit (equal to ``[^0-9]``)                                       |
| ``\h``            | matches any horizontal whitespace character (equal to ``[[:blank:]]``)                               |
| ``\H``            | matches any character that's not a horizontal whitespace character                                   |
| ``\w``            | matches any word character (equal to ``[a-zA-Z0-9_]``)                                               |
| ``\W``            | matches any non-word character (equal to ``[^a-zA-Z0-9_]``)                                          |
| ``\^``            | matches the character ``^`` literally                                                                |
| ``\$``            | matches the character ``$`` literally                                                                |
| ``\N``            | matches any non-newline character                                                                    |
| ``\g'0'``         | recurses the 0th subpattern                                                                          |
| ``\o{101}``       | matches the character ``A`` with index with ``101(oct)``                                             |
| ``\x61``          | matches the character ``a (hex 61)`` literally                                                       |
| ``\x{1 2}``       | matches ``1 (hex)`` or ``2 (hex)``                                                                   |
| ``\17``           | matches the character ``oct 17`` literally                                                           |
| ``abc``           | matches the ``abc`` literally                                                                        |
| ``.``             | matches any character (except for line terminators)                                                  |
| ``|``             | alternative                                                                                          |
| ``[^a]``          | match a single character not present in the list below                                               |
| ``[a-c]``         | matches ``a``, ``b``, or ``c``                                                                       |
| ``[abc]``         | matches ``a``, ``b``, or ``c``                                                                       |
| ``[:upper:]``     | matches a uppercase letter ``[A-Z]``                                                                 |
| ``a?``            | matches the ``a`` zero or one time (**greedy**)                                                      |
| ``a*``            | matches ``a`` between zero and unlimited times (**greedy**)                                          |
| ``a+``            | matches ``a`` between one and unlimited times (**greedy**)                                           |
| ``a??``           | matches ``a`` between zero and one times (**lazy**)                                                  |
| ``a*?``           | matches ``a`` between zero and unlimited times (**lazy**)                                            |
| ``a+?``           | matches ``a`` between one and unlimited times (**lazy**)                                             |
| ``a{2}``          | matches ``a`` exactly 2 times                                                                        |
| ``a{0,}``         | matches ``a`` between zero and unlimited times                                                       |
| ``a{1,2}``        | matches ``a`` one or two times                                                                       |
| ``{,}``           | matches ``{,}`` literally                                                                            |
| ``(?#blabla)``    | comment ``blabla``                                                                                   |
| ``(a)``           | capturing group, matches ``a`` literally                                                             |
| ``(?<name1> a)``  | named capturing group ``name1``, matches ``a`` literally                                             |
| ``(?:)``          | non-capturing group                                                                                  |
| ``(?i)``          | match the remainder of the pattern with the following effective flags: gmi (i modifier: insensitive) |
| ``(?<!a)z``       | matches any occurrence of ``z`` that is not preceded by ``a`` (negative look-behind)                 |
| ``z(?!a)``        | match any occurrence of ``z`` that is not followed by ``a`` (negative look-ahead)                    |

.. ATTENTION::
    1. Supported encoding method in current release is ASCII (extended ASCII codes are excluded).
    2. Nested repetition is not supported

#### VM Usage
Compile the pattern: if supported ``ONIG_NORMAL`` along with the configurations (including instruction list, bit-set map etc.) will be given if the input is a valid pattern; if not supported the compiler will give an error code ``XF_UNSUPPORTED_OPCODE``.

Only the stack is allocated in the HW itself. The ``STACK_SIZE`` should better be set to be a multiple of 4096 for not wasting the space of individual URAM block. 

#### Code Example
To use the regex-VM you need to:

1. Compile the software regular expression compiler by running ``make`` command in path ``L1/tests/text/regex_vm/re_compile``

2. Include the ``xf_re_compile.h`` header in path ``L1/include/sw/xf_data_analytics/text`` and the ``oniguruma.h`` header in path ``L1/tests/text/regex_vm/re_compile/lib/include``

```cpp

    #include "oniguruma.h"
    #include "xf_re_compile.h"
```
3. Compile your regular expression by calling ``xf_re_compile``

```cpp

    int r = xf_re_compile(pattern, bitset, instr_buff, instr_num, cclass_num, cpgp_num, NULL, NULL);
```
4. Check the return value to see if its a valid pattern and supported by hardware VM. ``ONIG_NORMAL`` is returned if the pattern is valid, and ``XF_UNSUPPORTED_OPCODE`` is returned if it's not supported currently.

```cpp

    if (r != XF_UNSUPPORTED_OPCODE && r == ONIG_NORMAL) {
        // calling hardware VM here for acceleration
    }
```
5. Once the regular expression is verified as a supported pattern, you may call hardware VM to match any message you want by

```cpp
    // for data types used in VM
    #include "ap_int.h"
    // header for hardware VM implementation
    #include "xf_data_analytics/text/regexVM.hpp"

    // allocate memory for bit-set map
    unsigned int bitset[8 * cclass_num];
    // allocate memory for instruction buffer (derived from software compiler)
    uint64_t instr_buff[instr_num];
    // allocate memory for message
    ap_uint<32> msg_buff[MESSAGE_SIZE];
    // set up input message buffer according to input string
    unsigned str_len = strlen((const char*)in_str);
    for (int i = 0; i < (str_len + 3) / 4;  i++) {
        for (int k = 0; k < 4; k++) {
            if (i * 4 + k < str_len) {
                msg_buff[i].range((k + 1) * 8 - 1, k * 8) = in_str[i * 4 + k];
            } else {
                // pad white-space at the end
                msg_buff[i].range((k + 1) * 8 - 1, k * 8) = ' ';
            }
        }
    }
    // allocate memory for offset addresses for each capturing group
    uint16_t offset_buff[2 * (cpgp_num + 1)];
    // initialize offset buffer
    for (int i = 0; i < 2 * CAP_GRP_NUM; i++) {
        offset_buff[i] = -1;
    }
    ap_uint<2> match = 0;
    // call for hardware acceleration (basic hardware VM implementation)
    xf::data_analytics::text:regexVM<STACK_SIZE>((ap_uint<32>*)bitset, (ap_uint<64>*)instr_buff, msg_buff, str_len, match, offset_buff);
    // or call for hardware acceleration (performance optimized hardware VM implementation)
    xf::data_analytics::text:regexVM_opt<STACK_SIZE>((ap_uint<32>*)bitset, (ap_uint<64>*)instr_buff, msg_buff, str_len, match, offset_buff);
```

The match flag and offset addresses for each capturing group are presented in ``match`` and ``offset_buff`` respectively with the format shown in the tables below.

Truth table for the 2-bit output ``match`` flag of hardware VM:

| **Value** |  **Description**  |
|:-------------:|:---------:|
| 0     | mismatched              |
| 1     | matched                 |
| 2     | internal stack overflow |
| 3     | reserved for future use |

Arrangement of the offset buffer ``offsetBuff``:

| **Address** |  **Description**  |
|:-------------:|:---------:|
| 0       | start position of the whole matched string  |
| 1       | end position of the whole matched string    |
| 2       | start position of the 1st capturing group   |
| 3       | end position of the 1st capturing group     |
| 4       | start position of the 2nd capturing group   |
| 5       | end position of the 2nd capturing group     |
| ...     | ...                                         |

#### Implementation

The 64-bit instruction format for communication between software compiler and hardware VM can be explained like this:
<img align="center" src="regex-vm/data_analytics/images/instruction_format.png" alt="drawing" width="300"/>

They employ absolute address.
Location of the source of the software compiler: ``L1/src/sw/xf_re_compile.c``

##### HW-based optimization
1. Simplify the internal logic for each OP we added as mush as we can.

2. Merge the newly added OP into another if possible to let them share the same logic.

3. Offload runtime calculations to software compiler for pre-calculation if possible to improve the runtime performance.

4. Separate the data flow and control flow, do pre-fetch and post-store operations to improve memory access efficiency.

5. Resolve the read-and-write dependency of on-chip RAMs by caching intermediate data in registers to avoid unnecessary accesses.

6. Execute a predict (2nd) instruction in each iteration to accelerate the process under specific circumstances. (performance optimized version executes 2 instructions / 3 cycles)


For the following scenarios, the predict instruction will not be executed:

    1. Read/write the internal stack simultaneously

    2. OP for 2nd instruction is any_char_star, pop_to_mark, or mem_start_push

    3. Jump on OP address happened in 1st instruction

    4. Read/write the offset buffer simultaneously

    5. Pointer for input string moves in 1st instruction and 2nd instruction goes into the OP which needs character comparision

    6. Write the offset buffer simultaneously


Profiling
=========

The hardware resource utilization of hardware VM is shown in the table below (performance optimized version at FMax = 352MHz).

| **Primitive**      | **CLB**   |  **LUT** |   **FF**   |  **BRAM**  | **URAM** | **DSP** | **SRL** |
|:-------------:|:----:|:----:|:----:|:----:|:----:|:----:|:----:|
| hardware VM    | 305   | 1690 |  973   |    0   |  4   |  0  | 0   |

## L2 Kernel

```cpp
template <
    int PU_NM,
    int INSTR_DEPTH,
    int CCLASS_NM,
    int CPGP_NM,
    int MSG_LEN,
    int STACK_SIZE
    >
void reEngine (
    ap_uint <64>* cfg_in_buff,
    ap_uint <64>* msg_in_buff,
    ap_uint <16>* len_in_buff,
    ap_uint <32>* out_buff
    )
```

The reEngine executes the input messages with configured RE pattern.
The pattern is pre-compiled to a list of instructions and is provied by user through the cfg_buff.
Dynamically configurable
User could improve the throughput by increasing the template parameter PU_NM to accelerate the matching process by sacrificing the on-board resources.


### reEngine Usage

The ``reEngineKernel`` will automatically be responsible for splitting them to individual inputs and feeding them into the buffers required by L1 regex-VM, also collecting the results after the matching process done and placing them into output result buffer correspondingly.


**cfg_buff**

<img align="center" src="regex-vm/data_analytics/images/cfgbuff_format.png" alt="drawing" width="300"/>

**msg_buff**
<img align="center" src="regex-vm/data_analytics/images/msgbuff_format.png" alt="drawing" width="300"/>

**len_buff**
<img align="center" src="regex-vm/data_analytics/images/lenbuff_format.png" alt="drawing" width="300"/>

---------------------------------

To use the regex-VM you need to:

1. Compile the software regular expression compiler by running ``make`` command in path ``L1/tests/text/regex_vm/re_compile``

2. Include the ``xf_re_compile.h`` header in path ``L1/include/sw/xf_data_analytics/text`` and the ``oniguruma.h`` header in path ``L1/tests/text/regex_vm/re_compile/lib/include``

```cpp

    #include "oniguruma.h"
    #include "xf_re_compile.h"
```

3. Compile your regular expression by calling ``xf_re_compile``

```cpp

    // Number of instructions tranlated from the pattern
    unsigned int instr_num = 0;
    // Number of character classes in the pattern
    unsigned int cclass_num = 0;
    // Number of capturing groups in the pattern
    unsigned int cpgp_num = 0;
    // Bit set map
    unsigned int* bitset = new unsigned int[8 * CCLASS_NM];
    // Configuration buffer
    uint64_t* cfg_buff = aligned_alloc<uint64_t>(INSTRUC_SIZE);
    // Suppose 1k bytes is long enough for names of each capturing group
    uint8_t* cpgp_name_val = aligned_alloc<uint8_t>(1024);
    // Suppose the number of capturing groups is less than 20
    uint32_t* cpgp_name_offt = aligned_alloc<uint32_t>(20);
    // Leave 2 64-bit space for configuration headers
    int r = xf_re_compile(pattern, bitset, cfg_buff + 2, &instr_num, &cclass_num, &cpgp_num, cpgp_name_val, cpgp_name_offt);

    // Print a name table for all of the capturing groups
    printf("Name Table\n");
    for (int i = 0; i < cpgp_num; i++) {
        printf("Group-%d: ", i);
        for (int j = 0; j < cpgp_name_offt[i + 1] - cpgp_name_offt[i]; j++) {
            printf("%c", cpgp_name_val[j + cpgp_name_offt[i]]);
        }
        printf("\n");
    }
```

4. Check the return value to see if its a valid pattern and supported by hardware VM. ``ONIG_NORMAL`` is returned if the pattern is valid, and ``XF_UNSUPPORTED_OPCODE`` is returned if it's not supported currently.

```cpp
    if (r != XF_UNSUPPORTED_OPCODE && r == ONIG_NORMAL) {
        // Prepare the buffers and call reEngine for acceleration here
    }
```

5. Once the regular expression is verified as a supported pattern, you may prepare the input buffers and get the results by

```cpp
    // Function for writing one line of log to the corresponding buffers
    int writeOneLine (uint64_t* msg_buff, uint16_t* len_buff, unsigned int& offt, unsigned int& msg_nm, std::string& line) {
        typedef union {
            char c_a[8];
            uint64_t d;
        } uint64_un;
        unsigned int sz = line.size();
        if (sz > 4088) {
            printf("Message length exceeds the max limitation\n");
            return 0;
        }
        if ((zs + 7) / 8 + offt > MAX_MSG_SZ || msg_nm > MAX_MSG_NM) {
            printf("Input log size exceeds supported max size\n");
            return -1;
        } else {
            // transform the input char sequence into individual 64-bit blocks and put them into msg_buff
            for (unsigned int i = 0; i < (sz + 7) / 8; i++) {
                uint64_un out;
                for (unsigned int j = 0; j < 8; j++) {
                    if (i * 8 + j < sz) {
                        out.c_a[j] = line[i * 8 + j];
                    } else {
                        out.c_a[j] = ' ';
                    }
                }
                msg_buff[offt++] = out.d;
            }
            // save the length of current line in bytes
            len_buff[msg_nm++] = sz;
            return 0;
        }
    }
```



```cpp
    // Header for reEngine
    #include "re_engine_kernel.hpp"
    // Header for reading log file as std::string
    #include <iostream>
    #include <fstream>
    #include <string.h>

    // Total number of configuration blocks
    // leave 2 blocks for configuration header
    unsigned int cfg_nm = 2 + instr_num;
    // Message buffer (64-bit width for full utilizing the 2 memory ports of BRAMs)
    uint64_t* msg_buff = aligned_alloc<uint64_t>(MAX_MSG_SZ);
    // Length buffer
    uint16_t* len_buff = aligned_alloc<uint16_t>(MAX_MSG_NM);
    // Append bit-set map to the tail of instruction list
    for (unsigned int i = 0; i < cclass_num * 4; i++) {
        uint64_t tmp = bitset[i * 2 + 1];
        tmp = tmp << 32;
        tmp += bitset[i * 2];
        cfg_buff[cfg_nm++] = tmp;
    }
    // Set configuration header accordingly
    typedef union {
        struct {
            uint32_t instr_nm;
            uint16_t cc_nm;
            uint16_t gp_nm;
        } head_st;
        uint64_t d;
    } cfg_info;
    cfg_info cfg_h;
    cfg_h.head_st.instr_nm = instr_num;
    cfg_h.head_st.cc_nm = cclass_num;
    cfg_h.head_st.gp_nm = cpgp_num;
    cfg_buff[0] = cfg_nm;
    cfg_buff[1] = cfg_h.d;
    // String of each line in the log
    std::string line;
    // We provide a 5k line apache log
    std::ifstream log_file(log_data/access_5k.log);
    if (log_file.is_open()) {
        // Read the apache log line-by-line
        while (getline(log_file, line)) {
            if (line.size() > 0) {
                if (writeOneLine(msg_buff, len_buff, offt, msg_nm, line) != 0) {
                    return -1;
                }
            }
        }
        // Set the header of message buffer (number of message blocks in 64-bit)
        msg_buff[0] = offt;
        // Set the header of length buffer (concatenate the first 2 blocks, it presents the total number of messages in msg_buff)
        len_buff[0] = msg_nm / 65536;
        len_buff[1] = msg_nm % 65536;
    } else {
        printf("Opening input log file failed.\n");
        return -1;
    }
    // Result buffer
    uint32_t* out_buff = aligned_alloc<uint32_t>((cpgp_num + 1) * msg_nm);
    // Call reEngine
    reEngineKernel(reinterpret_cast<ap_uint<64>*>(cfg_buff), reinterpret_cast<ap_uint<64>*>(msg_buff), reinterpret_cast<ap_uint<16>*>(len_buff), reinterpret_cast<ap_uint<32>*>(out_buff));
```

The match flag and offset addresses for each capturing group are presented in ``out_buff`` with the format shown in the figure below:

**out_buff**

<img align="center" src="regex-vm/data_analytics/images/outbuff_format.png" alt="drawing" width="300"/>

Profiling
=========

The hardware resource utilizations of reEngine (the one given in L2 test as an example on U200) is shown in the table below (performance optimized version at **FMax = 200MHz**).

| **Item**           |  **LUT**   |   **REG**    |  **BRAM**  | **URAM**   | **DSP**    |
| reEngine       | 499292 |  341196   | 792    | 576    |  36    |
| (U200)         | 54.94% |  17.27%   | 70.53% | 60.00% | 0.53%  |

Number of PUs on each SLR is listed in the table below:

| **SLR**     | **Number of PUs** |
| 0       | 5             |
| 1       | 2             |
| 2       | 5             |

Therefore, the kernel throughput should be:

**Throughput = 12 * 387 MB/s = 4.64 GB/s**