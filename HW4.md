~~~systemverilog
`timescale 1ns / 1ps

module top(
    input wire CLK100MHZ,
    input wire [3:0] SW,        // Switches for movement delta
    input wire BTNC,            // Center button
    input wire BTNR,            // Reset button
    output wire [7:0] LED       // LED outputs for candle states
);

    // Internal Signals
    wire extinguishFlag;         // From extinguisher
    wire [2:0] extPosition;      // Position to extinguish
    wire [2:0] positionQ;        // Current igniter position
    wire [2:0] positionD;        // Next position (igniter logic)
    wire [7:0] candle_state;     // Candle states
    wire debounced_BTNC;         // Debounced signal for BTNC

    // Debounce and One-Shot Module
    debounce_and_oneshot debounce(
        .out(debounced_BTNC),   // Debounced signal
        .btn(BTNC),             // Center button
        .clk_50MHz(CLK100MHZ),        // System clock
        .rst(~BTNR)             // Active-low reset
    );

    // Extinguisher Module
    extinguisher extinguish(
        .extinguish(extinguishFlag),
        .position(extPosition),
        .enable(debounced_BTNC),  // Triggered by debounced button press
        .clk(CLK100MHZ),
        .clr_n(BTNR)
    );

    // Igniter Module
    igniter ignite (
        .position_q(positionQ),
        .position_d(positionD),
        .delta(SW),               // Movement delta from switches
        .enable_move(debounced_BTNC),
        .sys_clk(CLK100MHZ),
        .clr_n(BTNR)
    );

    // Candle Controller Module
    candle_controller candle(
        .candle_state(candle_state),
        .pos_to_set(positionQ),   // Match position determines candle to light
        .set_enable(debounced_BTNC),
        .pos_to_clear(extPosition),
        .clear_enable(extinguishFlag),
        .sys_clk(CLK100MHZ),
        .clr_async(~BTNR)
    );
~~~

~~~systemverilog
`timescale 1ns / 1ps

module extinguisher(
    input logic enable,
    input logic clk,
    input logic clr_n,
    output logic [2:0] position,
    output logic extinguish 
);

logic [3:0] counter;  // 4 bits for 0-15

// Sequential logic 
always @(posedge clk, negedge clr_n) begin
    // Resets counter.
    if (!clr_n) begin
        counter <= 0;
    end
    // Adds to counter
    else begin
        counter <= counter + 1;
    end
end

// Combinatorial output logic
always_comb begin
    // Stores positions less than 8
    position = counter[2:0];
    // Extinguishes if less than 8 (50%) and enabled.
    extinguish = enable && (counter < 8);
end

endmodule

module igniter(
    output logic [2:0] position_q,
    output logic [2:0] position_d,
    input logic [3:0] delta,
    input logic enable_move,
    input logic sys_clk,
    input logic clr_n  
);

always_comb begin
    // Gets new postion if enabled.
    if (enable_move) begin
        position_d = (position_q + delta) % 8;
        end
    else begin
        position_d = position_q;
    end
end
// Sequential block
always@(posedge sys_clk, negedge clr_n) begin
    // Resets q
    if (!clr_n)
        position_q <= 3'b000;
    else
    // Stores new position
        position_q <= position_d;
end

endmodule

module candle_controller(
    input logic sys_clk,
    input logic clr_async,
    input logic clear_enable,
    input logic set_enable,
    input logic [2:0] pos_to_set,
    input logic [2:0] pos_to_clear,
    output logic [7:0] candle_state
    );

    always_ff @(posedge sys_clk, posedge clr_async) begin
    // Asynch active high clear
        if (clr_async) begin
        candle_state <= 0;
        end else begin
        // Set enable high, use postion to set bit high in candle state
            if (set_enable) begin
                candle_state[pos_to_set] <= 1'b1;
            end
            // Clear enable high, check for seat and clear not overlapping
            if (clear_enable && (!set_enable && (pos_to_clear == pos_to_clear))) begin
                candle_state[pos_to_clear] <= 1'b0;
            end
       end
     end
endmodule

module debounce_and_oneshot(
    output reg out,
    input btn,
    input clk_50MHz,
    input rst
    );

parameter MINWIDTH = 5000000; //how many cycles must the btn be pressed
parameter COUNTERWIDTH = 32;

reg [COUNTERWIDTH-1:0] counter;
//reg [COUNTERWIDTH-1:0] new_counter;

reg shot;

always @(posedge clk_50MHz, posedge rst) begin
  if (rst) begin
    counter <= 0;
    out <= 1'b0;  
    shot <= 1'b0;  
  end else begin
	   if (~btn) begin
		    counter <= 0;
		    out <= 1'b0;
          shot <= 1'b0;  
      end else if (counter!=MINWIDTH) begin
		    counter<=counter+1;
		    out <= 1'b0;
          shot <= 1'b0;  
      end else begin //we have reached MINWIDTH
			 counter<=counter;
		    if (shot == 0) begin
			   shot <= 1'b1;
      		 out<=1'b1;
			 end else begin 
			 	shot <= shot;
      		 out<=1'b0;
			 end
		end
		
	end
end //end always


endmodule

~~~
~~~systemverilog
`timescale 1ns / 1ps

module extinguisher(
    input logic enable,
    input logic clk,
    input logic clr_n,
    output logic [2:0] position,
    output logic extinguish 
);

logic [3:0] counter;  // 4 bits for 0-15

// Sequential logic 
always @(posedge clk, negedge clr_n) begin
    // Resets counter.
    if (!clr_n) begin
        counter <= 0;
    end
    // Adds to counter
    else begin
        counter <= counter + 1;
    end
end

// Combinatorial output logic
always_comb begin
    // Stores positions less than 8
    position = counter[2:0];
    // Extinguishes if less than 8 (50%) and enabled.
    extinguish = enable && (counter < 8);
end

endmodule

module igniter(
    output logic [2:0] position_q,
    output logic [2:0] position_d,
    input logic [3:0] delta,
    input logic enable_move,
    input logic sys_clk,
    input logic clr_n  
);

always_comb begin
    // Gets new postion if enabled.
    if (enable_move) begin
        position_d = (position_q + delta) % 8;
        end
    else begin
        position_d = position_q;
    end
end
// Sequential block
always@(posedge sys_clk, negedge clr_n) begin
    // Resets q
    if (!clr_n) begin
        position_q <= 3'b000;
        end
    else begin
    // Stores new position
        position_q <= position_d;
        end
end

endmodule

module candle_controller(
    input logic sys_clk,
    input logic clr_async,
    input logic clear_enable,
    input logic set_enable,
    input logic [2:0] pos_to_set,
    input logic [2:0] pos_to_clear,
    output logic [7:0] candle_state
    );

    always_ff @(posedge sys_clk, posedge clr_async) begin
    // Asynch active high clear
        if (clr_async) begin
        candle_state <= 0;
        end else begin
        // Set enable high, use postion to set bit high in candle state
            if (set_enable) begin
                candle_state[pos_to_set] <= 1'b1;
            end
            // Clear enable high, check for seat and clear not overlapping
            if (clear_enable && (!set_enable && (pos_to_clear == pos_to_clear))) begin
                candle_state[pos_to_clear] <= 1'b0;
            end
       end
     end
endmodule

module debounce_and_oneshot(
    output reg out,
    input btn,
    input clk_50MHz,
    input rst
    );

parameter MINWIDTH = 5000000; //how many cycles must the btn be pressed
parameter COUNTERWIDTH = 32;

reg [COUNTERWIDTH-1:0] counter;
//reg [COUNTERWIDTH-1:0] new_counter;

reg shot;

always @(posedge clk_50MHz, posedge rst) begin
  if (rst) begin
    counter <= 0;
    out <= 1'b0;  
    shot <= 1'b0;  
  end else begin
	   if (~btn) begin
		    counter <= 0;
		    out <= 1'b0;
          shot <= 1'b0;  
      end else if (counter!=MINWIDTH) begin
		    counter<=counter+1;
		    out <= 1'b0;
          shot <= 1'b0;  
      end else begin //we have reached MINWIDTH
			 counter<=counter;
		    if (shot == 0) begin
			   shot <= 1'b1;
      		 out<=1'b1;
			 end else begin 
			 	shot <= shot;
      		 out<=1'b0;
			 end
		end
		
	end
end //end always


endmodule
~~~
