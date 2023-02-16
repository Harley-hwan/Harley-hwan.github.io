#include <iostream>
#include <cstdlib>
#include <ctime>
#include <vector>
#include <algorithm>

int main() {
    // 현재 시간을 시드로 사용하여 난수 생성기 초기화
    std::srand(static_cast<unsigned int>(std::time(nullptr)));

    // 1부터 45까지의 숫자를 저장하는 벡터 생성
    std::vector<int> numbers(45);
    for (int i = 0; i < 45; i++) {
        numbers[i] = i + 1;
    }

    // 벡터를 무작위로 섞음
    std::random_shuffle(numbers.begin(), numbers.end());

    // 벡터에서 첫 6개의 요소를 선택하여 로또 번호로 출력
    std::cout << "로또 추천 번호: ";
    for (int i = 0; i < 6; i++) {
        std::cout << numbers[i] << " ";
    }
    std::cout << std::endl;

    return 0;
}
