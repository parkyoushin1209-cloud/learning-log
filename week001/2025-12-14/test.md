#define _CRT_SECURE_NO_WARNINGS
#include <stdio.h>
#include <stdlib.h>

void print_binary(int x) {
	for (int i = sizeof(x) * 8; i >= 0; i--) { //sizeof(x) int자료형의 바이트 크기 반환 바이트 = 8비트
		printf("%d", (x >> i) & 1); // i번째 비트 확인
	}
	printf("\n");
}// 파이썬의 경우 bin(x)로 출력

int main() {
	int n = 5;
	printf("%d\n", (n >> 2) & 1); //n의 2번째 비트 확인  & = and 연산 둘 다 1이면 1-> (n 두번째 비트가 1이면 1 0이면 0으로  **비트 추
