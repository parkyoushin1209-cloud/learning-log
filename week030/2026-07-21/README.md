202일차 학습한 내용
1. APB, VPI 예시 코드 분석 및 TB 구조 재정립
2. spec 분석을 통한 제약 사항 재확인 및 base 시퀀스의 제약 변경

스코어보드에 다음과 같은 코드를 두고 관찰 횟수 와 임계점을 비교해 main_phase() 에서 마무리하는 방법도 있음을 알 수 있었다.
 task reset_phase(uvm_phase phase);
      m_n_obs = 0; // 관찰한 횟 수
      m_sb.delete(); // 큐 비우기
   endtask
            
   task main_phase(uvm_phase phase);
      phase.raise_objection(this, "Have not checked enough data");
      wait (m_n_obs > n_obs_thresh);
      phase.drop_objection(this, "Enough data has been observed");
   endtask
endclass

또한 Primecell GPIO spec 분석 관정에서 nGPAFEN은 소프트웨어 제어 모드일 경우 HIGH로 묶여야 한다는 점을 발견하고 이를 해결하기 위해
pre_body에서 GPIOAFSEL 정보를 읽어와 변수에 저장해 둔 뒤 이 변수를 이용해 !cur_gpioafsel -> gpio_tr.nGPAFEEN == 1 이라는 제약을 두었다
또한 레지스터 모델을 매 시퀀스마다 불러와야 하는 번거로움을 없애기 위해 `uvm_declare_p_sequencer 매크로와 uvm_config_db를 사용해 base 시퀀스에서 레지스터 모델을 미리 설정해 두었다
마지막으로 외부 신호의 경우 인터페이스 포트에서 output으로 두고 tb_top에 wire를 둠으로써 다른 하드웨어와의 연결도 고려할 수 있다는 점도 알게 되었다.
마지막으로 tx_monitor, rx_monitor를 항상 별개로 만들어 active_monitor, passive_monitor에 두어야 한다는 오개념을 잡고 지금까지 작성했던 코드들의 일부를 수정하는
과정을 진행했다.
