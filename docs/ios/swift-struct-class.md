# Class 와 Struct

모든 언어의 class 와 struct 정의는 거이 비슷하다. swift 내에서의 class, struct 의 특성 및 사용가이드를 정의해보고자 한다.

## Class 특성

* Reference Type
* ARC로 메모리 관리
* 상속 가능
* 타입 캐스팅을 통해 런타임에서 클래스 인스턴스의 타입을 확인할 수 있다.
* 인스턴스가 소멸될때 deinit 메서드가 호출된다. (참조타입이니까)
* 참조 비교 연산자 사용가능( `===`, `!==` )

## Struct 특성

* Value Type
* 상속이 가능하지 않다
* 여러 인스턴스를 만들고 값을 변경해도 , 각 인스턴스의 값은 다르다. (값 타입이니까;)

## 차이점

* 구조체는 상속할 수 없다.
* 타입캐스팅은 클래스의 인스턴스에만 허용된다.
* deinit은 클래스의 인스턴스에서만 호출된다.
* 참조 횟수 계산은 클래스의 인스턴스에만 적용된다.
* 구조체는 생성자를 구현하지 않아도 default initalizer 를 사용할 수 있다.
* 구조체 내부에 구조체, 클래스 내부에 클래스 등 중첩타입의 정의 및 선언이 가능하다.
* 반면에 구조체는 init은 사용 가능하다 deinit은 사용 불가능하다.
* Swift 표준 라이브러리의 기본 타입은 모두 구조체이다.(String, Bool, Int, Array, Dictionary, Set)


## 초기화 예제

!!! Class
    `init` `convenience init` `required init`
    === "convenience"

        ```swift
        class Person {
            var name: String
            var age: Int
            
            init(name: name, age: age){
                self.name = name
                self.age = age
            }
            
            convenience init(name: name){
                self.init(name: name, age: 27) // 지정 이니셜라이저 호출
            }
        }

        let aiden = Person(name: "Aiden")// Person(name: "Aiden", age: 27)
        //==========================================================================
        // convenience init을 사용하고 싶지 않다면 프로퍼티에 초기값을 할당해줘도 된다.
        class Person {
            var name: String
            var age: Int = 27
            
            init(name: name){
                self.name = name
            }
        }

        let aiden = Person(name: "Aiden")// Person(name: "Aiden", age: 27)
        ```

    === "required"

        ```swift
        class SomeClass {
            required init() {
                // 이 곳에 이니셜라이저 구현 작성하기
            }
        }

        class SomeSubclass: SomeClass {
            required init() {
                // 이 곳에 자식 클래스의 필수 이니셜라이저 구현 작성하기
            }
        }
        ```
    
    === "Inheritance"

        ```swift
        class Shape {
            var numberOfSides: Int = 0
            var name: String

            init(name: String) {
            self.name = name
            }
        }

        class Square: Shape {
            var sideLength: Double = 0.0

            init(sideLength: Double, name: String) {
                self.sideLength = sideLength // 1. 자식 클래스의 속성에 값 할당
                super.init(name: name) // 부모 클래스의 이니셜라이저 호출
                numberOfSides = 4 // 부모 클래스가 정의한 속성 값 변경
            }
        }
        ```

!!! Struct
    nil 을 허용하는 형태의 var 만이 초기화시 자동 nil 할당.

    === "var"

        ```swift
        struct People {
            var weight: Double?
            var footSize: Int
            
            init(footSize: Int) {
                self.footSize = footSize
            }
        }

        var stark = People(footSize: 270)
        print(stark) // People(weight: nil, footSize: 270)
        stark.weight = 82.9
        print(stark) // People(weight: Optional(82.9), footSize: 270)
        ```

    === "let"

        ```swift
        struct People {
            let footSize: Int
            
            init(footSize: Int) {
                self.footSize = footSize
            }
        }

        let leedsStark = People(footSize: 270)
        print(leedsStark) // People(footSize: 270)
        ```

