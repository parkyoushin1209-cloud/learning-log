200일차 학습한 내용
1. TLM 포트 연결 및 apb환경, 에이전트, 모니터 구현 및 연결
2. GPIODATA 처리 로직 구상

PL061 GPIO의 GPIODATA는 접근 주소에 따라 write/read mask가 결정되는 alias register 구조를 가지며 
UVM RAL 기반 alias 모델링을 검토한 결과, 다수의 alias register 생성 및 callback 기반 동기화가 필요하여 유지보수 측면의 한계가 있음을 확인하였다.

이에 따라 RAL은 일반 레지스터 모델링에 사용하고, GPIODATA alias 동작은 APB transaction 기반으로 검증하도록 분리했다. 
APB sequence에서 alias 주소를 생성하고 Reference Model에서 주소 기반 mask 연산을 수행하여 예상 결과를 생성한 뒤, 
Monitor에서 수집한 DUT 결과와 비교하는 구조를 구현하였다.
