139일차 학습한 내용
1. UVM 팩토리 사용 이유 및 장점, 일부 메서드 확인

UVM 팩토리 이용 시 정확한 객체 지정을 런 타임으로 미룰 수 있음
new를 통해 생성한다면 부모클래스의 생성자 메서드 오버라이딩이 어려울 수 있으나 팩토리는 더 많은 유연성을 제공할 것임

inst_name = class_name::type_id::create(name, parent) 형식으로 작성 가능
또한 팩토리 오버라이딩도 아래의 문법으로 가능함 
factory.set_type/inst_override_by_type((inst일 때만)inst_name,이미 존재하는 것(바꿀 거), 새로 바꿀 타입)
