
# SIMULATION FILE

## Control signals

In this file you can introduce all the information regarding the simulation.

First, introduce some information about the control signals for a correct simulation:

	N_RESET_CYCLES = `1` -- Number of reset clock cycles. By default always 1. Select the TOTAL number of cycles, including the first reset

	N_LOAD__CYCLES = `1` -- If the design is a serial implementation or just a pipeline (where there is no "done" signal), this tells the tool how many initial cycles to make the loading (and then the number of cycles at the end to capture complete output) or how many cycles should the pipeline run respectively. Even if you do not have a "load" signal but you still have a serial design that needs some 

	N_SHARES = `1` -- Specify number of shares in case it is a combined countermeasure

Second, introduce number of random inputs to run in the experiment

	N_TRACES = `0` -- Select the number of random inputs to be randomly generated by the tool (if you do not specify your own inputs)

Finally, if you want to provide the inputs yourself, leave N_TRACES = 0 (the program stops reading the file in the case =0) and introduce the inputs below (you can see the format of the inputs at the bottom of this file together with some examples).


### Inputs
### End Inputs

---

### Format
* When using Synopsys, inputs will follow the exact same order as given in your 
module, including the bit order (big/small endian). However, if using Yosys, variables 
will follow the same order as declared in the module, but all of them are small endian 
(right-most bit index = 0)

* Control signals: rst, clk, and load are not counted in the inputs assignment.

* Inputs may be given in binary of hexadecimal. If in hexadecimal, they should be 
preceded by "x". Example: x01DAAE;. Note that the input test vector must end with ";".
However, if input length is not multiple of 4, it must be provided in binary.
All the test vectors must have the same format (either binary or hexadecimal).

* In a serial implementation, even though inputs are provided in different cycles, 
it is still considered a single test vector and it should be given in full length. 
Then, the cycles needed to input and output are specified in Load__cycles.

* IMPORTANT: for serial implementations with additional inputs, like random variables, or
other public or secret variables. Even if they do not change value along the cycles 
needed for feeding the input, you need to specify the value in the input test vector 
as many times as loading cycles. For each loading cycle, the tool chops the full input 
test vector into chunks as long as the number of inputs, and every cycle feeds one of 
the chunks in order. See some example below.

* You cannot specify different inputs with different clk cycles simulations.

* To simulate a pipeline design, which has no ready signal, it is necessary to 
specify the number of cycles you want to simulate the circuit. You can do this by 
selecting the number of load cycles.
### Examples:


1. Round based implementation with two inputs: 
``` 
input [7:0] a;  
input [7:0] b;  
```
The inputs will receive values in the following order: a7, a6, ..., a0, b7, b6, ..., b0.  
N_LOAD__CYCLES = `1`
##### Inputs
xAAFF;
##### End Inputs


2. Serial full AES implementation without additional inputs  
input clk, rst, load;  
input [7:0] AESin;  
The total length of the input is 128 bits, which are input in 16 clk cycles 
in chunks of 8 bits. Note that control signals are ignored.  
N_LOAD__CYCLES = `16`  
##### Inputs
xAA  
FF  
AA  
FF  
AA  
FF  
AA  
FF  
AA  
FF  
AA  
FF  
AA  
FF  
AA  
FF; (This is a single test vector)  
xAAAAFFFFAAAAFFFFAAAAFFFFAAAAFFFF;
##### End Inputs


3. Serial implementation 2 shares AES with a public value P, which is needed only once 
at the input and does not change, and a random input R read at every loading cycle.  
```
input clk, rst, load;  
input [7:0] AESin;  
input [7:0] P;  
input [1:0] R;  
```
Per cycle we input 8+8+2 bits, so the total size of one input test vector is 288 
bits. Since it is multiple of 8, we can specify it in hexadecimal, but for the 
example we will show it in binary.  
N_LOAD__CYCLES = `16`
##### Inputs
111111110101010100  
111111110101010111  
111111110101010100  
111111110101010111  
111111110101010100  
111111110101010111  
111111110101010100  
111111110101010111  
111111110101010100  
111111110101010111  
111111110101010100  
111111110101010111  
111111110101010100  
111111110101010111  
111111110101010100;
##### End Inputs
This means that the full length AES input is all 1's, the public value 
is 01010101, and the random variable alternates between 00 and 11.


4. Three stages pipeline, to which we want to feed three different inputs and 
simulate until we see the output produced by the last input.
```
input [3:0] pipeIn;
```
With the load cycles variable, we can decide how many cycles to simulate 
the circuit.  
N_LOAD__CYCLES = `6`  
##### Inputs
1010  
0101  
1111  
0000  
0000  
0000;
##### End Inputs


