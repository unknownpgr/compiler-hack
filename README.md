# Compiler for new language
if나 for, while같은 제어 구문까지 override 가능한 언어를 만들고 싶은데.
복잡하게 생각하지 말고, 다음과 같은 형식을 생각하자.

```
control(p1)
{
    p2
}
```

이때 고민은, p1을 어떻게 처리해야 아름답게 처리되냐는 것이다. p1에는 분명히 코드 블럭이 들어온다. 문제를 환원하면, 코드 블럭들을 어떻게 파라매터로 받을 수 있는가다.
- 그냥 세미콜론으로 구분할까?
    - 만약 여러 줄의 코드 블럭을 받고 싶다면?
- 코드 블럭을 나타내는 기호 {}를 정의하면 어떨까?
    - 멋진 생각이긴 한데, 그러면 스코프가 분리된다.
- 바로 위의 방법을 그대로 차용하되, 코드 블럭을 제어구문 내에서 사용하면 스코프를 공유하도록 하면 어떨까?
    - 좋은 방법인 것 같다.

#### 2020 / 04 / 26
- 말도 안 되는 문법의 언어를 만듬.
- 일단 자바스크립트처럼 사용 가능

#### 2020 / 04 / 27
- 다음과 같은 정말 추상적이기 그지없는 코드 구조를 만들어보자.
    - 코드의 실행 단위는 코드블럭이다.
    - 코드블럭은 하나의 코드라인이거나, 코드라인들의 묶음이다.
    - 코드블럭은 반환값을 가질 수 있다.
    - 코드블럭은 기본적으로 실행되지 않는다.

#### 2020 / 04 / 28
- 어제 대충 문법을 완성하였다. 물론 if문도 없는 단순한 언어이기는 하다.
- 그렇다면 이제 진짜 프로그래밍 언어를 작성해보자.
- 프로그래밍 언어라면, 아래와 같은 속성을 가져야 한다.
    1. 변수를 선언할 수 있다.
    2. 변수를 참조할 수 있다.
    3. 변수에 값을 할당할 수 있다.
    4. 변수의 값을 검사하여 특정 위치로 점프할 수 있다.(0이면/아니면 점프)
- 지금 문제되는 것은 과연 프로그래밍 언어에서 '예기치 못한'변수가 생길 수 있냐는 점이다. 예를 들어 if문 안에 변수를 만든다면?
    - 이게 문제가 되는 이유는 변수의 스코프와, 변수 공간 할당 때문이다.
    - 만약 모든 변수가 <strong>컴파일 타임에 예상가능하다면</strong>, 우리는 변수 공간을 고민할 필요도 없고, 스코프도 고민할 필요가 없다.
    - 그러나 c언어에서 malloc이 보여주듯, 그렇지 않음이 자명하다.
    - 그런데 고민되는 점은, 만약 이럴 경우 scope는 어떻게 되냐는 점이다.
    - 결론 : 눈에 보이는 변수의 스코프만 고려하자.

#### 2020 / 04 / 29
- AST만드는 법을 공부해야겠다.

#### 2020 / 05 / 01
- AST만드는 법을 알아냈다. Antlr4에서 nonterminal에 label붙이는 것을 지원한다. 그러므로 쉽게 AST를 분석할 수 있다.
- 추가적으로 visitor pattern과 listener pattern에 대해 알게 되었다.
    - visitor pattern이 더욱 flexible하다고 볼 수 있지만, listener는 explicit stack을 사용하는 반면, visitor는 call stack을 사용한다. visitor는 그러므로 recursion이 너무 깊어질 경우 터질 위험이 있다.
    - 근데, call stack이 아무리 깊어 봐야 100~1000단계를 넘을 수 있을까? 어지간한 상용 언어는 아무리 못해도 64단계는 버틴다.

#### 2020 / 05 / 02
- Antlr4에서 Visitor pattern을 제대로 사용해본다!

#### 2020 / 05 / 04
- 인터넷에 Python 예제가 너무 없어서, Java 예제를 사용하기로 한다.
- 그리고 python으로 하니까 IDE가 마음에 안 든다. 이렇게 딱딱한 작업은 Java가 좋다.

#### 2020 / 05 / 05
- 오늘은 어린이날이니까, 아무것도 안 하고 놀 거다. 물론 어린이가 아니기는 하다.

#### 2020 / 05 / 06
- 이틀 정도 고민해봤는데 아무리 생각해도 어떻게 함수를 객체로 만드는지 모르겠다. 그나마 내가 생각할 수 있는 솔루션은 모든 변수를 heap에다 저장하는 방법이다. 그것 말고는 도저히 답이 보이지 않는다.
- 그렇다면 모든 변수를 heap에 저장하는 것을 전제로 하고 한 번 생각해보자.
    - 그렇게 하는 이상 GC는 필수적이다. 그렇다면, GC를 할 변수/타이밍을 컴파일 타임에 결정할 수 있는가?
    - 그렇게 할 수 없다. 왜냐하면 레퍼런스 개수 자체가 동적으로 변할 수 있기 때문이다. 예컨대
        ```
        object a;
        for(int i=0;i<10;i++){
            a = a.ref;
        }
        ```
        위와 같은 구문이 있다고 하면, for문의 개수에 따라 레퍼런스가 달라질 것이다.
- 그렇다면 컴파일 타임에 레퍼런스 변화를 완벽히 추정할 수 있는가?
    - '레퍼런스'라는 것을 어떻게 나타낼 수 있는지부터 논의해보자.
      
        - 레퍼런스라는 것은 필연적으로 단방향이다. 그러므로 방향 그래프와 같은 것으로 나타낼 수 있다.
    - 이때 헷갈리는 개념 중 하나는 바인딩이다. 위 코드에서 볼 수 있듯, 같은 변수 이름이 같은 객체를 나타내는 것은 아니다. 예를 들어 int a = 1;이라고 하면, a라는 변수와 1이라는 값을 바인딩한 것이다. 이후 다시 a = 2;라고 하면, 이것은 a라는 이름에 바인드 된 객체를 변화시키는 게 아니라, a에 새로운 객체'2'를 다시 바인딩하는 것이다.
    이때 '1'에 대한 참조는 사라진다.
    - 그렇다면 과연 객체에 새로운 변수를 바인딩하는 게 아니라, 객체의 값을 수정하는 경우는 있는가? 애초에 자바마냥 primitive type과 reference type을 칼같이 구분하는 것이 자연스러운 코딩 방법인가?
    - heap에 할당된 변수를 수정하면 어떤 일이 벌어지는가?
    - heap에 할당된 변수는 애초에 direct한 방법으로는 수정할 수가 없고, 오직 pointer를 통해서만 수정이 가능하다.
    - 실제로 C언어에서 어떻게 처리하는지 궁금해졌다. 내가 알기로, C언어에서는 pointer를 사용하지 않는 한 레퍼런스 참조란 없다. 음... 만약 아래와 같이 하면 어떤 일이 벌어지는가?
        ```
    #include <stdio.h>
    
        typedef struct {
            int value;
    }Type;
    
        void main() {
            Type a;
            a.value = 1;
        Type* aPtr = &a;
    
            Type b;
        b.value = 2;
    
            a = b;
        printf("%d, %d\n", a.value, aPtr->value);
    
            b.value = 3;
            printf("%d, %d", a.value, aPtr->value);
        }
        ```
    - 흥미롭게도, 2,2 / 2,2 가 출력되었다. 즉, C언어에서는 포인터를 사용하지 않는 한, 레퍼런스만 바뀌는 일은 없다. 그러므로 C언어에서는 포인터를 사용하지 않는 한, number type들 간의 변환 등의 특별한 경우를 제외하고는, 동적 타입 변환이란 있을 수가 없다.
    - 다시 원래 논제로 돌아와서, 이로부터 컴파일 타임에 레퍼런스를 완벽히 추적할 수 있는지가 궁금해졌다.
    - 애초에 C언어처럼 레퍼런스는 레퍼런스라고 명시하는 것이 적절한가, 아니면 Java마냥 전부 레퍼런스인 게 적절한가? 알 수 없다.

#### 2020 / 05 / 07
- 만약 Java처럼 싹다 레퍼런스로 한다면 어떤 문제가 생기는가?
    - primitive type과 reference type의 참조가 일관적이지가 않다.
    - 객체 복사가 어렵다.
    - 변수 이름과 실제 들어있는 값이 일관적이지 않을 수도 있다.
- 만약 C처럼 싹다 primitive type이면 어떤 문제가 생기는가?
    - 메모리 낭비가 심하다.
    - 일일이 reference를 명시해줘야만 한다.
    - Pointer의 pointer의 pointer같은 거지같은 구조가 만들어질 수 있다.
- 만약 primitive type까지 전부 reference type으로 한다면 어떤 문제가 생기는가?
  
    - primitive type은 '속성'을 가지지 않는다. 그러므로 reference type으로 하는 의미가 없다.
- 아래와 같은 실험을 해 봤다.
    ```
        private static void objectRef(Object o) {
            o = new Object();
            System.out.println(o);
        }

        public static void main(String[] args){
            Object o = new Object();
            System.out.println(o);
            objectRef(o);
            System.out.println(o);
        }
    ```
    이로부터 자바에서 =연산은, 객체를 수정하는 것이 아니라, binding만을 바꾼다는 것을 알게 됐다. 애초에 그럴 수밖에 없다.
- 나는 Java / C# / Javascript처럼 유연하면서도 C처럼 '확실한' 언어를 만들고자 한다. C언어의 단점은 무엇인가? 왜 우리는 C언어로 고급 프로그래밍을 하지 않는가?
    - 고급의 프로그래밍을 하려면 객체지향이 지원돼야 하는데, C언어에서는 객체지향을 지원하지 않는다.
    - 무엇보다, 간단한 문자열 처리부터 시작해서 손댈 게 너무 많다.
- 그렇다면 객체지향과 Lambda를 '쉽게' 지원하는 C언어같은 걸 만들어보자. 물론 이미 C++이라는 좋은 언어가 있지만, 뭐, 내가 상업용으로 쓰일 언어가 필요한 건 아니잖아?
    - 먼저 GC가 매우 귀찮은 부분임을 고려하여, GC를 생각하지 않기로 한다.
    - 그리고 모든 변수는 힙에 저장하기로 한다. 왜냐하면 컴파일 타임에 힙에 저장할 변수와 스택에 저장할 변수를 구분하는 것은 매우 귀찮은 작업이기 때문이다. 지금은 최적화같은 것은 고려하지 않고, 일단 동작하는 것에 중점을 두기로 한다.
    - 나는 지금까지 런타임에 스코프를 고려할 필요가 있다고 착각하고 있었다. 그런데 생각해보니, 스코프는 컴파일 타임에만 고려하면 되는 것 같다.
- 프로그래밍 언어를 만들기 위해서 몇 가지 고려해야 할 점이 있음을 깨달았다.
  
    1. 변수 선언은

#### 2020 / 06 / 17

 일단 기말고사 과제를 위하여 프로젝트를 Hack asm을 생성하도록 바꿨다. 역시 가장 문제되는 부분은 function이다. 만약 scope가 전역이라면, variable definition 시에 그냥 변수 i,j,k 이렇게 주면 된다. scope가 전역이 아니라도 그게 허용되는가? 

- 안 된다. 왜냐하면 함수를 재귀적으로 호출할 경우, 문제가 생기기 때문이다. 따라서 stack을 만든다. 애초에 main 함수가 따로 있는 까닭이 있었어. stack과 heap을 분리해야하기 때문이었다.
- 일단 context 는 function과 global 두 개만 있으면 될 것 같다. 그 외에 뭐, 필요할 만한 게 있을까.
- 문제는 recursive expression을 어떻게 해결하냐는 것이다.
- 예를 들어 a+((b+(c+d))+e).
  - 이 경우, 접근은 a부터 하게 된다. 위에서 밑으로 들어가니까.

다음 경우를 한 번 풀어보자.
```
    (a+b)+((c+d)+(e+f))
```
이는 단순히 변수 두 개만으로는 해결할 수 없는 문제다.
이 문제를 해결하면 아래와 같이 풀 수 있다.
```
    x1 = a+b
    x2 = c+d
    x3 = e+f
    x4 = x2+x3
    x5 = x1+x4
```
그러면 문제는 이렇게 된다.
- 만약 연산을 발견하면, 연산자를 stack에 넣는다.
- 만약 피연산자를 즉시 계산할 수 있다면, 피연산자 두 개를 스택에 넣는다.
- 그렇지 않은 경우, 연산에서 빠져나올 때 stack에서 두 개를 pop 하여 연산한 후 스택에 넣는다.

#### 2020 / 06 / 18

두 메모리의 값을 더할 때에는 다음과 같이 한다. 이는 더하는 것 뿐만이 아니라, 모든 이항 연산을 하는 것에 적용된다.

```
// c = a + b;
@a
@b
@c

@a
D=M
@b
D=D+M
@c
M=D
```



변수 선언은 어떻게 이뤄지는가?

```
/*
스택의 주소를 BASE라고 하고, 스택 포인터가 저장된 위치를 STP라 하자.
그러므로 초기에는 STP의 값이 BASE이며, 실제 값이 저장되는 위치는 STP-1번지다.

예를 들어, var a라 하자. 그렇다면 스택의 어떤 지점에 a가 저장돼있을 것이다.
그런데 이런 식으로 참조를 하려면, 한 서브 루틴 내에서 상대 참조가 가능해야만 한다.
그러므로 스택 전체의 base만 있어서는 안 되고, 현재 서브루틴의 base도 저장이 돼 있어야 한다. 현재 서브루틴의 base를 저장하는 위치를 CBASE라 하자.

그러므로 함수를 call할 때 다음과 같은 과정으로 한다고 하자.
1. stack에 CBASE값 push
2. CBASE에 STP값 push
3. stack에 현재 PC값 push
4. stack에 파라매터들 push

그렇다면, 프로그램에서 어떤 서브루틴이 호출된 직후, stack의 모양은 다음과 같다.

BASE // main 함수의 이전 base. 무의미함.
NULL // main 함수의 return value, 무의미함.
PARAM // main 함수의 param. hack에서는 없으니 무의미함.
VAR
...
VAR
CBASE	<< 이것은 main 함수의 CBASE값임.
RETURN 	<< CBASE는 여기를 가리킴
PARAM
...
PARAM
		<< STP는 여기를 가리킴
        
그러므로 변수를 새로 생성할 때에는
1. 컴파일러의 해쉬맵에 이름 / related index를 저장하고
2. STP++하는 코드를 추가.
이때 related index란, 1+paramLen+index.
CBASE기준으로 0이면 RETURN이므로 여기는 못 씀.
paramLen길이만큼은 또 파라매터이므로 못 씀. 그 다음부터 쓸 수 있음.

변수에 값을 할당할 때에는
1. 컴파일러의 해쉬맵에서 related index를 가져오고
2. CBASE에 더해서 그 주소를 얻은 후
3. 그 주소에 값을 쓴다.

함수를 종료할 때에는
1. @ret 	// 반환값의 주소
2. D=M		// D에 복사
3. @CBASE	// CBASE의 주소
4. A=M		// CBASE의 값 = RETURN
5. 0;JMP	// 원래 주소로 돌아옴. D에는 여전히 반환값 있음.
돌아온 후에는
6. @CBASE, A=M, M=D // D값 return 위치에 백업
7. @CBASE, D=M, @STP, M=D // STP값 한칸 앞으로 복구
8. @CBASE, A=M-1, D=M, @CBASE, M=D // CBASE값 복구
9. @STP, D=M, @STP, A=A-1, M=D // 리턴값 복구
10. @STP, M=M-1


마지막으로 함수를 종료하고 나서는 
1. @STP		// STP의 주소
2. A=M		// STP의 값
3. M=D		// 함수를 호출한 함수의 스택에 값을 쓰고
4. @STP		// STP의 주소
5. M=M+1	// STP 1 증가

이렇게 하면 함수를 수행하고, 그 값이 스택의 맨 마지막 위치에 써진다.
*/
```



함수 선언은 다음과 같이 한다.

```
/*
함수가 호출될 때, Stack 에는 return address, parameters 순서로 들어 있다.
*/
~~~
@SKIP
0;JMP
(FUNC)
do some task
(SKIP)
```

#### 2020 / 06 / 25

- 변수 이름은 컴파일 타임에만 존재한다.

- 실제 변수를 참조할 때 사용하는 값은, stack pointer로부터의 상대주소다.

- 변수를 생성하는 것은 오직 컴파일 타임에 이뤄지는 것이다. 그러므로 한 함수의 필요한 스택 사이즈는 컴파일 타임에 완전히 결정된다고도 볼 수 있다. 단, 함수 호출로 인하여 전체적인 코드가 변할 수 있을 뿐. 그러므로 `var a;`와 같은 코드는 **<u>실제 기계어 상으로는 아무 것도 하지 않는다</u>**. 다만 컴파일 타임에 해시맵에 내용을 등록할 뿐. 해시맵에는 변수의 이름과 변수의 상대 주소가 같이 등록돼있다.

- 값을 참조하는 부분이 계속 헷갈린다.

  - 값 참조의 가장 기본은, **<u>A레지스터에 변수의 주소를 올리는 것이다.</u>**
  - 이걸로 고민 끝. 좀 시시할만큼 고민이 빨리 끝나버렸다.

- 함수는 어떻게 관리하면 좋을까?

  - 일단 생각을 잘 해야 하는데, 내가 지금 구현하고 있는 언어에서는 함수 안에 함수가 들어올 일이 없다. 그러므로 부모 함수를 가리키는 포인터 등도 전혀 필요하지 않다. 따라서 함수는

    1. 전역 context
    2. 현재 작업중인 context

    이 두 개면 족하다.

  - 전역 context는 그냥 함수로 구현하는 것이 맞다고 본다.

  - 따라서 변수 선언은 반드시 현재 context에 하도록 하고, 만약 현재 context에서 변수 선언을 발견할 수 없으면 전역 context를 탐색하도록 한다.

- 마지막으로, 지역에서 전역 변수를 참조할 때 어떻게 참조하면 좋은가?

  - 변수 참조는 다음과 같은 과정으로 이루어진다.
    1. 컴파일 타임에 결정되는 변수의 상대주소를 D에 저장.
    2. CBASE값을 D에 더함.
    3. 그러면 변수의 실제 주소값이 얻어짐. 이를 다시 A에 옮김.
  - 따라서 전역 변수의 스코프는 확실히 특수하게 만들어질 필요가 있다.

- 이제 완벽하게 함수 호출에 대해 이해한 것 같다.

  - 자, 궁금한 것은 이거다. 과연 D 레지스터를 안 쓰고 접근이 가능한가?

  - 불가능하다. 그렇다면, 이건 되나?

    1. D레지스터의 값을 어딘가 잠시 백업
    2. A레지스터에 값 저장
    3. D레지스터의 값 복구

    이건 자명히 불가능하다. 그럼 이건 되나?

    1. D레지스터의 값을 잠시 백업(상수 위치 A)
    2. A레지스터의 값을 계산
    3. A레지스터의 값을 잠시 백업(상수 위치 B)
    4. D레지스터의 값을 복구
    5. A레지스터의 값을 복구

    ```
    // Backup D register
    @CA
    M=D
    
    // Do some calculations here
    A=D
    
    // Backup A register
    @CB
    M=D
    
    // Restore D register
    @CA
    D=M
    
    // Restore A register
    @CB
    A=M

    오 이건 된다.
    ```
    
    #### 2020 / 05 / 26
    
    이게 된다면, 이것과 변수 참조를 합칠 수는 없나?
    
    ```
    @CA
    M=D
    
    @STP
    D=M
    @value_offset_from_compiler
    A=A+D
    ```
    
    딱히 합친다고 해서 효율적으로 되지는 않는 듯하다.
    
    만약 call stack과 변수 공간을 완전히 분리하면 어떤 일이 벌어지나?
    
    - 별로 달라질 것 없고, 그냥 불편하기만 할 것 같다. 힙을 쓴다면 또 모를까.
    
  - 드디어 제대로 된 컴파일러를 하나 구현했다!
  
  - 함수 호출, 분기가 된다.
  
  - 즉, 제대로 튜링 완전한 언어다.
  
  - 다만 문제가 하나 있는데, 뭔가 while이 끼이면 제대로 안 돌아간다는 점이다.
  
  - 이걸 어떻게 수정한다지......
  
  - 어디 스택을 저장해놔야 하나...

#### 2020 / 06 / 28

- 왜 제대로 잘 안 돌아가는지 대충 알았다. While 내에서 계산을 할 때, 변수를 계속 생성해서 이런 일이 발생하는 거였다. 변수를 만들고 이름 할당은 컴파일 타임에 하는데, 스택은 런타임에 증가시키니 이런 일이 발생한다. 이걸 어떻게 고치면 좋지?
- 필요한 모든 변수를 코드의 앞 부분에 선언해두면 어떤 문제가 발생하나?
  - 스택 포인터가 꼬인다. 함수를 호출할 때, BASE, STACK TOP을 저장해두는 부분이 있다. 이게 꼬여버린다.
- 다시 보니 함수가 문제인 것 같다. 왜이러지?
  - 실수로 Stack-top-pointer 등을 제대로 고치지 않은 것 같다.