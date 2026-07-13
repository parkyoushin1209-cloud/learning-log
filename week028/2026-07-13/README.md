195일차 학습한 내용
1. RAL 학습
사용자 정의 필드

class protected_field_cbs extends uvm_reg_cbs;
  local uvm_reg_field protect_mode;

  function new(string name, uvm_reg_field p_mode);
    super.new(name);
    protect_mode = p_mode;
  endfunction

  virtual task pre_write(uvm_reg_item rw);
    uvm_reg_field field;
    if (protect_mode.get()) begin
      if ($cast(field, rw.element)) begin
        rw.value = field.get(); // 보호 모드 시 이전 값 유지
      end
    end
  endtask
endclass

// 등록 예시
// protected_field_cbs protect_cb = new("protect_cb", protect_mode);
// uvm_callbacks#(my_field_t, uvm_reg_cbs)::add(my_field, protect_cb);


레지스터 타입

class my_reg_type extends uvm_reg;
  `uvm_object_utils(my_reg_type)
  
  rand uvm_reg_field F1;
  rand uvm_reg_field F2[3];

  function new(string name="my_reg_type");
    // 커버리지 모델 생성 결정
    super.new(name, 32, build_coverage(UVM_CVR_REG_BITS + VENDOR_CVR_REG));
  endfunction

  virtual function void build();
    this.F1 = uvm_reg_field::type_id::create("F1");
    this.F1.configure(this, 1, 0, "RW", 0, 0, 1, 1, 0); // 예시 설정
  endfunction

  // RMW(Read-Modify-Write) 구현 예시
  task RMW(output uvm_status_e status, input uvm_reg_data_t data, input uvm_reg_data_t mask);
    uvm_reg_data_t tmp;
    read(status, tmp);
    tmp &= ~mask;
    tmp |= (data & mask);
    write(status, tmp);
  endtask
endclass


레지스터 파일
class my_rf_type extends uvm_reg_file;
  `uvm_object_utils(my_rf_type)
  rand my_reg1_type R1;
  
  virtual function void map(uvm_reg_map imp, uvm_reg_addr_t offset);
    imp.add_reg(this.R1, offset + 'h04); // 오프셋 적용
  endfunction

  virtual function void set_offset(uvm_reg_map imp, uvm_reg_addr_t offset);
    this.R1.set_offset(imp, offset + 'h04); // 오프셋 재귀적 설정
  endfunction
endclass


블록 타입
class my_blk_type extends uvm_reg_block;
  `uvm_object_utils(my_blk_type)
  uvm_reg_map AHB;
  rand my_reg1_type R1;

  virtual function void build();
    this.AHB = create_map("AHB", 'h0, 4, UVM_LITTLE_ENDIAN);
    this.default_map = this.AHB;
    
    this.R1 = my_reg1_type::type_id::create("R1");
    this.R1.configure(this, null);
    this.R1.build();
    this.default_map.add_reg(this.R1, 'h04); // 주소 매핑
  endfunction
endclass


백도어 모니터
class active_monitor extends uvm_reg_backdoor;
  virtual task wait_for_change();
    // DUT 경로 모니터링
    @(root.tb.dut.rf.f1); 
  endtask
endclass

// 블록에서 모니터 연결
// active_monitor_r1 am_r1 = new();
// R1.set_backdoor(am_r1);
// am_r1.start_update_thread(R1);
