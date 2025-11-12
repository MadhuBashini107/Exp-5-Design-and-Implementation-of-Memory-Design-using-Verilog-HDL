# Exp-5-Design-and-Simulate the-Memory-Design-using-Verilog-HDL
# Aim
To design and simulate a RAM,ROM,FIFO using Verilog HDL, and verify its functionality through a testbench in the Vivado 2023.1 environment.
Apparatus Required
Vivado 2023.1
Procedure
1. Launch Vivado 2023.1
Open Vivado and create a new project.
2. Design the Verilog Code
Write the Verilog code for the RAM,ROM,FIFO
3. Create the Testbench
Write a testbench to simulate the memory behavior. The testbench should apply various and monitor the corresponding output.
4. Create the Verilog Files
Create both the design module and the testbench in the Vivado project.
5. Run Simulation
Run the behavioral simulation to verify the output.
6. Observe the Waveforms
Analyze the output waveforms in the simulation window, and verify that the correct read and write operation.
7. Save and Document Results
Capture screenshots of the waveform and save the simulation logs. These will be included in the lab report.

# Code
# RAM

## Code ##
```verilog
module ram (
    input clk,
    input rst,
    input en,
    input [7:0] datain,
    input [9:0] address,
    output reg [7:0] dataout
);

    reg [7:0] mem [1023:0];

    always @(posedge clk) begin
        if (rst) begin
            dataout <= 8'b0;
        end
        else if (en) begin
            mem[address] <= datain;
        end
        else begin
            dataout <= mem[address];
        end
    end
endmodule
```


## Test bench ##
```verilog
`timescale 1ns/1ps
module ram_tb;

    reg clk;
    reg rst;
    reg en;
    reg [7:0] datain;
    reg [9:0] address;
    wire [7:0] dataout;

    ram DUT (
        .clk(clk),
        .rst(rst),
        .en(en),
        .datain(datain),
        .address(address),
        .dataout(dataout)
    );

    initial begin
        clk = 0;
        forever #5 clk = ~clk;
    end

    initial begin
        rst = 1;
        en = 0;
        datain = 0;
        address = 0;
        #10;

        rst = 0;
        en = 1;
        address = 10'd5;
        datain = 8'd25;
        #10;

        en = 1;
        address = 10'd10;
        datain = 8'd50;
        #10;

        en = 0;
        address = 10'd5;
        #10;

        address = 10'd10;
        #10;

        $finish;
    end

endmodule
```

## Output Waveform ##
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/58facb32-64f8-4d22-8944-d1b920f7c4e8" />


# ROM
## Code ##
 ```verilog
module rom (
    input clk,
    input [9:0] address,
    output reg [7:0] dataout
);

    reg [7:0] mem [1023:0];
    integer i;

    initial begin
        for(i = 0; i < 1024; i = i + 1)
            mem[i] = $random;
    end

    always @(posedge clk) begin
        dataout <= mem[address];
    end
endmodule

 ```
 ## Test bench ##
 ```verilog
 `timescale 1ns/1ps
module rom_tb;

    reg clk;
    reg [9:0] address;
    wire [7:0] dataout;

    rom DUT (
        .clk(clk),
        .address(address),
        .dataout(dataout)
    );

    initial begin
        clk = 0;
        forever #5 clk = ~clk;
    end

    initial begin
        address = 10'd0;
        #10;

        address = 10'd1;
        #10;

        address = 10'd2;
        #10;

        address = 10'd3;
        #10;

        address = 10'd4;
        #10;

        $finish;
    end
endmodule

 ```
 

## Output Waveform ##

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/524a0526-478a-4038-a7de-a2942a867f27" />


 # FIFO
 ## Code ##
 ```verilog
 module fifo (
    input clk,
    input rst,
    input wr,
    input rd,
    input [7:0] datain,
    output reg [7:0] dataout,
    output full,
    output empty
);

    reg [7:0] mem [15:0];
    reg [3:0] wptr;
    reg [3:0] rptr;
    reg [4:0] count;

    assign full  = (count == 16);
    assign empty = (count == 0);

    always @(posedge clk) begin
        if(rst) begin
            wptr <= 0;
            rptr <= 0;
            count <= 0;
            dataout <= 0;
        end
        else begin
            if(wr && !full) begin
                mem[wptr] <= datain;
                wptr <= wptr + 1;
                count <= count + 1;
            end

            if(rd && !empty) begin
                dataout <= mem[rptr];
                rptr <= rptr + 1;
                count <= count - 1;
            end
        end
    end
endmodule
```
 
 ## Test bench ##
 ```verilog
`timescale 1ns/1ps
module fifo_tb;

    reg clk;
    reg rst;
    reg wr;
    reg rd;
    reg [7:0] datain;
    wire [7:0] dataout;
    wire full;
    wire empty;

    fifo DUT (
        .clk(clk),
        .rst(rst),
        .wr(wr),
        .rd(rd),
        .datain(datain),
        .dataout(dataout),
        .full(full),
        .empty(empty)
    );

    initial begin
        clk = 0;
        forever #5 clk = ~clk;
    end

    initial begin
        rst = 1; wr = 0; rd = 0; datain = 0; #10;
        rst = 0;

        wr = 1; datain = 8'd10; #10;
        datain = 8'd20; #10;
        datain = 8'd30; #10;
        wr = 0;

        rd = 1; #10;
        #10;
        rd = 0;

        wr = 1; datain = 8'd40; #10;
        wr = 0;

        rd = 1; #10;
        rd = 0;

        $stop;
    end
endmodule
```

## Output Waveform ##

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/3c35759b-c044-4206-9eb1-73dd6f3bf7a6" />


# Conclusion
The RAM, ROM, FIFO memory with read and write operations was designed and successfully simulated using Verilog HDL. The testbench verified both the write and read functionalities by simulating the memory operations and observing the output waveforms. The experiment demonstrates how to implement memory operations in Verilog, effectively modeling both the reading and writing processes.
 
 

