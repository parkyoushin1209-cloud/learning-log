module m; int d2[]; endmodule : m
module top; m m1(); endmodule : top
checker mchk1(input event clk_ev,
              output logic d_out,
              output untyped ot);
wire w1, w2;
int dyn[];
int a_size, i=0, k1, k2, k3, k4, k5;
typedef logic [31:0] 132_type;
typedef logic [16:0] 116_type;
logic reset_n=1'b1, write=0;
116_type mem_array[132_type];
132_type addr=0, asize;
116_type wdata, rdata;

assign w1=!write;
assign k5=k2;
always @clk_ev begin
k1 <= top.m1.d2[0];
if(reset_n==1'b0) mem_array.delete;
else if (write) mem_array[addr] = wdata;
if(mem_array.exists(addr)) rdata <= mem_array[addr];
asize <= mem_array.size;
end

initial begin
    dyn = new[5];
    foreach(dyn[j])
    dyn[j] = j;
end : dy1

always_ff @clk_ev begin : a1
automatic int v;
v <= 0;
for(int z=0; z<32; z++) begin
    addr[i] =1'b1;
end
end
task tsk();
...
endtask : tsk

initial begin
    fork
        begin
            tsk();
        end
        begin
            k1 <= 1;
            k2 <= 2;
        end
    join
end
endchecker : mchk1
