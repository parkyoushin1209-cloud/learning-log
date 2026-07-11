192일차 학습한 내용
1. SVA 가이드라인 학습

<참고할 서적>
Formal verification: An Essential Toolkit for Modern VLSI Design
Erik Seligman

 SymbiYosys (SBY) & Yosys 형식검증
EBMC(형식 검증 툴 for 하드웨어)

<가이드라인>
체커 바인드 시 체커 정의가 먼저 컴파일 돼야 함

output port
always_comb + =
always_ff + <=
clocking_block
assign

inout port 또는 internal wire
assign
clocking_block

task, always 내부에서
넷에 할당 불가(=, <= , assign)

시퀀스 아이템 프로퍼티 
Control : 전송할 타입, 사이즈 관련
Payload : 전송할 주요 데이터 내용
configuration : 새로운 동작 모드, 에러 동작 등
Analysis : 분석을 돕는 편리한 필드(time stamp, rolling checksum)

