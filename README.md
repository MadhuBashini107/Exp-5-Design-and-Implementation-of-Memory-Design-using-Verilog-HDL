# Exp-5-Design-and-Simulate the-Memory-Design-using-Verilog-HDL
#Aim
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
// Verilog code
```
`timescale 1ns / 1ps
module ram_4kb (clk,we,addr,din,dout);
    input clk;
    input we;                  // Write Enable
    input [11:0] addr;         // 12-bit Address = 4096 locations
    input [7:0] din;           // Data input
    output reg [7:0] dout;      // Data output


reg [7:0] mem [0:4095];        // 4KB = 4096 x 8-bit

always @(posedge clk) 
begin
    if (we)
        mem[addr] <= din;      // Write operation
        dout <= mem[addr];         // Read operation
end
endmodule

// Test bench
module tb_ram_4kb;
reg clk;
reg we;
reg [11:0] addr;
reg [7:0] din;
wire [7:0] dout;
integer i;
ram_4kb dut (clk,we,addr,din,dout);
initial begin
    clk = 0;
    forever #5 clk = ~clk;
end
initial 
   begin
    we = 0;
    addr = 0;
    din = 0;
    #10;

       for (i = 0; i < 20; i = i + 1) 
        begin
        @(posedge clk);
        addr = $random % 4096;   
        din  = $random % 256;
        we   = 1;
        @(posedge clk);
        we   = 0;
        end
$finish;
end
endmodule
```
// output Waveform
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/a2607a44-93c0-4eb5-9f7a-0164377ad367" />

# ROM
 // write verilog code for ROM using $random
 `timescale 1ns / 1ps
module rom_memory (
    input wire clk,
    input wire write_enable,   
    input wire [11:0] address, 
    input wire [7:0] data_in,  
    output reg [7:0] data_out  
);
    reg [7:0] rom[0:4095];

    always @(posedge clk) begin
        if (write_enable) begin
            
            rom[address] <= data_in;
        end
        data_out <= rom[address];
    end
endmodule

 // Test bench
module rom_memory_tb;

    
    reg clk;
    reg write_enable;
    reg [11:0] address;
    reg [7:0] data_in;
    wire [7:0] data_out;
    rom_memory uut (
        .clk(clk),
        .write_enable(write_enable),
        .address(address),
        .data_in(data_in),
        .data_out(data_out)
    );
    always #5 clk = ~clk;  
    initial begin
        clk = 0;
        write_enable = 0;
        address = 0;
        data_in = 0;
        #10 write_enable = 1; address = 12'd0; data_in = 8'hA5;  
        #10 write_enable = 1; address = 12'd1; data_in = 8'h5A;  
        #10 write_enable = 1; address = 12'd2; data_in = 8'hFF;  
        #10 write_enable = 1; address = 12'd3; data_in = 8'h00;  
        #10 write_enable = 0; address = 12'd0;
        #10 address = 12'd1;
        #10 address = 12'd2;
        #10 address = 12'd3;
        #10 $stop;
    end
    initial begin
        $monitor("Time = %0t | Write Enable = %b | Address = %h | Data In = %h | Data Out = %h", 
                 $time, write_enable, address, data_in, data_out);
    end

endmodule

// output Waveform
<img width="1917" height="1079" alt="image" src="https://github.com/user-attachments/assets/63a03a35-3009-43ea-adfa-70b24d7dcef0" />

 # FIFO
 // write verilog code for FIFO

 `timescale 1ns / 1ps
  module fifosync_exp (clk,rst,wr_en,rd_en,data_in,full,data_out,empty,count);
    input clk;
    input rst;
    input wr_en;
    input [7:0] data_in;
    input rd_en;
    output reg full;
    output reg [7:0] data_out;
    output reg empty;
    output reg [4:0] count;
    reg [7:0] mem [0:15];
    reg [3:0] wr_ptr;
    reg [3:0] rd_ptr;
always @(posedge clk) 
   begin
        if (rst) 
          begin
            wr_ptr   <= 0;
            rd_ptr   <= 0;
            count    <= 0;
            data_out <= 0;
            full     <= 0;
            empty    <= 1;
          end 
      else 
        begin
            full  <= (count == 16);
            empty <= (count == 0);
if (wr_en && !full) 
     begin
            mem[wr_ptr] <= data_in;
            wr_ptr <= wr_ptr + 1'b1;
      end

  if (rd_en && !empty) 
      begin
                data_out <= mem[rd_ptr];
                rd_ptr <= rd_ptr + 1'b1;
      end

case ({wr_en && !full, rd_en && !empty})
                2'b10: count <= count + 1'b1;
                2'b01: count <= count - 1'b1;
                default: count <= count;
endcase
 full  <= (count == 16);
            empty <= (count == 0);
        end
    end
endmodule

 // Test bench
 
    module fifosync_exp_tb;
    reg clk;
    reg rst;
    reg wr_en;
    reg rd_en;
    reg [7:0] data_in;
    wire [7:0] data_out;
    wire full;
    wire empty;
    wire [4:0] count;

   fifosync_exp uut (clk,rst,wr_en,rd_en,data_in,full,data_out,empty,count );

   always #5 clk = ~clk;
   initial 
   begin
        clk = 0;
        rst = 1;
        wr_en = 0;
        rd_en = 0;
        data_in = 8'h10;            
        #10;
        rst = 0;                   
        #10;
                                 

  repeat (5) 
     begin
            @(posedge clk);
            wr_en = 1;
            data_in = data_in + 1;
     end
        @(posedge clk);
        wr_en = 0;
        repeat (3) 
          begin
            @(posedge clk);
            rd_en = 1;
        end
        @(posedge clk);
        rd_en = 0;
        #20;
        $finish;
        end
        endmodule
```
// output Waveform


<img width="1290" height="158" alt="image" src="https://github.com/user-attachments/assets/ee9a69ae-72ca-4f63-8ca4-65918b3257c0" />

```
# Conclusion
The RAM, ROM, FIFO memory with read and write operations was designed and successfully simulated using Verilog HDL. The testbench verified both the write and read functionalities by simulating the memory operations and observing the output waveforms. The experiment demonstrates how to implement memory operations in Verilog, effectively modeling both the reading and writing processes.
 
 

