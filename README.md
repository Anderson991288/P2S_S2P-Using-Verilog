# P2S_S2P-Using-Verilog


## 32-bit Parallel to Serial RTL Code 

1. Open Vivado project

   ![image](https://github.com/user-attachments/assets/181333cf-1d8b-4634-867f-32dc3f631142)


2. Code

   This design outputs each bit of the 32-bit parallel data sequentially according to the enable signal in every clock cycle. When the counter reaches 32, it restarts.
   
```
module ParallelToSerial (
    input wire clk,           // Clock signal
    input wire rst,           // Reset signal
    input wire enable,        // Enable signal
    input wire [31:0] pdata,  // 32-bit parallel data input
    output reg sdata          // 1-bit serial data output
);

    reg [4:0] bit_counter;    // Counter to track the bit position (0 to 31)

    always @(posedge clk or posedge rst) begin
        if (rst) begin
            bit_counter <= 0;
            sdata <= 0;
        end else if (enable) begin
            if (bit_counter < 32) begin
                sdata <= pdata[bit_counter]; // Output the current bit
                bit_counter <= bit_counter + 1;
            end else begin
                bit_counter <= 0; // Reset counter after 32 bits
            end
        end
    end
endmodule
```

3.Testbench

This testbench will simulate different scenarios to verify the correctness of the design and monitor the serial data output during the simulation.

```
`timescale 1ns / 1ps

module tb_ParallelToSerial;

    reg clk;
    reg rst;
    reg enable;
    reg [31:0] pdata;
    wire sdata;

    // Instantiate the ParallelToSerial module
    ParallelToSerial uut (
        .clk(clk),
        .rst(rst),
        .enable(enable),
        .pdata(pdata),
        .sdata(sdata)
    );

    // Clock generation
    initial begin
        clk = 0;
        forever #5 clk = ~clk; // 10ns period clock
    end

    // Testbench logic
    initial begin
        // Initialize signals
        rst = 1;
        enable = 0;
        pdata = 32'h0;

        // Apply reset
        #20;
        rst = 0;

        // Test case 1: Send 32-bit parallel data
        #10;
        pdata = 32'hA5A5A5A5; // Example data
        enable = 1;

        // Wait for the data to be serialized
        #320; // 32 bits * 10ns per bit = 320ns

        // Disable enable signal
        enable = 0;
        pdata = 32'h0;

        // Test case 2: Another 32-bit parallel data
        #30;
        pdata = 32'h5A5A5A5A; // Example data
        enable = 1;

        // Wait for the data to be serialized
        #320;

        // End simulation
        $stop;
    end

    // Monitor the serial data output
    initial begin
        $monitor("At time %t, serial data = %b", $time, sdata);
    end

endmodule
```

4.Simulation result

![IMG_1588](https://github.com/user-attachments/assets/c1b1e0dd-2c6b-43b9-b1d8-ba6601fc941c)


![IMG_1590](https://github.com/user-attachments/assets/32955c7c-c0cc-4085-bfa8-2ac55e5dbdf4)


## Serial to Parallel RTL Code 
1.Open Vivado project

![image](https://github.com/user-attachments/assets/7295d070-73f7-48ec-8f99-a8d41f18dffe)


2. SIPO Shift Register

   A Serial-In Parallel-Out (SIPO) shift register is a common implementation of serial to parallel conversion.

   This shift register takes serial input and converts it into a 32-bit parallel output.

```
module s2p(clk, in, rst, pout);
  input clk, rst;
  input in;
  output reg [31:0] pout;
 
  always @ (posedge clk, posedge rst)
    begin
      if (rst)
        pout <= 0;
      else 
        pout <= {in, pout[31:1]};
    end 
endmodule
```

3.Testbench

```
module s2p_tb;

  // Testbench signals
  reg clk;
  reg rst;
  reg in;
  wire [31:0] pout;

  // Instantiate the SIPO module
  s2p uut (
    .clk(clk),
    .rst(rst),
    .in(in),
    .pout(pout)
  );

  // Clock generation
  always begin
    #5 clk = ~clk; // Toggle clock every 5 time units
  end

  // Test sepoutuence
  initial begin
    // Initialize signals
    clk = 0;
    rst = 0;
    in = 0;

    // Apply reset
    #10 rst = 1;
    #10 rst = 0;

    // Shift in 32 bits of data
    repeat (32) begin
      #10 in = $random % 2; // Generate random 0 or 1 for input
    end

    // Finish the simulation
    #100;
    $finish;
  end

  // Monitor the signals
  initial begin
    $monitor("Time: %0t | rst = %b | in = %b | pout = %b", $time, rst, in, pout);
  end

endmodule
```

4.Simulation result

![螢幕擷取畫面 2024-07-20 132527](https://github.com/user-attachments/assets/e84b62fd-73f6-47c7-a04e-28ea8ea10cc2)












