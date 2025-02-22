=========================================
Xilinx PairHMM Genomics
=========================================

PairHMM Algorithm resides in ``L2/tests/pairhmm`` directory.

Xilinx PairHMM Algorithm is FPGA based implementation of
standard PairHMM. Xilinx implementation of PairHMM application is 
aimed for achieving high throughput. This Xilinx PairHMM 
application is developed and tested on Xilinx Alveo U200. 


Results
-------

Resource Utilization 
~~~~~~~~~~~~~~~~~~~~~

Table below presents resource utilization of Xilinx Smithwaterman Genomics
kernels. The final Fmax achieved is 300MHz 

============= ====== ====== ======  ==== 
Flow           LUT    DSP    BRAM   URAM 
============= ====== ====== ======  ====  
PairHMM        253K   2K      95     56
============= ====== ====== ======  ====

Performance Data
~~~~~~~~~~~~~~~~

Table below presents kernel throughput achieved for a single compute
unit. 

====================== =========================
Topic                      Results
====================== =========================
Genomics Throughput         11GCUPS
====================== =========================

Note: Overall throughput can still be increased with multiple compute
units.

Software & Hardware
-------------------

::

     Software: Xilinx Vitis 2021.2
     Hardware: xilinx_u250_gen3x16_xdma_3_1_202020_1(Xilinx Alveo U250)

Executable Usage
----------------
 
To execute pairHMM kernel  : ``./build/xil_pairhmm ./build/xclbin_<xsa_name>_<TARGET mode>/<___.xclbin>``
 
           

      - Note: Default arguments are set in Makefile
      
