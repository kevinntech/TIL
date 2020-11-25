# 자바(Java) 기본 문법 정리
> 아래 내용은 [자바의 정석](https://book.naver.com/bookdb/book_detail.nhn?bid=10191151 "자바의 정석")을 참고 하였습니다.

## 1. 변수(Variable)

#### 1) 변수(Variable)란?

* 변수(Variable)는 하나의 값을 저장할 수 있는 메모리 공간

#### 2) 변수의 선언

* 변수의 선언 방법

  `변수타입 변수이름;`
  
* 변수 선언 예시

    ```java
    int age;      // 정수 타입의 변수 age를 선언
    ```
 
#### 3) 변수에 값 저장하기

* 변수에 값 저장하기

    ```java
    int age;      // 정수 타입의 변수 age를 선언
    age = 25;     // 변수 age에 25를 저장
    ```
  
* 변수에 값 저장하기 (축약형)

    ```java
    int age = 25;
    ```
  
* 변수의 초기화는 변수에 처음으로 값을 저장하는 것을 말한다.

    * 지역변수는 사용하기 전에 반드시 초기화를 해야한다. 그렇지 않으면 컴파일 시 에러가 발생한다.

        ```java
        int x = 0; // 변수 x를 선언 후, 0으로 초기화
        int y = 0;
      
        int x = 0, y = 5; // 위의 두 줄을 한 줄로 표현할 수 있다.
        ```
      
#### 4) 변수의 값 읽어오기

* 변수에 값이 필요한 곳에 변수 이름을 적으면 된다.

    ```java
    int year = 0, age = 14; // int형 변수 year, age를 선언한 다음, 각각 0과 14로 초기화한다.
    year = age + 2000; // 2014
    ```
  
#### 5) 변수의 타입

* 변수의 타입은 저장할 값의 타입에 의해 결정된다.

* 저장할 값의 타입과 일치하는 타입으로 변수를 선언한다.

    ```java
    char ch = '가';    // char는 문자 타입
    double pi = 3.14; // double은 실수 타입
    ```
  
* 변수의 타입은 크게 기본형과 참조형으로 나눌 수 있다.

    * `기본형(Primitive type)`
    
        * 8개의 타입이 있다. (char, byte, short, int, long, float, double, boolean)
    
        * 실제 값을 저장한다.
        
    * `참조형(Reference type)`
    
        * 기본형을 제외한 나머지 타입 (String, System 등)
    
        * 객체의 주소를 저장한다. (4 byte 또는 8 byte)
              
#### 6) 상수와 리터럴

* 변수(Variable) : 하나의 값을 저장할 수 있는 공간

* 상수(constant) : 한 번만 값을 저장 할 수 있는 공간

* 리터럴(literal) : 그 자체로 값을 의미하는 것

    ```java
    final int MAX = 100; // MAX는 상수
    MAX = 200; // 에러 발생
    ```
  
#### 7) 리터럴의 접두사와 접미사

* 접두사

    * 정수형 리터럴 앞에 어떠한 접두사도 없다면 10 진수
    
    * 정수형 리터럴 앞에 0를 붙이면 8진수이다.
    
    * 정수형 리터럴 앞에 0x를 붙이면 16진수이다.
    
    * 정수형 리터럴 앞에 0b를 붙이면 2진수이다. (JDK 1.7 부터 추가됨)
    
        ```java
        int i = 100;
        int oct = 0100;
        int hex = 0x100;
        ```

* 접미사

    * Long 타입의 리터럴에는 접미사 l 또는 L를 사용한다.
 
         ```java
         long l = 10_000_000_000L; // 100억은 int의 최대 값인 20억을 넘기 때문에 접미사 L을 붙이지 않으면 에러가 발생한다.
         long l = 100; // OK
         ```
    
    * float 타입의 리터럴에는 접미사 f 또는 F를 사용한다.
    
    * double 타입의 리터럴에는 접미사 d 또는 D를 사용한다.

         ```java
         float f = 3.14f;
         double d = 3.14d; // d는 생략가능
         ```

* 소수점(.)과 e의 의미
    
    * 소수점(.)의 의미는 다음과 같다.   
        
        * `10.` → `10.0d`
        
        * `.10` → `0.10d`
    
        * `10f` → 10.0f
    
    * e는 10^n를 의미한다.
    
        * `1e3` → `1000.0d`
    
    * 리터럴에 소수점(.) 또는 e, 실수형 접미사(f, d)가 포함된 경우, 실수형 리터럴로 간주한다.

#### 8) 문자와 문자열

 ```java
char ch = 'A';    // char형 변수는 단 하나의 문자밖에 저장 할 수 없다.
char ch = 'AB';   // 둘 이상의 문자를 저장하려고 하면 에러가 발생한다.
String s = "ABC"; // 문자열을 저장하려면 변수의 타입은 String이어야 한다.

char ch = ''; // char형 변수에는 반드시 하나의 문자를 저장해야하므로 에러가 발생한다.
String s = ""; // String 타입의 변수에는 빈 문자열을 저장 할 수 있다.

String s1 = "A" + "B" // String은 덧셈 연산자를 이용해서 문자열을 결합할 수 있다.

// "" + 7는 문자열 "7"이 된다. 즉, 숫자를 문자열로 변경하려면 숫자 + 문자열을 하면된다.
 ```

* 문자열에 어떤 타입을 더하면 그 결과는 문자열이 된다.

#### 9) 두 변수의 값 교환하기

 ```java
int x = 10, y = 20;
int temp;

temp = x;   // x의 값을 temp에 저장
x = y;      // x의 값을 y에 저장
y = temp;   // temp의 값을 y에 저장
 ```

#### 10) 기본형(Primitive type)의 종류와 범위

* 기본형의 종류

    * 논리형 - true와 false 중 하나를 값으로 갖으며, 조건식과 논리적 계산에 사용된다. boolean이 있다.
    
    * 문자형 - 문자를 저장하는데 사용되며, 변수 당 하나의 문자만을 저장 할 수 있다.
    
    * 정수형 - 정수 값을 저장하는데 사용된다. 주로 사용하는 것은 int와 long이며, byte는 이진 데이터를 다루는데 사용되며, short는 c 언어와의 호환을 위해 추가되었다. (잘 쓰이지 않음)

    * 실수형 - 실수 값을 저장하는데 사용된다. float와 double이 있다.

* 기본형의 크기

    * 논리형 - boolean (1byte)

    * 문자형 - char (2byte)

    * 정수형 - byte (1byte) , short (2byte) , int (4byte, 디폴트 타입), long (8byte)
    
    * 실수형 - float (4byte) , double (8byte, 디폴트 타입)
    
* 기본형의 표현범위 (정수형)

    * 1 bit는 이진수 한 자리(0 또는 1)를 의미한다.
    
        * n 비트로 표현 할 수 있는 값의 개수는 2^n개 이다.
        
            * Ex) 8 비트는 256개의 값을 표현 할 수 있다.
 
        * n 비트로 표현 할 수 있는 부호 없는 정수(양수)의 범위는 0 ~ 2<sup>n</sup> - 1 이다.
        
            * 0이 포함되므로 -1을 한다.
            
            * Ex) 8 비트로 표현 할 수 있는 부호 없는 정수의 범위는 1 ~ 256이 아닌 0 ~ 255 이다.
            
        * n 비트로 표현 할 수 있는 부호 있는 정수의 범위는 -2<sup>n-1</sup> ~ 2<sup>n-1</sup> - 1 이다.        

            * Ex) 8 비트로 표현 할 수 있는 부호 있는 정수의 범위는 -128 ~ 127이다.
          
    * 1 byte는 8 bit이다.
    
    * 음수를 표현하기 위해서 첫 번째 비트를 부호 비트(Sign bit)로 사용한다. 
    
    * 부호 비트(Sign bit)의 값이 0이면 양수이고 1이면 음수이다.
         
    * byte로 표현 할 수 있는 값의 범위는 -2<sup>7</sup> ~ 2<sup>7</sup> - 1이다.

    * short로 표현 할 수 있는 값의 범위는 -2<sup>15</sup> ~ 2<sup>15</sup> - 1이다.
    
    * char로 표현 할 수 있는 값의 범위는 0 ~ 2<sup>16</sup> - 1이다.
    
        * char는 문자 코드 값을 저장하기 때문에 음수가 필요 없어서 0 부터 시작한다.
    
    * int로 표현 할 수 있는 값의 범위는 -2<sup>31</sup> ~ 2<sup>31</sup> - 1이다.
    
        * int의 범위는 약 -20억 ~ 20억 이다.
        
    * long으로 표현 할 수 있는 값의 범위는 -2<sup>63</sup> ~ 2<sup>63</sup> - 1이다.
    
        * long의 범위는 약 -800경 ~ 800경 이다.

* 기본형의 표현범위 (실수형)

    * 부동소수점수는 부호(Sign), 지수(Exponent), 가수(Mantissa) 세 부분으로 나눠서 저장한다.
    
    * 실수형은 오차가 발생 할 수 있기 때문에 정밀도가 중요하다.
    
        * float의 정밀도는 7 자리이다. (7 자리까지는 오차 없이 표현 할 수 있다.)
        
            * 정밀도는 오차 없이 표현 할 수 있는 자리 수를 말한다.
            
            * 정밀도를 결정하는 것은 가수의 bit 수이며 float 타입의 가수는 23 bit 이지만 정규화를 통해서 24 bit까지 저장 할 수 있다. 

                * 2<sup>24</sup>는 10<sup>7</sup> 보다는 크고 10<sup>8</sup> 보다는 작다.
                
                * 그래서 float의 정밀도가 7 자리인 것이다.
            
        * double의 정밀도는 15 자리이다.
        
#### 11) 형식화된 출력 - printf()

* 형식화된 출력 - printf()

    * println()의 단점 - 출력형식을 지정 할 수 없다.
    
        * 실수의 자리 수 조절 불가 - (소수점 n자리만 출력 X)
        
        * 10진수로만 출력된다.
        
    * printf()로 출력형식을 지정 할 수 있다.
    
         ```java
        System.out.printf("%.2f", 10.0 / 3);
        System.out.printf("%d", 0x1A);
        System.out.printf("%X", 0x1A);
         ```
      
* printf()의 지시자 (1)

    * 불리언
        * `%b` : 불리언(boolean) 형식으로 출력
    
    * 정수
        * `%d` : 10진(decimal) 정수의 형식으로 출력
        * `%o` : 8진(octal) 정수의 형식으로 출력
        * `%x` , `%X` : 16진(hexa-decimal) 정수의 형식으로 출력
        
    * 실수
        * `%f` : 부동 소수점(floating-point)의 형식으로 출력
        * `%e` , `%E` : 지수(exponent) 표현식의 형식으로 출력
        
    * 문자
        * `%c` : 문자(character)로 출력
        * `%s` : 문자열(string)로 출력
        
    ```java
    // "age:14 year:2017\n"이 화면에 출력된다. \n 또는 %n는 개행문자(줄바꿈)를 의미한다.
    System.out.printf("age:%d year:%d \n", 14, 2017);
    ```
    
* printf()의 지시자 (2)

    * 정수를 10진수, 8진수, 16진수로 출력
    
        ```java
        System.out.printf("%d", 15); // 10진수 15
        System.out.printf("%o", 15); // 10진수 15는 8진수로 17다.
        System.out.printf("%x", 15); // 10진수 15는 16진수로 f다.
        System.out.printf("%s", Integer.toBinaryString(15)); // 2진수 1111로 표현
        ```
      
    * 8진수와 16진수에 접두사 붙이기
    
        * 지시자 앞에 `#`를 붙이면 접두사가 붙어서 출력된다.
    
        ```java
        System.out.printf("%#o", 15); // 017
        System.out.printf("%#x", 15); // 0xf
        System.out.printf("%#X", 15); // 0XF
        ```
      
    * 실수 출력을 위한 지시자 `%f` - 지수 형식(`%e`) , 간략한 형식(`%g`)
    
        ```java
        float f = 123.4567890f;
        /* %f는 소수점 아래 6자리까지 출력된다. 
           정밀도가 7자리 이므로 앞의 7자리까지만 정확하다. 
           %e는 지수형식으로 출력된다. */
        System.out.printf("%f", f); // 123.456787
        System.out.printf("%e", f); // 1.234568e+02

        /* %g는 소수점을 포함해서 7자리까지 간략히 출력한다. 
           지수형태로 표현하는 것이 더 간략하다고 표현되면 지시자 %e와 같이 출력한다. */            
        System.out.printf("%g", 123.456789); // 123.457 
        System.out.printf("%g", 0.00000001); // 1.00000e-8
        ```

* printf()의 지시자 (3)

    * 지시자에 숫자를 붙이면 값이 출력되는 자리 수를 지정 할 수 있다.
    
        ```java
        System.out.printf("[%5d]%n", 10); // 5자리로 출력된다. 빈 자리는 공백으로 채운다.
        System.out.printf("[%-5d]%n", 10); // 왼쪽 정렬을 하려면 지시자에 -를 붙인다.
        System.out.printf("[%05d]%n", 10); // 지시자에 0을 붙이면 빈 자리를 0으로 채운다.
        ```
      
    * 실수형 지시자는 `%전체자리.소수점아래자리f`와 같이 지정 할 수 있다.
    
        ```java
        /* 소수점 아래는 빈 자리를 0으로 채우며 정수 앞 쪽은 공백으로 채운다. */
        System.out.printf("d=%14.10f%n", d); // 전체 14자리 중 소수점 아래 10자리
        ```
      
    * 문자열 지시자는 사용법이 지시자 d와 비슷하다.
    
        ```java
        System.out.printf("[%s]%n", url);
        System.out.printf("[%.8s]%n", url); // 지시자 앞에 .과 숫자를 붙이면 문자열의 일부만 출력 할 수 있다.
        ```
        
#### 12) 정수형의 오버플로우(Overflow)

* `오버플로우(Overflow)`는 표현 가능한 범위를 넘는 것을 말한다.

* `최대값 + 1`는 `최소값`이 되며 `최소값 - 1`는 `최대값`이 된다.

```java
/* byte의 최대 값인 127을 저장
   byte 타입의 값의 범위는 -128 ~ 127 */
byte b = 127;

/* b에 저장된 값을 1 증가한다.
   byte가 저장 할 수 있는 최대 값을 넘었지만 에러가 발생하지 않는다.
   결과는 128이 아닌 -128을 결과로 얻는다. 
   그 이유는 최대 값을 넘어서면 최소 값이 되기 때문이다. */
b = b + 1; // b

byte b2 = 128; // 에러 발생
```

#### 13) 타입 간의 변환 방법

* 문자와 숫자 간의 변환

    * 숫자를 문자로 변경하려면 문자 '0'를 더하면 된다.

    * 문자를 숫자로 변경하려면 문자 '0'를 빼면 된다.

* 문자열과 숫자 간의 변환

    * 숫자를 문자열로 변경하려면 빈 문자열 ""를 더하면 된다.

    * 문자열을 숫자로 변경하려면 `Integer.parseInt()`를 사용하면 된다.

        * 실수 형태의 문자열을 실수로 변경하려면 `Double.parseDouble()`를 사용하면 된다.
        
* 문자열과 문자 간의 변환

    * 문자열을 문자로 변경하려면 `charAt()`를 사용하면 된다.
    
```java
public class TypeTest {
    public static void main(String[] args) {
        String str = "3";

        char result1 = 3 + '0'; // 숫자 3를 문자 3으로 변환
        int result2 = '3' - '0'; // 문자 3를 숫자 3으로 변환
        String result3 = 3 + ""; // 숫자 3를 문자열 3으로 변환
        int result4 = Integer.parseInt(str); // 문자열 3를 숫자 3으로 변환
        char result5 = str.charAt(0); // 문자열 "3"를 문자 '0'으로 변환
    }
}
```

## 2. 연산자(Operator)

#### 1) 연산자 관련 용어

* 연산자는 연산을 수행하는 기호를 말한다. (+, -, *, /)

* 피연산자는 연산자의 연산 수행 대상을 말한다. (변수, 상수, 리터럴 등...)

* 모든 연산자는 연산 결과를 반환한다. (괄호는 연산자가 아니다.)

#### 2) 연산자의 종류

* 단항 연산자 : 연산에 필요한 피연산자의 개수가 1개다.

    * 부호 연산자 : `+` , `-` 
    * 형변환 연산자 : `(type)`
    * 증감 연산자 : `++` , `--` 
    * 비트 전환 연산자 : `~`
    * 논리 부정 연산자 : `!`
    
* 이항 연산자 : 연산에 필요한 피연산자의 개수가 2개다.

    * 산술 연산자
    
        * 사칙 연산자 : `+` , `-` , `*` , `/`
        * 시프트 연산자 : `<<` , `>>`
        * 나머지 연산자 : `%`
        
    * 비교 연산자 : `>`, `<`, `>=`, `<=`, `==`, `!=`
    
    * 논리 연산자
        
        * 논리 연산자 : `&&`, `||`
        * 비트 연산자 : `&`, `|`, `^`
        
* 삼항 연산자 : `? :`
    
* 대입 연산자 : `=`

#### 3) 연산자의 우선순위

* 연산자의 우선순위는 하나의 식에 연산자가 둘 이상이 있을 때, 어떤 연산을 먼저 수행할지를 자동 결정하는 것을 말한다.

    * ① 단항 연산자
    
    * ② 이항 연산자
    
        * 산술 연산자
        
        * 비교 연산자
        
        * 논리 연산자
    
    * ③ 삼항 연산자
    
    * ④ 대입 연산자
    
* 연산자의 우선순위는 상식적으로 생각하자. 우리는 이미 다 알고 있다.

#### 4) 연산자의 결합규칙

* 연산자의 결합규칙은 우선순위가 같은 연산자가 있을 때, 어떤 것을 먼저 수행할 것인가를 말한다.

    * 연산자의 연산 진행 방향은 기본적으로 왼쪽에서 오른쪽이다.
    
    * 단, 단항과 대입 연산자만 오른쪽에서 왼쪽이다.
    
#### 5) 증감 연산자

* 증가 연산자(++) : 피연산자의 값을 1 증가시킨다.

* 감소 연산자(--) : 피연산자의 값을 1 감소시킨다.

    * 전위형 : 값이 참조되기 **전에** 증가시킨다. `j = ++i;`
    
    * 후위형 : 값이 참조된 **후에** 증가시킨다. `j = i++;`
    
#### 6) 부호 연산자

* `-`는 피연산자의 부호를 반대로 변경한다.

* `+`는 아무런 일도 하지 않는다. (실제로는 사용하지 않음)

#### 7) 형변환 연산자

* 형변환은 변수 또는 상수의 타입을 다른 타입으로 변환하는 것을 말한다.

    * `(type) 피연산자`
    
        ```java
        double d = 85.4;
        int score = (int) d; // int로 형변환
        ```

* 형변환 - 예시

    * 표현범위가 좁은 타입에서 넓은 타입으로 형변환하는 경우, 값 손실이 발생하지 않는다.
    
        ```java
        // byte -> int
        byte b = 10;
        int i = b; // 형 변환 생략 가능
        ```

    * 표현범위가 넓은 타입에서 좁은 타입으로 형변환하는 경우, 값 손실이 발생 할 수 있다.
    
        ```java
        // int -> byte
        int i2 = 300;
        byte b2 = (byte) i2; // 형 변환 생략 불가 (명시적으로 형 변환을 해야한다.)
        ```      

    * 타입의 크기가 같더라도 변수의 타입이 다른 경우, 명시적인 형변환을 해야한다.
    
        ```java
        char c1 = 'A';
        short S1 = c1; // 컴파일 불가
        float f1 = 13.3f;
        int i1 = f1; // 컴파일 불가
        ```   

    * 정수와 소수 간의 연산결과는 소수이다.
    
        ```java
        // 정수와 정수 간의 연산결과는 정수이다.
        int x = 10;
        int y = 3;
        System.out.println(x/y); // 결과 : 3
        ```
      
        ```java
        // 정수와 소수 간의 연산는 소수로 자동 형변환된다.
        int x = 10;
        double y = 3;
        System.out.println(x/y); // 결과 : 3.333...5
        ```      
      
  
* 자동 형변환

    * 형변환을 하는 이유는 서로 다른 두 타입을 일치시키기 위해서인데, 형변환을 생략하면 컴파일러가 알아서 자동적으로 형변환을 한다.
    
    * 표현범위가 좁은 타입에서 넓은 타입으로 형변환하는 경우에는 값 손실이 없으므로 두 타입 중에서 표현범위가 더 넓은 쪽으로 형변환된다.

        ```java
        byte b = 10;
        int i = b; // 형변환 생략 가능
        ```
      
    * 자동 형변환 - 예시

        ```java
        byte b = 100; // OK. byte 타입의 범위(-128 ~ 127)의 값을 대입하면
        byte b = (byte) 100; // OK. 컴파일러가 byte 타입으로 자동 형변환하여 대입한다.
      
        int i = 100;
        byte b = i; // 에러. int 타입의 변수를 byte 타입의 변수에 대입하면 에러가 발생한다. (변수이기 때문에 컴파일러가 변수 안에 어떤 값이 들어있는지 알 수 없음)
        byte b = (byte) i; // OK. byte 타입으로 직접 형변환하여 대입하는 것은 가능하다.
        ```
      
#### 8) 사칙 연산자, 산술변환

* 사칙 연산자 (+, -, *, /)

    ```java
    class OperatorEx5{
        public static void main(String args[]) { 
            int a = 10;
            int b = 4;
    
            System.out.printf("%d + %d = %d%n",  a, b, a + b);
            System.out.printf("%d - %d = %d%n",  a, b, a - b);
            System.out.printf("%d * %d = %d%n",  a, b, a * b);
            System.out.printf("%d / %d = %d%n",  a, b, a / b);
            System.out.printf("%d / %f = %f%n",  a, (float)b, a / (float)b);
        }
    }
    ```

* 산술 변환은 연산 전에 피연산자의 타입을 일치시키는 것을 말한다.

    * 두 피연산자의 타입을 같게 일치시킨다. (값 손실을 최소화하기 위해 보다 큰 타입으로 일치시킨다.)
    
        * long + int → long + long → long
        * float + int → float + float → float
        * double + float → double + double → double
    
    * 피연산자의 타입이 int보다 작은 타입이면 int로 변환된다.
    
        * byte + short → int + int → int
        * char + short → int + int → int
        * char - char → int - int → int
            * '2' - '0' → 50 - 48 → 2 (문자를 숫자로 변경하는 경우)
        
#### 9) 반올림, 나머지 연산자

* 반올림

    * `Math.round()` : 실수를 소수점 첫째자리에서 반올림한 정수를 반환한다.
    
        ```java
        long result = Math.round(4.52); // result에 5가 저장된다.
        ```
      
        ```java
        class MathRoundEx{
            public static void main(String args[]) { 
                // pi의 값을 소수점 넷째 자리인 5에서 반올림하려고 함
                double pi = 3.141592;
                double shortPi = Math.round(pi * 1000) / 1000.0;
                // double shortPi = (double) Math.round(pi * 1000) / 1000;
                System.out.println(shortPi); // 결과 : 3.142
      
                // 3.141을 얻고 싶다면?
                System.out.println((int)(pi*1000) / 1000.0); // 결과 : 3.142
            }
        }
        ```

* 나머지 연산자(`%`)
    
    * 왼쪽의 피연산자를 오른쪽 피연산자로 나누고 남은 나머지를 반환한다.

    * 나누는 피연산자는 0이 아닌 정수만 허용하며 부호는 무시된다.

        ```java
        class OperatorEx19 {
            public static void main(String args[]) { 
                int x = 10;
                int y = 8;
        
                System.out.printf("%d을 %d로 나누면, %n", x, y); 
                System.out.printf("몫은 %d이고, 나머지는 %d입니다.%n", x / y, x % y); 
            }
        }
        ```

#### 10) 비교 연산자, 문자열의 비교

* 비교 연산자 (`>`, `<`, `>=`, `<=`, `==`, `!=`)

    * 비교 연산자는 두 피연산자를 비교해서 true(참) 또는 false(거짓)을 반환한다.
    
* 비교 연산자의 종류

    * `>` : 좌변 값이 **크면** true 아니면 false

    * `<` : 좌변 값이 **작으면** true 아니면 false
    
    * `>=` : 좌변 값이 **크거나 같으면** true 아니면 false
    
    * `<=` : 좌변 값이 **작거나 같으면** true 아니면 false
                
    * `==` : 두 값이 **같으면** true 아니면 false

    * `!=` : 두 값이 **다르면** true 아니면 false
    
* 문자열의 비교

    * 문자열을 비교할 때는 `==` 대신에 `equals()`를 사용해야 한다.
    
    * 문자열의 내용이 같은지 비교하기 위해서는 `equals()`를 사용해야 한다.
    
        ```java
        String str1 = new String("abc");
        String str2 = new String("abc");
        
        System.out.println(str1 == str2); // false
        System.out.println(str1.equals(str2)); // true
        ```
  
#### 11) 논리 연산자, 논리 부정 연산자

* 논리 연산자 (`&&`, `||`)

    * 논리 연산자는 두 조건식을 연결할 때 사용한다.
    
        * `||` : 피연산자 중 어느 한 쪽이 true이면 true를 결과로 얻는다. 
    
        * `&&` : 피연산자 양쪽 모두 true이어야 true를 결과로 얻는다.

    * 논리 연산자 - 예시
    
        * `x > 10 && x < 20`는 x는 10 보다 크고, 20보다 작다.
    
        * `i%2==0 || i%3==0`는 i는 2의 배수 또는 3의 배수이다.
        
        * `(i%2==0 || i%3==0) && i%6!=0`는 i는 2의 배수 또는 3의 배수지만 6의 배수는 아니다.

        * `'0' <= ch && ch <= '9'`는 문자 ch는 숫자('0'~'9')이다.

        * `('a' <= ch && ch <= 'z') || ('A' <= ch && ch <= 'Z')`는 문자 ch는 대문자 또는 소문자이다.
                
* 논리 부정 연산자 (`!`)

    * 논리 부정 연산자는 피연산자가 true이면 false를, false이면 true를 결과로 반환한다.
    
        ```java
        boolean b = true;
        System.out.println(!b);
        ```   
    
#### 12) 조건 연산자, 대입 연산자

* 조건 연산자 (`? :`)

    * 조건 연산자는 조건식의 평가결과에 따라 다른 결과를 반환한다. `조건식? 식1 : 식2`
    
    * 조건식의 평가결과가 true이면 식1이, false이면 식2가 연산결과가 된다.
    
        ```java
        result = (x > y) ? x : y; // 괄호 생략 가능
        ```
      
* 대입 연산자 (`=`)

    * 대입 연산자는 오른쪽 피연산자를 왼쪽 피연산자에 저장 후 저장된 값을 반환한다.

        ```java
        System.out.println(x = 3); // 변수 x에 3이 저장되고
        → System.out.println(3); // 연산결과인 3이 출력된다.
        ```

    * 복합 대입 연산자 : 대입 연산자는 다른 연산자와 결합하여 사용 될 수 있다.
    
        * `+=` , `-=` , `/=` , `%=` ...
        
        * `i *= 10 + j;`는 `i = i * (10 + j);`와 같다.
        
## 3. 조건문과 반복문

#### 1) if문

* 조건식이 참(true)일 때, 괄호{} 안의 문장들을 수행한다.

* 거짓이면 if문 다음 문장으로 넘어간다.

    ```java
    if(조건식){
      // 조건식이 참(true)일 때, 수행될 문장들을 적는다.
    } 
    ```
  
    ```java
    if(score > 60){
      System.out.println("합격입니다.");
    } 
    ```
  
#### 2) if-else문

* 조건식이 참(true)일 때와 거짓(false)일 때로 나눠서 처리한다.

* 조건식의 결과에 따라 두 개의 블럭 중 어느 한 블럭{}의 내용이 수행되고 전체 if문을 벗어나게 된다.

    ```java
    if(조건식){
      // 조건식이 참(true)일 때, 수행될 문장들을 적는다.
    }else{
      // 조건식이 거짓(false)일 때, 수행될 문장들을 적는다.
    }
    ```
  
    ```java
    if(input == 0){
      System.out.println("0 입니다.");
    }else{
      System.out.println("0이 아닙니다."); 
    }
    ```
  
#### 3) if-else if문

* 처리해야 할 경우의 수가 셋 이상인 경우에 사용한다.

    ```java
    if(조건식1){
      // 조건식1의 연산결과가 참일 때 수행될 문장들을 적는다.
    }else if(조건식2){
      // 조건식2의 연산결과가 참일 때 수행될 문장들을 적는다.
    }else if(조건식3){
      // 조건식3의 연산결과가 참일 때 수행될 문장들을 적는다.
    }else{
      // 위의 어느 조건식도 만족하지 않을 때 수행될 문장들을 적는다.
    }
    ```

    * 첫 번째 조건식부터 순서대로 평가해서 결과가 참인 조건식을 만나면, 해당 블럭{}만 수행하고 if-else if문 전체를 벗어난다.
    
    * 만일 결과가 참인 조건식이 하나도 없으면, 마지막에 있는 else 블럭의 문장들이 수행된다. 그리고 else 블럭은 생략이 가능하다.
    
    ```java
    class If_Else_If_Ex {
    	public static void main(String[] args) { 
    		int score  = 0;   // 점수를 저장하기 위한 변수
    		char grade =' ';  // 학점을 저장하기 위한 변수. 공백으로 초기화한다.
    
    		System.out.print("점수를 입력하세요.>");
    		Scanner scanner = new Scanner(System.in);
    		score = scanner.nextInt(); // 화면을 통해 입력받은 숫자를 score에 저장
    
    		if (score >= 90) {         // score가 90점 보다 같거나 크면 A학점
    			 grade = 'A';             
    		} else if (score >=80) {   // score가 80점 보다 같거나 크면 B학점 
    			 grade = 'B'; 
    		} else if (score >=70) {   // score가 70점 보다 같거나 크면 C학점 
    			 grade = 'C'; 
    		} else {                   // 나머지는 D학점
    			 grade = 'D'; 
    		}
    		System.out.println("당신의 학점은 "+ grade +"입니다.");
    	}
    }
    ```

#### 4) 중첩 if문

* 중첩 if문은 if 문 블럭 내에 또 다른 if문을 포함시키는 것을 말한다.
    
    ```java
    if(조건식1){
      // 조건식1의 연산결과가 true일 때 수행될 문장들을 적는다.
      if(조건식2){
        // 조건식1의 연산결과가 true일 때 수행될 문장들을 적는다.
      }else{
        // 조건식1이 true이고, 조건식2가 false일 때 수행될 문장들을 적는다.
      }
    }else{
      // 조건식1의 연산결과가 false일 때 수행될 문장들을 적는다.
    }
    ```

    ```java
    if (score >= 90) {           // score가 90점 보다 같거나 크면 A학점(grade)
        grade = 'A';
        if (score >= 98) {        // 90점 이상 중에서도 98점 이상은 A+
            opt = '+';	
        } else if (score < 94) {  // 90점 이상 94점 미만은 A-
            opt = '-';
        }
    } else if (score >= 80){     // score가 80점 보다 같거나 크면 B학점(grade)
        grade = 'B';
        if (score >= 88) {
            opt = '+';
        } else if (score < 84)	{
            opt = '-';
        }
    } else {                     // 나머지는 C학점(grade)
        grade = 'C';
    }
    ```

#### 5) switch 문

* switch 문은 처리해야 하는 경우의 수가 많을 때, 유용한 조건문이다.

    ```java
    switch(조건식){
          case 값1 :
              // 조건식의 결과가 값1과 같을 경우, 수행될 문장들
              break; // switch문을 벗어난다.
          case 값2 :
              // 조건식의 결과가 값2와 같을 경우, 수행될 문장들
              break;
          //...
          default :
              // 조건식의 결과와 일치하는 case문이 없을 때 수행될 문장들
    }   
    ```
  
    * switch 문 동작 순서
    
        * ① 조건식을 계산한다.
        * ② 조건식의 결과와 일치하는 case문으로 이동한다.
        * ③ 이후의 문장들을 수행한다.
        * ④ break문이나 switch문의 끝을 만나면 switch문 전체를 빠져나간다.

* switch 문의 제약조건

    * switch 문의 조건식 결과는 정수 또는 문자열(JDK 1.7 부터)이어야 한다.
    
    * case문의 값은 정수 상수(문자 포함), 문자열만 가능하며, 중복되지 않아야 한다.
    
        ```java
        int num, result;
        final int ONE = 1;
        
        switch(result){
              case '1' :      // OK. 문자 리터럴 (정수 49와 동일)
              case ONE:       // OK. 정수 상수
              case "YES":     // OK. 문자열 리터럴 (JDK 1.7부터 허용)
              case num:       // 에러. 변수는 불가
              case 1.0:       // 에러. 실수도 불가
                  ...
        }   
        ```

```java
class FlowEx6 {
	public static void main(String[] args) { 
		System.out.print("현재 월을 입력하세요.>");

		Scanner scanner = new Scanner(System.in);
		int month = scanner.nextInt();  // 화면을 통해 입력받은 숫자를 month에 저장

		switch(month) {
			case 3: 
			case 4: 
			case 5:
				System.out.println("현재의 계절은 봄입니다.");
				break;
			case 6: case 7: case 8:
				System.out.println("현재의 계절은 여름입니다.");
				break;
			case 9: case 10: case 11:
				System.out.println("현재의 계절은 가을입니다.");
				break;
			default:
				System.out.println("현재의 계절은 겨울입니다.");
		}
	}
} 
```

#### 6) for문

* for 문은 반복 횟수를 알고 있을 때, 사용하기 적합하다.

    ```java
    for(초기화; 조건식; 증감식){
      // 수행될 문장
    }
    ```
  
    * 제일 먼저 ①초기화가 수행되고, 그 이후부터는 조건식이 참인 동안 ②조건식 → ③수행될 문장 → ④증감식의 순서로 계속 반복된다.

    * 그러다가 조건식이 거짓이 되면, for문 전체를 빠져나가게 된다.
    
    ```java
    for (int i = 1; i <= 3; i++) { // 괄호{} 안의 문장을 3번 반복한다.
        System.out.println("Hello");
    }
    ```
  
* for문 관련 용어

    * 초기화 : 반복문에 사용될 변수를 초기화하는 부분이며 처음에 한번만 수행된다.
    * 조건식 : 조건식의 값이 참(true)이면 반복을 계속하고, 거짓(false)이면 반복을 중단하고 for문을 벗어난다.
    * 증감식 : 반복문을 제어하는 변수의 값을 증가 또는 감소시키는 식이다.
    
* 무한 반복문

    * 조건식이 생략되면 참(true)으로 간주되어 무한 반복문이 된다.
    
    * 대신 블럭{} 안에 if문을 넣어서 특정 조건을 만족하면 for문을 빠져 나오게 해야한다.
    
        ```java
        for (;;) {
            ...
        }
        ```
      
#### 7) 중첩 for문

* 중첩 for문은 for문 안에 또 다른 for문을 포함시키는 것을 말한다.

    ```java
    for(int i=2; i<=9; i++){ // 구구단
        for(int j=1; j<=9; j++){
            System.out.println(i + "*" + j + "=" + (i * j));
        }
    }
    ```

#### 8) 향상된 for문

* JDK 1.5 부터 배열과 컬렉션에 저장된 요소에 접근할 때 기존보다 편리한 방법으로 처리 할 수 있도록 for문의 새로운 문법이 추가되었다.

    ```java
    for(타입 변수명 : 배열 또는 컬렉션){
      // 반복할 문장
    }
    ```

    * 배열 또는 컬렉션에 저장된 값이 매 반복마다 하나씩 순서대로 읽혀서 변수에 저장된다.
    
    * 그리고 반복문의 괄호{}내에서는 이 변수를 사용해서 코드를 작성한다. 
  
#### 9) while문 , do-while문

* while문은 조건식이 참(true)인 동안, 블럭{} 내의 문장을 반복한다. (반복 횟수를 모를 때 사용한다.)

    ```java
    while(조건식){
      // 조건식의 연산결과가 참(true)인 동안, 반복될 문장들을 적는다.
    }
    ```
  
    * while문 동작 순서
        * ① 조건식이 참(true)이면 블럭{}안으로 들어가고, 거짓(false)이면 while문을 벗어난다.
        * ② 블럭{}의 문장을 수행하고 다시 조건식으로 돌아간다.

    ```java
    int i=1; // 초기화
  
    while(i<=10){ // 조건식
      System.out.println(i);
      i++; // 증감식
    }
    ```
        
* while문의 무한 반복문

    ```java
    while(true){
      // ...
    }
    ```
  
* do-while문은 블럭{}을 먼저 수행한 후에 조건식을 평가한다.

* 그리고 do-while문은 최소한 한번은 수행될 것을 보장한다.        

    ```java
    class FlowEx28 {
    	public static void main(String[] args) { 
    		int input  = 0, answer = 0;
    
    		answer = (int)(Math.random() * 100) + 1; // 1~100 사이의 임의의 수를 저장
    		Scanner scanner = new Scanner(System.in);
    
    		do {
    			System.out.print("1과 100사이의 정수를 입력하세요.>");
    			input = scanner.nextInt();
    
    			if(input > answer) {
    				System.out.println("더 작은 수로 다시 시도해보세요.");	
    			} else if(input < answer) {
    				System.out.println("더 큰 수로 다시 시도해보세요.");			
    			}
    		} while(input!=answer);
    
    		System.out.println("정답입니다.");
    	}
    }
    ```
  
#### 10) break문 , continue문

* break문은 자신이 포함된 하나의 반복문을 벗어난다.

    ```java
    class FlowEx30 {
    	public static void main(String[] args) { 
    		int sum = 0;
    		int i   = 0;
    
    		while(true) {
    			if(sum > 100)
    				break;
    			// break문이 수행되면 아래 문장은 실행되지 않고 while문을 완전히 벗어난다.
    			++i;
    			sum += i;
    		} // end of while
    
    		System.out.println("i=" + i);
    		System.out.println("sum=" + sum);
    	}   
    }
    ```
  
* continue문은 자신이 포함된 반복문의 끝으로 이동한다. (다음 반복으로 넘어간다.)

    ```java
    class FlowEx31 {
    	public static void main(String[] args) {
    		for(int i=0;i <= 10;i++) {
    			if (i%3==0)
    				continue; // 조건식이 참이 되어 continue문이 수행되면 블럭의 끝으로 이동한다.
                              // break문과 달리 반복문을 벗어나지 않는다.
    			System.out.println(i);
    		}
    	}
    }
    ```

#### 11) 임의의 정수 만들기

* `Math.random()`는 0.0과 1.0 사이의 범위에 속하는 하나의 double 값을 반환한다.

* 0.0은 범위에 포함되고 1.0은 범위에 포함되지 않는다.

* 임의의 정수 만들기 - 예시 (1) : 1과 3 사이의 정수를 구하기

    * ① 각 변에 3을 곱한다.
    
        ```
        0.0 * 3 <= Math.random() * 3 < 1.0 * 3
        0.0 <= Math.random() * 3 < 3.0
        ```
      
    * ② 각 변을 int형으로 변환한다.

        ```
        (int) 0.0 <= (int) (Math.random() * 3) < (int) 3.0
        0 <= (int) (Math.random() * 3) < 3
        ```
          
    * ③ 각 변에 1을 더한다.

        ```
        0 + 1 <= (int) (Math.random() * 3) + 1 < 3 + 1
        1 <= (int) (Math.random() * 3) + 1 < 4
        ```
      
    * 정답 : `(int) (Math.random() * 3) + 1`

* 임의의 정수 만들기 - 예시 (2)

    ```java
    class MathRandomEx {
        public static void main(String args[]) {
            int num = 0;
    
            // 괄호{} 안의 내용을 5번 반복한다.
            for (int i = 1; i <= 5; i++) {
                num = (int) (Math.random() * 6) + 1; // 1~6까지의 임의의 정수 구하기
                System.out.println(num);
            }
        }
    }
    ```


        





    

      


      
  
    
   

    

            


    







    




    






        

        


    
    
    
    
    
    

    

    

    











  

  

  

  

  

    





     