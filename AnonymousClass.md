# 익명 클래스
- 익명 클래스는 말 그대로 이름이 없는 클래스
- __익명의 함의는 기억되지 않아도 된다는 것__
    - 다시 불러질 이유가 없기 때문에 일시적으로 한번만 사용되고 버려지는 객체
    - 일시적으로 한번만 사용되고 버려지기 때문에 부수효과를 지양하는 `함수형` 프로그래밍에 적합

## 일반 클래스 사용 예시
~~~java
class Car {
    public String engineStart() {
        return "부릉";
    }
}

class KiaCar extends Car {
    @Override
    public String engineStart() {
        return "딩디디딩딩~ 부릉";
    }
}

public class Main {
    public static void main(String[] args) {
        Car car = new KiaCar();
        car.engineStart();
    }
}
~~~

- 위와 같이 `KiaCar`를 사용하기 위해서는 `Car`클래스를 또 생성해서 오버라이딩 후 사용
- 클래스를 하나 더 생성해야 하고, 생성에 따라 관리가 필요하기 때문에 비용이 발생
- 또, 한번만 사용되는데 클래스로 생성해서 빼야하기에는 비효율적

## 익명 클래스 사용 예시
~~~java
class Car {
    public String engineStart() {
        return "부릉";
    }
}

public class Main {
    public static void main(String[] args) {
        // 익명 클래스 정의
        Car car = new Car() {
            @Override
            public String engineStart() {
                return "딩디디딩딩~ 부릉";
            }
        }; // 익명 클래스 끝에 세미콜론 필요
        // 익명 클래스 사용
        car.engineStart();
    }
}
~~~
- 재사용할 필요 없는 일회성 클래스를 정의하고 생성하지 않아도 됨
    - 일종의 코드를 줄이는 기법

## 유의점
- 클래스를 생성하여 상속 받으면 오버라이딩도 가능하고, 새로운 메서드도 생성 가능
- 익명 클래스는 오버라이딩한 메서드만 사용할 수 있다.
    - 즉, 새로 정의한 메서드는 __외부__ 에서 사용 불가능
    - 단, 정의한 익명 클래스 __내부__ 에서는 사용 가능

~~~java
class Car {
    public String engineStart() {
        return "부릉";
    }
}

public class Main {
    public static void main(String[] args) {
        Car car = new Car() {
            @Override
            public String engineStart() {
                return "딩디디딩딩~ 부릉";
            }

            public String engineStop() {
                return "딩딩디디딩-";
            }
        };
        car.engineStart();
        car.engineStop(); // !!! 사용 불가능 !!!
    }
}
~~~

# 익명 인터페이스
- 익명 클래스는 일회성 & 오버라이딩 용으로 사용
- 추상화 구조인 인터페이스를 일회용으로 구현하여 사용할 때 더 효과적으로 사용할 수 있다.
- 인터페이스를 인스턴스화 하여 사용하는 것처럼 사용 가능
    - 실제 구현은 자식 클래스를 생성해서 `implements` 하고 클래스를 초기화 한 것과 차이 없음
    - 예시
        - Runnable 인터페이스를 `implements`하지 않고 메서드 내부에서 구현할 때와 동일
- 단 인터페이스의 큰 본질은 다중 상속을 통해 구현할 수 있는 것인데, `둘 이상의 인터페이스` 혹은 `상속과 구현을 동시`에 할 때 익명 클래스를 사용할 수 없다.
    - class A implements B, C > 불가
    - class A extend B implements C > 불가
    

~~~java
Interface Car {
    String engineStart();
    String engineStop();
}

public class Main {
    public static void main(String[] args) {
        Car car = new Car() {
            @Override
            public String engineStart() {
                return "딩디디딩딩~ 부릉";
            }
            @Override
            public String engineStop() {
                return "딩딩디디딩-";
            }
        };

        // 인터페이스를 구현한 객체이기 때문에 사용 가능
        car.engineStart();
        car.engineStop();
    }
}
~~~

## 활용 예시
- 간단하게 문법적인 동작은 아래 코드를 통해 참고
~~~java
interface Operate {
    int operate(int a, int b);
}
~~~

~~~java
class Calculator {
    private final int a;
    private final int b;
	
    public Calculator(int a, int b) {
        this.a = a;
        this.b = b;
    }

    // 인터페이스 타입을 매개변수로 받는 메서드
    public int caculate(Operate op) {
        return op.operate(this.a, this.b); // 매개변수 객체의 메서드 실행, 리턴
    }
}
~~~

~~~java
public class Main {
    public static void main(String[] args) {
        Calculator calculator = new Calculator(20, 10);

        // operate() 추상 메소드를 익명 구현 객체로 넘김
        // operate() 메서드 실행, a + b가 리턴
        int result = calculator.caculate(new Operate() {
            public int operate(int a, int b) {
                return a + b;
            }
        });

        System.out.println(result); // 30


        // operate() 추상 메소드를 익명 구현 객체로 넘김
        // operate() 메서드 실행, a - b가 리턴
        int result2 = calculator.caculate(new Operate() {
            public int operate(int a, int b) {
                return a - b;
            }
        });

        System.out.println(result2); // 10
    }
}
~~~


- 아래 코드는 실제 어떤 구현을 할지 간단한 예시

~~~java
import java.util.Arrays;
import java.util.Comparator;
 
public class Main {
    public static void main(String[] args) {
 
        class User {
            String name;
            int age;
 
            User(String name, int age) {
                this.name = name;
                this.age = age;
            }
        }
 
        User[] users = {
            new User("홍길동", 32),
            new User("김춘추", 64),
            new User("임꺽정", 48),
            new User("박혁거세", 14),
        };
 
        // 1번 방법
        Arrays.sort(users, new Comparator<User>() {
            @Override
            public int compare(User u1, User u2) {
                return Integer.compare(u1.age, u2.age);
            }
        });

        // 혹은 2번 방법
        Comparator<User> byAge = (u1, u2) -> Integer.compare(u1.age, u2.age);
        Arrays.sort(users, byAge);

        // 혹은 3번 방법
        Arrays.sort(users, (u1, u2) -> Integer.compare(u1.age, u2.age));
    }
}
~~~

# 익명 객체(클래스)와 람다
- 익명 클래스의 목적은 간결하게 코드를 작성하고 관리하기 쉽게 하는 것
- 람다식 문법과 잘 어울림
- 단, 람다식 익명 클래스(객체)는 아래 제약이 존재
    - 인터페이스로만 구현 가능
        - 때문에 함수형 문법을 지원하는 자바 FunctionalInterface는 사용 가능
        - 하나의 추상 메서드만 선언되어 있는 인터페이스만 가능
            - default 메서드는 제외

~~~java
public interface Operate {
    // 추상 메서드 하나
    int operate(int a, int b);

    // default 메서드는 추상 메서드에 포함 X
    default void print() {
        System.out.println("출력");
    }
}
~~~

- 람다식 예제 코드
~~~java
// 일반 코드
new Comparator<User>() {
            @Override
            public int compare(User u1, User u2) {
                return Integer.compare(u1.age, u2.age);
            }
        }

// 람다식
(u1, u2) -> Integer.compare(u1.age, u2.age);
~~~