
import uvm_pkg::*;
`include "uvm_macros.svh"
module arbiter;
bit[15:0] req, grnt;
bit clk;
initial forever #5 clk = !clk;

generate for (genvar i = 0, i<=15; i++)
begin
    property p_arbiter;
    bit[16:0] v;
    (req[i]==1'b1, v=0, v[i+1]=1'b1) ##0 req < v |->
    grnt[i]==1'b1 ##0 $onehot(grnt);
    endproperty : p_arbiter
    ap_arbiter : assert property(@(posedge clk) p_arbiter);
end
endgenerate
ap_zero_req0 : assert property(@(posedge clk) req==0 |=> grnt==0);
ap_zero_req1 : assert property(@(posedge clk) req==1 |=> grnt==1);

always @(posedge clk)
if(!randomize(req, grnt) with {req < 15; $onehot(grnt) == 1'b1;}) // 이 onehot은 이미 있는 1을 바꿔주는 것이 아닌 1이
endmodule // 더 많이 존재한다면 randomize 실패




// 위의 코드는 오류 존재

import uvm_pkg::*;
`include "uvm_macros.svh"

module arbiter;

rand bit [15:0] req, grnt;
bit clk;

initial clk = 0;
initial forever #5 clk = !clk;

// ----------------------------
// Arbiter priority assertion
// ----------------------------
generate
  for (genvar i = 0; i < 16; i++) begin

    property p_arbiter;
      bit [16:0] v;

      (req[i] == 1'b1, v = 0, v[i+1] = 1'b1) ##0 (req < v)
      |-> (grnt[i] == 1'b1 ##0 $onehot(grnt));

    endproperty

    ap_arbiter : assert property(@(posedge clk) p_arbiter);

  end
endgenerate


// ----------------------------
// No request → no grant
// ----------------------------
ap_zero_req :
assert property(@(posedge clk)
  req == 0 |-> grnt == 0
);


// ----------------------------
// Random stimulus
// ----------------------------
always @(posedge clk) begin

  if (!randomize(req, grnt) with {
        $onehot0(grnt);     // grant는 0 또는 one-hot
      })
  begin
    $error("Randomization failed");
  end

end

endmodule
