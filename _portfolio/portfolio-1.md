---
title: "FIR Function Accelerator Using FPGA"
excerpt: "My first project on my personal Pynq-Z2 board with a Zynq-7000FPGA"
collection: portfolio
---

For this project we are using the Xilinx Vivado ISE and Jupyter Notebooks in order to enable hardware accleration for the Finite Impulse Response signal processing function. The goal of this project was to introduce myself to FPGAs and how to use them. I also wanted to dive deeper into how hardware acceleration occurs and learn how to program an FPGA to interface with software. By utilizing Xilinx's Vivado, I learned how to design and interact with professionally made IPs and incorporate them into a block diagram used to program an FPGA. Specific to the PYNQ-Z2, I think, I learned how to convert this file into an overlay that is used in Python to actually accelerate the function

![Block diagram](/images/Portfolio/firFilter/blockdiagram.png)

This is the block diagram for the FIR accelerator. Luckily, the Vivado platform had a premade FIR function IP so I didn't need to design the Verilog to represent this block. The platform is also pretty nice as it makes a lot of the connections between modules automatically and will even generate the needed modules if there are any that seem to be missing.

![Original Waveform](/images/Portfolio/firFilter/ogwaveform.png)

This is a diagram of the original FIR function working on a randomized sample input. As we can see, the purpose of the FIR filter is to do a bunch of convolutions between the input and previous input which is why our FIR output looks similar to the input signal. Essentially its cleaning it up. This took 0.0764... seconds to execute. How much faster can we get?


```
from pynq import Overlay 
import pynq.lib.done
import os 

overlay = Overlay('/home/xilinx/pynq/overlays/fir_accel/fir_accel.bit')
dma = overlay.filter.fir_dma

/home/xilinx/jupyter_notebooks

from pynq import allocate
import numpy as np 

in_buffer = allocate(shape=(n,), dtype=np.int32)
out_buffer = allocate(shape=(n,), dtype=np.int32)

np.copyto(in_buffer, samples)

import time 
start_time = time.time()
dma.sendchannel.transfer(in_buffer)
dma.recvchannel.transfer(out_buffer)

dma.sendchannel.wait()
dma.recvchannel.wait()

stop_time=time.time()
hw_exec_time = stop_time - start_time
print("Hardware Execution Time: ", hw_exec_time)
print("Hardware Acceleration Factor: ", sw_exec_time / hw_exec_time)
plot_to_notebook(t, samples, 1000, out_signal = out_buffer)

in_buffer.close()
out_buffer.close()
```

This is the actual code used to incorporate our block design onto the FPGA as an overlay that will be used to perform the FIR filter on the same input data set and using the same coefficients. PYNQ makes this pretty easy as we can directly interact with the FPGA using the PYNQ libraries.

![Hardware Waveform](/images/Portfolio/firFilter/hardwarewaveform.png)

This is the same graph we received earlier but this time it was generated using our hardware accelerated implementation of the FIR filter. As we can see, the execution time was almost 14x faster than our original execution time. On a small scale these implications are probably not too useful, but in real world applications where we aren't capped at 2000 samples and are instead using a number approaching infinity (which we probably wouldn't use an FIR filter for), these difference in execution time are probably very significant
