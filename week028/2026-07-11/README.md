193일차 학습한 내용
1. APB Protocol 노트 정리 및 복습
2. APB Protocol 분석 후 SVA 작성

내일 학습할 내용
1. SVA에 대한 파일 구조 만들기 및 작성한 프로퍼티 정리해두기
2. ALU TB에 대해 APB 프로토콜 적용 연습

느낀점
향후 APB 버스 프로토콜을 따르는 주변 장치에 대한 TB를 작성해 SoC 검증에 대한 발판을 마련할 생각임
이전에 만든 ALU TB에서는 콜백을 넣어두긴 했으나 별다른 프로토콜이 없어 별다른 메서드의 동작이 없었음 이를 향후 APB 프로토콜을 적용하고 이에 따라 에러 주입 및 적절한
반응이 나오는지에 대해 테스트해 볼 예정

학습 내용 정리
패리티: UART, 메모리 인터페이스, 버스의 간단한 오류 검출
체크섬: IP, UDP, TCP 헤더
CRC: Ethernet, USB, PCIe, CAN 등 고신뢰 통신
ECC: DDR 메모리, 서버 메모리, 캐시



==================================================================

WRITE_TRANSFER(with wait state or no wait state)

SETUP 페이즈 ~ 액세스 페이즈까지는 PADDR, PWDATA가 유효해야 함
PSEL && !PENABLE |=> PENABLE && $stable(PADDR) && $stable(PWDATA)
후속 조건에 [*1:$] ##n PREADY 사용해보기
대기 상태가 있든 없든 모든 경우 포괄 가능

$rose(PSEL) && !PENABLE

PSEL && !PENABLE |=> PENABLE
PSEL이 들어온 후 다음 사이클에 PENABLE 활성화

(PSEL && PENABLE && PREADY) |=> (!PENABLE)
각각에 대해 apb_back_to_back_transfer
apb_last_transfer

cover로 이 후 idle_phase로 가는거, setup_phase로 가는거 분리해두기

PADDR, PWDATA가 유효한 범위에 속하는지 확인하는 어써션


PENABLE이 assert, PREADY가 비활성화 일 때 유효해야 하는 신호(변하지 않아야 하는)
PADDR, PWRITE, PSELx PENABLE, PWDATA, PSTRB, PPROT, PAUSER, PWUSER

==================================================================

읽기 전송
PSTRB의 모든 비트는 0으로 고정되야 함
PSEL && !PWRITE |-> PSTRB == 'b0

PENABLE이 assert, PREADY가 비활성화 일 때 유효해야 하는 신호(변하지 않아야 하는)
PADDR, PWRITE, PSELx PENABLE,  PPROT, PAUSER

==================================================================

PSLVERR

PSEL && PENABLE && PREADY (APB 전송의 마지막 사이클)동안만 유효한 것으로 고려됨
//PREADY가 활성화될 때 같이 활성화 됨
주변 장치에 PSLVERR가 없다면 LOW로 묶임 
에러를 가진 트랜잭션은 주변 장치를 변경할 수도 안 할 수도 있음
(장치마다 다름) // 구현하고 ERR가 적절히 들어오는지 확인하기

(PSELx && PENABLE && !PREADY) |-> !PSLVERR
!PSELx |-> !PSLVERR
PSLVERR |-> PREADY && PSEL && PENABLE

from AXI to APB : 읽기 에러 - RRESP , 쓰기 에러 - BRESP
from AHB to APB : 읽기/쓰기 에러 - HRESP

==================================================================

PWAKEUP
PCK에 동기화
PSELx && PWAKEUP 이 동시에 HIGH 일 때 PREADY 활성화 전까지 유지 돼야 함
PREADY 활성화 전 PWAKEUP 활성화를 기다리는 것을 허용함
PSELx 최소 1사이클 전 활성화 권장(전, 중, 후 활성화 모두 가능함)
요청자-응답자 클럭은 함께 게이팅할 것이 권장됨. 그렇지 않을 경우 응답자가 셋업 페이즈를 놓칠 수도 있음
글리치 없는 신호를 제공해야 함(레지스터 출력 or 글리치-프리 OR 트리 이용)
=> 비동기 샘플링에 적합하기 위해
PWAKEUP이 존재하나 절대 활성화되지 않을 경우 교착상태에 빠질 수 있음

((PWAKEUP && PSELx) && !$past(PWAKEUP && PSELx)) |-> (PWAKEUP && PSELx) until_with PREADY
// PWAKEUP, PSELx가 모두 HIGH라면 PREADY가 활성화되기까지 두 값은 모두 유지되어야 함
// 추가로 매 클럭킹 이벤트에서 같은 의미를 가진 스레드가 생성되지 않도록 $past문을 추가함

==================================================================

사용자 정의 신호
PRUSER : PSELx가 활성화됐을 때 유효해야 함, SETUP ~ ACCESS 마무리동안 같은 값
PWDATA : PSEL && PWRITE 에서 유효해야 함, SETUP ~ ACCESS 마무리동안 같은 값
PRDATA : PSEL && PENABLE && PREADY && !PWRITE 에서 반드시 유효해야 함
PBUSER : PSEL && PENABLE && PREADY에서 반드시 유효해야 함

데이터 폭은 n바이트로 설정할 것을 권장함

sequence setup_phase;
  PSELx && !PENABLE;
endsequence

sequence access_phase;
  PSELx && PENABLE;
endsequence

sequence wait_state;
  PSELx && PENABLE && !PREADY
endsequence

sequence idle_phase;
  !PSEL && !PENABLE;
endsequence

sequence transfer_done;
  PSELx && PREADY && PENABLE
endsequence
