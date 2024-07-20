# P2S_S2P-Using-Verilog


## 32-bit Parallel to Serial RTL Code 

1. Open Vivado project

   ![image](https://github.com/user-attachments/assets/181333cf-1d8b-4634-867f-32dc3f631142)


3. Code

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



