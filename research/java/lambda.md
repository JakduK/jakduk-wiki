<!-- TITLE: Lambda -->
<!-- SUBTITLE: -->

# Lambda에서 local변수의 값을 변경 할 수 없는 이유
function interface 인스턴스가 생성될때 lambda내부(body)에서 참조하는 바깥 local변수들은 메소드 호출할때처럼,
primitive변수는 값복사, 레퍼런스변수는 레퍼런스가 복사된다. 즉, labmda내부에서 바깥 local변수의 값을 바꿔봤자 의미없다.
눈으로 보기에 local변수를 직접 접근하는 것처럼 보여 오해가 생긴다.

# Lambda에서 local변수의 값을 변경 할 수 없게 한 이유
* http://www.lambdafaq.org/what-are-the-reasons-for-the-restriction-to-effective-immutability/
정리는 나중에