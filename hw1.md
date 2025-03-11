#HW 1

Top wrapper:
~~~systemverilog
module top_wrapper
   (CLK100MHZ,
    SW,
    VGA_B,
    VGA_G,
    VGA_HS,
    VGA_R,
    VGA_VS);
  input CLK100MHZ;
  input [3:0]SW;
  output [3:0]VGA_B;
  output [3:0]VGA_G;
  output VGA_HS;
  output [3:0]VGA_R;
  output VGA_VS;

  wire CLK100MHZ;
  wire [3:0]SW;
  wire [3:0]VGA_B;
  wire [3:0]VGA_G;
  wire VGA_HS;
  wire [3:0]VGA_R;
  wire VGA_VS;

  top top_i
       (.CLK100MHZ(CLK100MHZ),
        .SW(SW),
        .VGA_B(VGA_B),
        .VGA_G(VGA_G),
        .VGA_HS(VGA_HS),
        .VGA_R(VGA_R),
        .VGA_VS(VGA_VS));
endmodule
~~~
Top module:
~~~systemverilog
(* CORE_GENERATION_INFO = "top,IP_Integrator,{x_ipVendor=xilinx.com,x_ipLibrary=BlockDiagram,x_ipName=top,x_ipVersion=1.00.a,x_ipLanguage=VERILOG,numBlks=5,numReposBlks=5,numNonXlnxBlks=0,numHierBlks=0,maxHierDepth=0,numSysgenBlks=0,numHlsBlks=0,numHdlrefBlks=5,numPkgbdBlks=0,bdsource=USER,synth_mode=Hierarchical}" *) (* HW_HANDOFF = "top.hwdef" *) 
module top
   (CLK100MHZ,
    SW,
    VGA_B,
    VGA_G,
    VGA_HS,
    VGA_R,
    VGA_VS);
  (* X_INTERFACE_INFO = "xilinx.com:signal:clock:1.0 CLK.CLK100MHZ CLK" *) (* X_INTERFACE_PARAMETER = "XIL_INTERFACENAME CLK.CLK100MHZ, CLK_DOMAIN top_CLK100MHZ, FREQ_HZ 100000000, FREQ_TOLERANCE_HZ 0, INSERT_VIP 0, PHASE 0.0" *) input CLK100MHZ;
  (* X_INTERFACE_INFO = "xilinx.com:signal:data:1.0 DATA.SW DATA" *) (* X_INTERFACE_PARAMETER = "XIL_INTERFACENAME DATA.SW, LAYERED_METADATA undef" *) input [3:0]SW;
  (* X_INTERFACE_INFO = "xilinx.com:signal:data:1.0 DATA.VGA_B DATA" *) (* X_INTERFACE_PARAMETER = "XIL_INTERFACENAME DATA.VGA_B, LAYERED_METADATA undef" *) output [3:0]VGA_B;
  (* X_INTERFACE_INFO = "xilinx.com:signal:data:1.0 DATA.VGA_G DATA" *) (* X_INTERFACE_PARAMETER = "XIL_INTERFACENAME DATA.VGA_G, LAYERED_METADATA undef" *) output [3:0]VGA_G;
  (* X_INTERFACE_INFO = "xilinx.com:signal:data:1.0 DATA.VGA_HS DATA" *) (* X_INTERFACE_PARAMETER = "XIL_INTERFACENAME DATA.VGA_HS, LAYERED_METADATA undef" *) output VGA_HS;
  (* X_INTERFACE_INFO = "xilinx.com:signal:data:1.0 DATA.VGA_R DATA" *) (* X_INTERFACE_PARAMETER = "XIL_INTERFACENAME DATA.VGA_R, LAYERED_METADATA undef" *) output [3:0]VGA_R;
  (* X_INTERFACE_INFO = "xilinx.com:signal:data:1.0 DATA.VGA_VS DATA" *) (* X_INTERFACE_PARAMETER = "XIL_INTERFACENAME DATA.VGA_VS, LAYERED_METADATA undef" *) output VGA_VS;

  wire CLK100MHZ;
  wire [3:0]SW;
  wire [3:0]VGA_B;
  wire [3:0]VGA_G;
  wire VGA_HS;
  wire [3:0]VGA_R;
  wire VGA_VS;
  wire vga_rectangle_1_0_blue;
  wire vga_rectangle_1_0_green;
  wire vga_rectangle_1_0_red;
  wire vga_sync_0_blank;
  wire [9:0]vga_sync_0_hcount;
  wire [9:0]vga_sync_0_vcount;

  top_bit_replicator_4x_0_0 bit_replicator_4x_0
       (.dIn(vga_rectangle_1_0_red),
        .dOut(VGA_R));
  top_bit_replicator_4x_0_1 bit_replicator_4x_1
       (.dIn(vga_rectangle_1_0_green),
        .dOut(VGA_G));
  top_bit_replicator_4x_0_2 bit_replicator_4x_2
       (.dIn(vga_rectangle_1_0_blue),
        .dOut(VGA_B));
  top_vga_rectangle_1_0_0 vga_rectangle_1_0
       (.SW(SW),
        .blank(vga_sync_0_blank),
        .blue(vga_rectangle_1_0_blue),
        .clk(CLK100MHZ),
        .green(vga_rectangle_1_0_green),
        .pos_h(vga_sync_0_hcount),
        .pos_v(vga_sync_0_vcount),
        .red(vga_rectangle_1_0_red));
  top_vga_sync_0_0 vga_sync_0
       (.blank(vga_sync_0_blank),
        .clk(CLK100MHZ),
        .hcount(vga_sync_0_hcount),
        .hsync(VGA_HS),
        .vcount(vga_sync_0_vcount),
        .vsync(VGA_VS));
endmodule
~~~
Bit replicator:
~~~systemverilog
module bit_replicator_4x(
    input dIn,
    output [3:0] dOut
    );
    
    assign dOut = {4{dIn}};
endmodule
~~~
VGA Rectangle:
~~~systemverilog
module vga_rectangle_1(
    output reg red,
    output reg green,
    output reg blue,
    input [9:0] pos_h,
    input [9:0] pos_v,
    input blank,
    input clk,
    input [3:0] SW
    );
      parameter WIDTH    =  20;
      parameter HIEGHT   = 100;
      parameter X_LEFT   = 320;
      parameter Y_BOTTOM = 240;

      //addinal intermediate logic signal wires
      wire flag_on_rect;   //high only when over rectangle

      wire[9:0] x,y;      //traditional cartesean coordinates, (left, bottom)=(0,0)

       //combinatorial logic to calculate x,y coordinate system
       assign x = pos_h;
       assign y = 480 - pos_v;
       //combinatorial logic to decide if present pixel is over a desired rectange region
       assign flag_on_rect = x >= (X_LEFT)           &&

                             x < (X_LEFT + WIDTH)   &&

                             y >= (Y_BOTTOM)         &&

                             y <  (Y_BOTTOM + HIEGHT);
                             
      //combinatorial logic and registers (seqential logic) that load on rising clock edge

      always @(posedge clk) begin
        red   <= ~flag_on_rect & blank;
        green <= flag_on_rect & blank;
        blue  <= ~flag_on_rect & blank;
           
            if (~SW[0] && ~SW[1] && ~SW[2] && SW[3]) begin
                red <= flag_on_rect & ~blank;
                blue <= flag_on_rect & ~blank;
           end 
           else if (~SW[0] && ~SW[1] && SW[2] && ~SW[3]) begin
                red <= flag_on_rect & ~blank;
                green <= flag_on_rect & ~blank;
           end
           else if (~SW[0] && SW[1] && ~SW[2] && ~SW[3]) begin
                blue <= flag_on_rect & ~blank;
                green <= flag_on_rect & ~blank;
           end
           else if (SW[0] && ~SW[1] && ~SW[2] && ~SW[3]) begin
                red <= flag_on_rect & ~blank;
           end
           else begin
               red  <= flag_on_rect & blank;
               green <= flag_on_rect & blank;
               blue  <= ~flag_on_rect & blank; 
           end
           end                                                                          
endmodule
~~~
VGA sync:
~~~systemverilog
module vga_sync(clk,hsync,vsync,hcount,vcount,pix_clk,blank);

   input clk;     // 50Mhz
   output hsync;
   output vsync;
   output [9:0] hcount, vcount;
   output 	pix_clk;
   output 	blank;
   
// pixel clock: 25Mhz = 40ns (clk/4)
    reg [1:0] pcount;        // used to generate pixel clock
    wire en = (pcount == 0);
    always @ (posedge clk) pcount <= pcount + 1 ;
    assign pix_clk = en;
   
   //****************************************************************
   //****************************************************************
   //***
   //***  Sync and Blanking Signals
   //***
   //****************************************************************
   //****************************************************************
   
   reg 		hsync,vsync,hblank,vblank;
   reg [9:0] 	hcount;      // pixel number on current line
   reg [9:0] 	vcount;	 // line number
   
   // horizontal: 794 pixels = 31.76us
   // display 640 pixels per line
   wire 	hsyncon,hsyncoff,hreset,hblankon;
   assign 	hblankon = en & (hcount == 639);    
   assign 	hsyncon = en & (hcount == 652-4);
   assign 	hsyncoff = en & (hcount == 746-4);
   assign 	hreset = en & (hcount == 793-4);
   
   wire 	blank =  (vblank | (hblank & ~hreset));    // blanking => black
   
   // vertical: 528 lines = 16.77us
   // display 480 lines
   wire 	vsyncon,vsyncoff,vreset,vblankon;
   assign 	vblankon = hreset & (vcount == 479);    
   assign 	vsyncon = hreset & (vcount == 492-4);
   assign 	vsyncoff = hreset & (vcount == 494-4);
   assign 	vreset = hreset & (vcount == 527-4);
   
   // sync and blanking
   always @(posedge clk) begin
      hcount <= en ? (hreset ? 0 : hcount + 1) : hcount;
      hblank <= hreset ? 0 : hblankon ? 1 : hblank;
      hsync <= hsyncon ? 0 : hsyncoff ? 1 : hsync;   // hsync is active low
      
      vcount <= hreset ? (vreset ? 0 : vcount + 1) : vcount;
      vblank <= vreset ? 0 : vblankon ? 1 : vblank;
      vsync <= vsyncon ? 0 : vsyncoff ? 1 : vsync;   // vsync is active low
   end
   
endmodule
~~~
