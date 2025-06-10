# 함수형 프로그래밍
- 외부 상태에 독립적으로 수행되는 함수들의 조합을 코드로 작성하는 방식의 프로그래밍 방식
- 순수 함수
    - 각 함수는 외부 상태를 알 필요도, 알 수도 없어야 한다.
    - 외부 상태를 모르기 때문에 항상 같은 입력에 대해 동일한 출력을 반환한다.
    - 외부 상태를 변경할 수 없기 때문에 의도하지 않은 부수효과가 발생하지 않는다.
- 불변성
    - 한 번 생성된 데이터는 변경되지 않고 데이터 변경이 필요할 시 변경된 값이 적용된 새로운 데이터를 생성한다.
- 일급 객체
    - 함수는 함수 자체로 일급 객체로 취급된다.
    - 함수를 변수에 할당하거나, 다른 함수의 인자로 전달, 함수에서 함수를 반환할 수 있다.

## 순수 함수
- 외부 상태 : 함수의 행동에 영향을 미칠 수 있는, 함수 밖에서 변경될 수 있는 변수나 상태
    - 전역 변수
    - 클래스의 인스턴스 변수
    - 외부 시스템 상태
- 실제 애플리케이션을 만들 때 외부 시스템(파일 시스템, DB 등)을 참조하지 않는 경우는 매우 드물기 때문에 순수 함수로만 구성된 애플리케이션을 작성하는 것은 사실상 불가능하다.


~~~java
public class FuncExampleCode {
    public static void main(String[] args) {
        String pattern = "pattern";

        List<String> lines = readLines();
        List<String> matchedLines = findPattern(lines, pattern);
        String summary = summarizeResults(matchedLines);

        System.out.println(summary);
    }

    // 외부 파일을 읽는 대신 내부에서 정의된 텍스트 데이터를 반환하는 순수 함수
    private static List<String> readLines() {
        return new ArrayList<>(Arrays.asList(
                "first",
                "second",
                "third",
                "fourth pattern"
        ));
    }

    // 주어진 패턴과 일치하는 줄을 찾는 순수 함수
    private static List<String> findPattern(List<String> lines, String pattern) {
        List<String> matchedLines = new ArrayList<>();
        for (String line : lines) {
            if (line.contains(pattern)) {
                matchedLines.add(line);
            }
        }
        return matchedLines;
    }

    // 결과를 요약하는 순수 함수
    private static String summarizeResults(List<String> matchedLines) {
        return "패턴이 발견된 줄의 수: " + matchedLines.size();
    }
}
~~~

### forEach
- Java8 부터 나온 함수형 프로그래밍을 위한 라이브러리 stream의 최종 연산
- stream이 나온 배경 -> 함수형 프로그래밍을 위함
- forEach는 가장 덜 stream다운 연산
- stream의 최종 연산은 결과를 도출하는 작업을 수행
- forEach는 각 요소를 반복하며 동작을 수행하고, 반환 값은 존재하지 않음
    - 또, forEach는 최종 연산에서 __상태를 변경할 수 있음__
- 반환 값도 없을 뿐더러, 상태를 변경할 수 있기 때문에 가장 덜 stream다운 연산이다.
- 함수형으로 작성할 때 forEach를 사용하는 경우 출력(System.out.println())과 같이 상태를 변경시키지 않는 연산만 수행해야 순수 함수 유지 가능

## 불변성
- 한 번 생성되면 상태가 변경되지 않는 것이 불변성이고 함수형 프로그래밍의 특징이다.

~~~java
public final class Person {
    private final String name;
    private final int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public Person updateAge(int newAge) {
        return new Person(this.name, newAge);
    }

    public Person updateName(String newName) {
        return new Person(newName, this.age);
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }
}
~~~

~~~java
public class ImmutableExample {
    public static void main(String[] args) {
        Person messi = new Person("Messi", 35);
        Person updateMessi = messi.updateAge(36);

        System.out.println("messi.age " + messi.getAge());
        System.out.println("updateMessi.age " + updateMessi.getAge());

        System.out.println("messi.address" + messi);
        System.out.println("updateMessi.address" + updateMessi);
    }
}
// ======================== 출력 ========================
messi.age 35
updateMessi.age 36

messi.addressfuntional.Person@6d06d69c
updateMessi.addressfuntional.Person@7852e922
~~~

- `updateAge()`에서 새로운 객체를 생성하기 때문에 messi 객체와 updateMessi 객체의 주소가 다르다.
    - age 상태를 변경하기 위해 새로운 Person 객체를 생성
    - `updateAge()`는 messi(Person)의 age값을 모르고 있으면서 상태를 변경

## 일급 객체
~~~java
public class FirstClassFunctionExampleSecond {
    public static void main(String[] args) {
        List<Person> persons = Arrays.asList(
                new Person("son", 29),
                new Person("messi", 30),
                new Person("ronaldo", 31),
                new Person("kane", 33),
                new Person("byeong", 32)
        );

        Predicate<Person> lengthPredicate = person -> person.getName().length() >= 5;
        Function<Person, String> nameUpperCaseFunction = person -> person.getName().toUpperCase();
        Consumer<String> printConsumer = System.out::println;

        StringBuilder names = new StringBuilder();
        Consumer<Person> concatName = person -> names.append(person.getName());
        persons.forEach(concatName);
        System.out.println(names);

        Comparator<Person> byAge = Comparator.comparing(person -> person.getAge());
        Collections.sort(persons, byAge);
        persons.forEach(person -> System.out.println(person.getName() + " " + person.getAge()));

        Comparator<Person> byAgeThenName = Comparator.comparing(Person::getAge).thenComparing(Person::getName);
        Collections.sort(persons, byAgeThenName);
        persons.forEach(person -> System.out.println(person.getName() + " " + person.getAge()));


        List<String> nameConverter = persons.stream()
                .filter(lengthPredicate)
                .map(nameUpperCaseFunction)
                .collect(Collectors.toList());

        nameConverter.stream()
                .forEach(printConsumer);

        Supplier<Double> averageAgeSupplier = () -> persons.stream()
                .mapToInt(person -> person.getAge())
                .average()
                .orElse(0.0);

        System.out.println("Average Persons Age = " + averageAgeSupplier.get());

        IntBinaryOperator maxOperator = Math::max;

        int maxAge = persons.stream().mapToInt(Person::getAge)
                .reduce((a, b) -> maxOperator.applyAsInt(a, b))
                .orElseThrow(() -> new IllegalArgumentException("Person is Empty"));

        IntBinaryOperator multiplyOperator = (a, b) -> a * b;

        int multiplyAge = persons.stream().mapToInt(Person::getAge)
                .reduce((a, b) -> multiplyOperator.applyAsInt(a, b))
                .orElseThrow(() -> new IllegalArgumentException("age is not integer"));

        System.out.println("maxAge = " + maxAge);
        System.out.println("multiplyAge = " + multiplyAge);

        int targetComparison = 29 * 30 * 31 * 32 * 33;

        BiPredicate<Integer, Integer> isSame = (a, b) -> a.equals(b);
        boolean ageIsSame = isSame.test(multiplyAge, targetComparison);

        System.out.println("Age Is Same ? " + ageIsSame);

        BiFunction<Person, Person, Integer> twoPersonAddAge = (p1, p2) -> p1.getAge() + p2.getAge();
        Integer totalAge = twoPersonAddAge.apply(persons.get(0), persons.get(1));
        System.out.println("totalAge = " + totalAge);

        BiConsumer<Person, Person> printPersonDetails = (p1, p2) -> {
            System.out.println(p1.getName() + " is " + p1.getAge() + " years old.");
            System.out.println(p2.getName() + " is " + p2.getAge() + " years old.");
        };

        printPersonDetails.accept(persons.get(0), persons.get(0));
    }
}
~~~

### Predicate
~~~java
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
}
~~~

- 인자 T를 받아서 boolean을 반환
- 람다식으로는 T -> boolean
    - person -> person.getName().length() >= 5;
        - person은 T
        - person.getName().length() >= 5;은 boolean

### Consumer
~~~java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
}
~~~

- 인자 T를 받고 아무것도 반환하지 않음
- 람다식으로는 T -> void
- 이름과 같이 소비만 하고 끝남
- Consumer<Person> concatName = person -> names.append(person.getName());
    - person을 받아 names라는 StringBuilder 객체에 person.getName()을 append

### Supplier
~~~java
@FunctionalInterface
public interface Supplier<T> {
    T get();
}

Supplier<Double> averageAgeSupplier = () -> persons.stream()
        .mapToInt(person -> person.getAge())
        .average()
        .orElse(0.0);
~~~

- 아무 인자도 받지 않고 T를 반환
- 람다식으로는 () -> T
- 이름처럼 아무것도 받지 않고 특정 객체를 반환

### Function
~~~java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}
~~~

- 인자 T를 받아 R을 반환
- 람다식으로 T -> R
- T와 R은 같은 타입 사용 가능
- Function<Person, String> nameUpperCaseFunction = person -> person.getName().toUpperCase();
    - 인자 T는 Person
    - 인자 R은 String
    - person을 받아서 person의 name에 toUpperCase() 적용
- `persons.stream().filter(lengthPredicate).map(nameUpperCaseFunction).collect(Collectors.toList());`

### Comparator
~~~java
@FunctionalInterface
public interface Comparator<T> {
    int compare(T o1, T o2);
}
~~~

- 정렬이 필요할 때 정렬의 기준을 정할 수 있는 인터페이스
- compareTo() 메서드를 오버라이딩 후 커스텀으로 정렬하고 싶은 기준을 구현
    - Collections.sort(Object, Comparator)와 같이 사용
        - Collections.sort(persons, byAge);

### BiXXX
- 두 개의 인자를 받는 함수를 나타낸다.
- 두 개의 입력에 대한 연산을 수행할 때 사용

~~~java
BiPredicate<Integer, Integer> isSame = (a, b) -> a.equals(b);
BiFunction<Person, Person, Integer> twoPersonAddAge = (p1, p2) -> p1.getAge() + p2.getAge();
BiConsumer<Person, Person> printPersonDetails = (p1, p2) -> {
            System.out.println(p1.getName() + " is " + p1.getAge() + " years old.");
            System.out.println(p2.getName() + " is " + p2.getAge() + " years old.");
        };
~~~

## Lambda식
- 함수를 하나의 식으로 표현
- 함수 이름이 없기 때문에 익명 함수의 한 종류
- Lambda식으로 선언된 함수는 일급 객체이기 때문에 Stream의 매개변수로 전달 가능
- Lambda식 내에서 사용되는 지역 변수는 final이 붙지 않아도 상수로 간주
    - 함수형 프로그래밍의 불변성에 일맥상통
- Lambda식을 사용하면서 만든 익명 함수는 재사용이 불가능하다.
- 디버깅이 어렵다.

~~~java
int add(int x, int y) {
    return x + y;
}
// 위 함수를 Lambda식으로 변경

(int x, int y) -> {return x + y;} // 기본
(x, y) -> {return x + y;} // 자료형 생략 가능
(x, y) -> x + y; // 실행문이 한 문장의 반환문이면 중괄호와 return 생략 가능

str -> {return str.length();} // 매개변수가 하나일 때 소괄호 생략 가능

x, y -> {System.out.println(x + y);} // 에러 (매개변수 2개면 소괄호 생략 불가능)
str -> return str.length(); // 에러 (중괄호와 return이 동시에 생략되어야 함)
~~~

~~~java
(p1, p2) -> {
            System.out.println(p1.getName() + " is " + p1.getAge() + " years old.");
            System.out.println(p2.getName() + " is " + p2.getAge() + " years old.");
        }

// 위 Lambda식을 일반 메서드로 변경
public void printNameAndAge(Person p1, Person p2) {
    System.out.println(p1.getName() + " is " + p1.getAge() + " years old.");
    System.out.println(p2.getName() + " is " + p2.getAge() + " years old.");
}
~~~

