# HW 3 Completed
## Question 1
My testbench:
~~~systemverilog
module tb();
  logic a, b, clk;
  logic [1:0] u, v;
  
  DUT dut (.*);  // calls module
  
  // Clock generation
  initial begin
    forever #5 clk = ~clk;
  end
  
  // Stimulus and printing
  initial begin
    clk = 0;
    a = 0; b = 0; #10;
    // Test 4 input combinations
    a = 0; b = 0; #10;
    a = 0; b = 1; #10;
    a = 1; b = 0; #10;
    a = 1; b = 1; #10;
    
    $finish;
  end
  
  always @(posedge clk)begin
  #10;
    $strobe("%0t: a=%b b=%b | _u=%b _v=%b | u=%b v=%b", $time, a, b, dut.blk._u, dut.blk._v, u, v);
  end
endmodule
~~~
Results: <br>
![image](https://github.com/user-attachments/assets/6584062d-ed8e-4c5f-9cdb-d31ba7372f8b)

## Question 2
My testbench:
~~~systemverilog
module tb();
  logic a, b, clk;
  logic [1:0] u, v;
  
  DUT dut (.*);  // calls module
  
  // Clock generation
  initial begin
    forever #5 clk = ~clk;
  end
  
  // Stimulus and printing
  initial begin
    clk = 0;
    a = 0; b = 0; #10;
    // Test 4 input combinations
    a = 0; b = 0; $display("%0t: a=%b b=%b | _u=%b _v=%b | u=%b v=%b", $time, a, b, dut.blk._u, dut.blk._v, u, v);#10;
    a = 0; b = 1;$display("%0t: a=%b b=%b | _u=%b _v=%b | u=%b v=%b", $time, a, b, dut.blk._u, dut.blk._v, u, v); #10;
    a = 1; b = 0; $display("%0t: a=%b b=%b | _u=%b _v=%b | u=%b v=%b", $time, a, b, dut.blk._u, dut.blk._v, u, v);#10;
    a = 1; b = 1; $display("%0t: a=%b b=%b | _u=%b _v=%b | u=%b v=%b", $time, a, b, dut.blk._u, dut.blk._v, u, v);#10;
    
    $finish;
  end
  
  always @(posedge clk)begin
  #10;
    $strobe("%0t: a=%b b=%b | _u=%b _v=%b | u=%b v=%b", $time, a, b, dut.blk._u, dut.blk._v, u, v);
  end
endmodule
~~~
Results: <br>
![image](https://github.com/user-attachments/assets/c0c33691-ff85-477b-8917-777aedcea169)

## Question 3
My testbench:
~~~systemverilog
module tb();
  logic a, b, clk;
  logic [1:0] u, v;
  
  DUT dut (.*);  // calls module
  
  // Clock generation
  initial begin
    forever #5 clk = ~clk;
  end
  
  // Stimulus and printing
  initial begin
    clk = 0;
    $monitor("%0t: a=%b b=%b | _u=%b _v=%b | u=%b v=%b", $time, a, b, dut.blk._u, dut.blk._v, u, v);
    a = 0; b = 0; #10;
    // Test 4 input combinations
    a = 0; b = 0; $display("%0t: a=%b b=%b | _u=%b _v=%b | u=%b v=%b", $time, a, b, dut.blk._u, dut.blk._v, u, v);#10;
    a = 0; b = 1;$display("%0t: a=%b b=%b | _u=%b _v=%b | u=%b v=%b", $time, a, b, dut.blk._u, dut.blk._v, u, v); #10;
    a = 1; b = 0; $display("%0t: a=%b b=%b | _u=%b _v=%b | u=%b v=%b", $time, a, b, dut.blk._u, dut.blk._v, u, v);#10;
    a = 1; b = 1; $display("%0t: a=%b b=%b | _u=%b _v=%b | u=%b v=%b", $time, a, b, dut.blk._u, dut.blk._v, u, v);#10;
    
    $finish;
  end
 
endmodule
~~~
Results: <br>
![image](https://github.com/user-attachments/assets/e0f18871-192f-4c06-b0bd-7ba42862387f)



