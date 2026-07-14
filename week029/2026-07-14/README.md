195일차 학습한 내용
1. RAL 레지스터 모델링 익히기
2. AI를 통해 문제 생성 후 모델링해보기

내일 학습할 내용
어댑터, 프레딕터 학습 
통합 방법 학습
기본 API 사용 연습

결과물

class mode_field extends uvm_reg_field;
`uvm_object_utils(mode_field)

function new(string name = "mode_field");
super.new(name);
endfunction : new

endclass : mode_field

class enable_field extends uvm_reg_field;
`uvm_object_utils(enable_field)

function new(string name = "enable_field");
super.new(name);
endfunction : new

endclass : enable_field

class ctrl_reg extends uvm_reg;
`uvm_object_utils(ctrl_reg)

rand mode_field MODE;
rand enable_field ENABLE;

function new(string name = "ctrl_reg");
super.new(name, 32, build_coverage(UVM_NO_COVERAGE));
endfunction : new

virtual function void build();
MODE = mode_field::type_id::create("MODE", null, get_full_name());
MODE.configure(this, 4, 4, "RW", 0, 1, 1, 1, 0);
ENABLE = enable_field::type_id::create("ENABLE", null, get_full_name());
ENABLE.configure(this, 4, 0, "RW", 0, 0, 1, 1, 0);
endfunction : build
endclass : ctrl_reg

======================================== 재료1

class ready_field extends uvm_reg_field;
`uvm_object_utils(ready_field)

function new(string name = "ready_field");
super.new(name);
endfunction : new

endclass : ready_field

class error_cnt_field extends uvm_reg_field;
`uvm_object_utils(error_cnt_field)

function new(string name = "error_cnt_field");
super.new(name);
endfunction : new

endclass : error_cnt_field

class status_reg extends uvm_reg;
`uvm_object_utils(status_reg)

rand ready_field READY;
rand error_cnt_field ERROR_CNT;

function new(string name = "status_reg");
super.new(name, 32, build_coverage(UVM_NO_COVERAGE));
endfunction : new

virtual function void build();
READY = ready_field::type_id::create("READY", null, get_full_name());
ERROR_CNT = error_cnt_field::type_id::create("ERROR_CNT", null, get_full_name());
READY.configure(this, 16, 0, "RO", 0, 0, 1, 1, 0);
ERROR_CNT.configure(this, 16, 16, "RO", 0 , 0, 1, 1, 0);
endfunction : build

endclass : status_reg

======================================== 재료2

class value_field extends uvm_reg_field;
`uvm_object_utils(value_field)

function new(string name = "value_field");
super.new(name);
endfunction : new

endclass : value_field

class reg_type extends uvm_reg;
`uvm_object_utils(reg_type)

rand value_field VALUE;

function new(string name = "VALUE");
super.new(name, 32, build_coverage(UVM_NO_COVERAGE))
endfunction : new

virtual function void build();
VALUE = value_field::type_id::create("VALUE", null, get_full_name());
VALUE.configure(this, 8, 0, "RW", 0, 0, 1, 1, 0);
endfunction : build

endclass : reg_type

class cfg_regfile extends uvm_reg_file;
`uvm_object_utils(cfg_regfile)

rand reg_type MODE, ENABLE, COUNT;

function new(string name = "cfg_regfile");
super.new(name);
endfunction : new

virtual function void build();
uvm_reg_block blk = get_block();
MODE = reg_type::type_id::create($sformatf("%s.MODE", get_name()), 
                                 null, 
                                 blk.get_full_name());
ENABLE = reg_type::type_id::create($sformatf("%s.ENABLE", get_name()), 
                                 null, 
                                 blk.get_full_name());
COUNT = reg_type::type_id::create($sformatf("%s.COUNT", get_name()), 
                                 null, 
                                 blk.get_full_name());
MODE.configure(blk, this, "MODE");
ENABLE.configure(blk, this, "ENABLE");
COUNT.configure(blk, this, "COUNT");
MODE.build();
ENABLE.build();
COUNT.build();
endfunction : build

virtual function void map(uvm_reg_map mp, uvm_reg_data_t offset);
mp.add_reg(MODE, offset + 'h00, "RW");
mp.add_reg(ENABLE, offset + 'h04, "RW");
mp.add_reg(COUNT, offset + 'h08, "RW");
endfunction : map
endclass : cfg_regfile

========================================================== 재료3

class data_mem extends uvm_mem;
`uvm_object_utils(data_mem)

function new(string name = "data_mem");
super.new(name, 64, 4, "RW", build_coverage(UVM_NO_COVERAGE));
endfunction : new

virtual function void sample(uvm_reg_data_t data, uvm_reg_data_t byte_en, bit is_read, uvm_reg_map map);
endfunction : sample

endclass : data_mem

========================================================== 재료4
class data_field extends uvm_reg_field;
`uvm_object_utils(data_field)

function new(string name = "data_field");
super.new(name);
endfunction : new

endclass : data_field

class reg_type extends uvm_reg;
`uvm_object_utils(reg_type)

rand data_field DATA;

function new(string name = "reg_type");
super.new(name, 32, build_coverage(UVM_NO_COVERAGE));
endfunction

virtual function void build();
DATA = data_field::type_id::create("DATA", null, get_full_name());
DATA.configure(this, 32, 0, "RW", 0, 0, 1, 1, 0);
DATA.build();
endfunction : build
endclass : reg_type

class sub_blk extends uvm_reg_block;
`uvm_object_utils(sub_blk)

reg_type R0, R1;

function new(string name = "sub_blk");
super.new(name, build_coverage(UVM_NO_COVERAGE));
endfunction : new

virtual function void build();
R0 = reg_type::type_id::create("R0", null, get_full_name());
R1 = reg_type::type_id::create("R1", null, get_full_name());
R0.configure(this, null, "R0");
R1.configure(this, null, "R1");
R0.build();
R1.build();

default_map = create_map("default_map", 'h200, 4, UVM_LITTLE_ENDIAN);
default_map.add_reg(R0, 'h00, "RW");
default_map.add_reg(R1, 'h04, "RW");
endfunction : build

endclass : sub_blk

====================================================== 재료 5
class top_block extends uvm_reg_block;
`uvm_object_utils(top_block);

rand ctrl_reg CTRL;
rand status_reg STATUS;
rand cfg_regfile CFG;
data_mem DATA_MEM;
rand sub_blk SUB_BLOCK;

function new(string name = "top_block");
super.new(name, build_coverage(UVM_NO_COVERAGE));
endfunction : new

virtual function void build();
default_map = create_map("default_map", 'h1000, 4, UVM_LITTLE_ENDIAN);

CTRL = ctrl_reg::type_id::create("CTRL", null, get_full_name());
CTRL.configure(this, null, "CTRL");
CTRL.build();
default_map.add_reg(CTRL, 'h000, "RW");

STATUS = status_reg::type_id::create("STATUS", null, get_full_name());
STATUS.configure(this, null, "STATUS");
STATUS.build();
default_map.add_reg(STATUS, 'h004, "RO");

CFG = cfg_regfile::type_id::create("CFG", null, get_full_name());
CFG.configure(this, null, "CFG");
CFG.build();
CFG.map(default_map, 'h010);

DATA_MEM = data_mem::type_id::create("DATA_MEM", null, get_full_name());
DATA_MEM.configure(this, "DATA_MEM");
default_map.add_mem(DATA_MEM, 'h020, "RW");

SUB_BLOCK = sub_blk::type_id::create("SUB_BLOCK", null, get_full_name());
SUB_BLOCK.configure(this, "SUB_BLOCK");
SUB_BLOCK.build();
default_map.add_submap(SUB_BLOCK.default_map, 'h200);

lock_model();
endfunction : build

endclass : top_block
