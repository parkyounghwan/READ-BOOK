---
description: >-
  어떤 언어에 대해서, 해당 언어의 문법에 대한 표현을 정의 하면서 그것을 사용하여 해당 언어로 기술된 문장을 해석하는 해석자를 함께
  정의한다.
---

# Interpreter Pattern \(해석자\)

### \* bytecode 변환

\*Interpreter 자체에 대한 해석, 인터프리터는 바이트 코드를 변환할 때 사용한다.

\*Interpreter 자체에 대한 해석, 인터프리터는 바이트 코드를 변환할 때 사용한다.

### 사전지식

**문맥 자유 문법**

* 특정 언어의 구문을 명확하게 기술하는 표기법.
* 문법과 의미를 서로 분리 시킬 수 있다는 아이디어.
* 프로그래밍 언어의 경우 '문맥 무관 언어' 의 구조 \(기계의 효율적인 번역이 가능하도록 충분히 단순한 구조를 유지시켜 만듦\) 

**BNF 표기법 \(Backus-Naur Form\)**  
프로그래밍 언어의 구문을 서술 할 수 있는 표기법.

> Symbol ::= Expression \(심볼 ::= 표현식\)

좌변을 우변으로써 정의함.



### 참고 링크

* 정보통신기술용어해설 \(BNF 표기법\) [http://www.ktword.co.kr/word/abbr\_view.php?m\_temp1=3019&id=588&nav=2](http://www.ktword.co.kr/word/abbr_view.php?m_temp1=3019&id=588&nav=2)
* 인터프리터 패턴 [https://palpit.tistory.com/199](https://palpit.tistory.com/199)

