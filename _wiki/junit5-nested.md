---
layout  : wiki
title   : JUnit5로 계층 구조의 테스트 코드를 작성하기
summary : 5의 @Nested 어노테이션을 쓰면 된다
date    : 2019-12-22 10:54:33 +0900
updated : 2019-12-22 19:53:40 +0900
tag     : java test
toc     : true
public  : true
parent  : index
latex   : false
---
* TOC
{:toc}

## Describe - Context - It 패턴

내가 선호하는 BDD 테스트 코드 작성 패턴이다.

이 패턴은 코드를 설명하는 테스트 코드를 작성하는 패턴이다.
[Better Specs][betterspecs]에 잘 설명되어 있으므로, 관심이 있다면 정독한다.

이 패턴은 `Describe`, `Context`, `It` 세 단어를 핵심 키워드로 삼는다.

* `Describe`는 설명할 테스트 대상을 명시하는 역할을 한다.
* `Context`는 테스트 대상이 특정 상황에 놓인 경우를 명시한다.
    * 영어로 `Context`문을 작성할 때에는 반드시 `with` 또는 `when`으로 시작하도록 한다.
* `It`은 테스트 대상의 행동을 설명한다.
    * `It returns true`, `It responds 404`와 같이 심플하게 설명할수록 좋다.

이 방식은 테스트 코드를 계층 구조로 만들어 주기 때문에 테스트 코드를 작성하거나 읽거나 분석할 때 스코프 범위만 신경쓰면 된다는 장점이 있다.
한편 "빠뜨린" 테스트 코드를 찾기 쉽기 때문에 높은 테스트 커버리지가 필요한 경우 큰 도움이 된다.


## JUnit5의 @Nested를 사용한 계층 구조의 `D-C-I` 테스트 코드를 작성


반드시 계층 구조가 아니어도 `D-C-I` 패턴의 테스트 코드를 작성하는 것은 가능하다.

그러나 테스트 코드가 계층 구조를 이루면 테스트 결과를 보기 좋다는 장점이 있다.

Java에서는 다른 언어와 달리 메소드 내부에 메소드를 곧바로 만들 수가 없고,
JUnit4가 이너 클래스로 작성한 테스트 코드를 특별히 처리하지 않아서 애매한 느낌이었는데
JUnit5의 `@Nested`를 사용하면 계층 구조의 테스트 코드를 작성할 수 있다는 것을 알게 되었다.

다음은 가볍게 작성한 클래스 하나를 테스트하는 코드를 IntelliJ에서 돌려본 결과를 캡처한 것이다.

![]( /post-img/junit5-nested/dci-eng.png )

한국어로 바꿔서도 해 보았다.

![]( /post-img/junit5-nested/dci-kor.png )

계층 구조이기 때문에 특정 범위를 폴드하는 것도 가능하다.

위의 테스트를 가동하는 데에 사용한 소스코드는 [내 저장소][example]에서 볼 수 있다.

* [ComplexNumber.java][example-1] - 복소수 클래스
* [ComplexNumberTest.java][example-eng] - 테스트 코드(영어)
* [ComplexNumberKoTest][example-kor] - 테스트 코드(한국어)

한편 html 보고서를 출력하면 다음과 같이 나온다.

![]( /post-img/junit5-nested/result.png )

### 예제 코드: ComplexNumberTest


```java
import static org.junit.jupiter.api.Assertions.*;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Nested;
import org.junit.jupiter.api.Test;

import static org.hamcrest.CoreMatchers.is;
import static org.hamcrest.MatcherAssert.assertThat;

@DisplayName("Describe: ComplexNumber class")
class ComplexNumberTest {

    @Nested
    @DisplayName("Describe: of method")
    class DescribeAdd {

        @Nested
        @DisplayName("Context: with a real number")
        class Context_with_naturals {
            private final double givenNatual = 3d;
            private ComplexNumber given = ComplexNumber.of(givenNatual);

            @Test
            @DisplayName("It returns a complex number with 0i.")
            void it_has_0_imagine_value() {
                assertThat(given.getImagine(), is((0d)));
            }
        }
    }

    @Nested
    @DisplayName("Describe: sum method")
    class DescribeSum {
        @Nested
        @DisplayName("Context: with two complex numbers with real and i parts")
        class Context_with_naturals {
            private ComplexNumber a, b;

            @BeforeEach
            void prePareNumbers() {
                a = ComplexNumber.of(1d, 2d);
                b = ComplexNumber.of(32d, 175d);
            }

            ComplexNumber subject() {
                return ComplexNumber.sum(a, b);
            }

            @Test
            @DisplayName("It returns a complex number with the sum of the two real values.")
            void it_returns_complex_has_each_real_sum() {
                final double expect = a.getReal() + b.getReal();
                final double result = subject().getReal();
                assertThat(result, is(expect));
            }

            @Test
            @DisplayName("It returns a complex number with the sum of the two i values")
            void it_returns_complex_has_each_imagine_sum() {
                final double expect = a.getImagine() + b.getImagine();
                final double result = subject().getImagine();
                assertThat(result, is(expect));
            }
        }
    }

    @Nested
    @DisplayName("Describe: toString")
    class GivenToString {
        @Nested
        @DisplayName("Context: with only real value")
        class Context_with_naturals {
            private final double givenNatual = 3d;
            private final String expectPattern = "^3(?:\\.0+)?$";
            private ComplexNumber given = ComplexNumber.of(givenNatual);

            @Test
            @DisplayName("It returns a string that represents only real value")
            void it_has_0_imagine_value() {
                assertTrue(given.toString().matches(expectPattern));
            }
        }

        @Nested
        @DisplayName("Context: with a real value and an imagine value")
        class Context_with_imagine {
            private final double givenNatual = 3d;
            private final double givenImagine = 7d;
            private ComplexNumber given = ComplexNumber.of(givenNatual, givenImagine);
            private String expectPattern = "^3(?:\\.0+)?\\+7(?:\\.0+)?i$";

            @Test
            @DisplayName("It returns a string in the form of real and i addition.")
            void it_has_0_imagine_value() {
                System.out.println(given);
                assertTrue(given.toString().matches(expectPattern));
            }
        }

    }
}
```


## 타 언어 테스트 프레임워크의 D-C-I 패턴

### Ruby

다음은 [Better Specs][betterspecs]에서 인용한 것이다.

```ruby
describe '#destroy' do

  context 'when resource is found' do
    it 'responds with 200'
    it 'shows the resource'
  end

  context 'when resource is not found' do
    it 'responds with 404'
  end

  context 'when resource is not owned' do
    it 'responds with 404'
  end
end
```

### Go - Ginkgo

Go 언어에서는 Ginkgo 테스트 프레임워크를 사용해 `Describe - Context - It` 패턴의 테스트 코드를 작성할 수 있다.

다음은 [Ginkgo: A Golang BDD Testing Framework][ginkgo]에서 인용한 것이다.

```go
var _ = Describe("Book", func() {
    var (
        longBook  Book
        shortBook Book
    )

    BeforeEach(func() {
        longBook = Book{
            Title:  "Les Miserables",
            Author: "Victor Hugo",
            Pages:  1488,
        }

        shortBook = Book{
            Title:  "Fox In Socks",
            Author: "Dr. Seuss",
            Pages:  24,
        }
    })

    Describe("Categorizing book length", func() {
        Context("With more than 300 pages", func() {
            It("should be a novel", func() {
                Expect(longBook.CategoryByLength()).To(Equal("NOVEL"))
            })
        })

        Context("With fewer than 300 pages", func() {
            It("should be a short story", func() {
                Expect(shortBook.CategoryByLength()).To(Equal("SHORT STORY"))
            })
        })
    })
})
```

### PHP - Kahan

Php 언어에서는 Kahlan으로 `Describe - Context - It` 패턴의 테스트 코드를 작성할 수 있다.

Kahlan을 쓰면 매우 세련된 느낌의 테스트 코드를 작성할 수 있었다. Php가 주력 언어가 아닌데도 Kahlan을 사용하면서 굉장히 좋다는 느낌을 받았었다.

다음은 [Kahlan github의 README.md][kahlan]에서 인용한 것이다.


```php
<?php

describe("Example", function() {

    it("makes an expectation", function() {

         expect(true)->toBe(true);

    });

    it("expects methods to be called", function() {

        $user = new User();
        expect($user)->toReceive('save')->with(['validates' => false]);
        $user->save(['validates' => false]);

    });

    it("stubs a function", function() {

        allow('time')->toBeCalled()->andReturn(123);
        $user = new User();
        expect($user->save())->toBe(true)
        expect($user->created)->toBe(123);

    });

    it("stubs a class", function() {

        allow('PDO')->toReceive('prepare', 'fetchAll')->andReturn([['name' => 'bob']]);
        $user = new User();
        expect($user->all())->toBe([['name' => 'bob']]);

    });

});
```

### Kotlin - Spek

Kotlin에는 [Spek][kotlin-spek]이 있다.

```kotlin
object CalculatorSpec: Spek({
    describe("A calculator") {
        val calculator by memoized { Calculator() }

        describe("addition") {
            it("returns the sum of its arguments") {
                assertEquals(3, calculator.add(1, 2))
            }
        }
    }
})
```

[betterspecs]: http://www.betterspecs.org/ko/
[ginkgo]: https://onsi.github.io/ginkgo/
[kahlan]: https://github.com/kahlan/kahlan
[kotlin-spek]: https://www.spekframework.org/specification/

[example]: https://github.com/johngrib/example-junit5/
[example-1]: https://github.com/johngrib/example-junit5/blob/56811e8647e115f11a6bf10c911734ea41a87677/src/main/java/com/johngrib/example/ComplexNumber.java
[example-eng]: https://github.com/johngrib/example-junit5/blob/56811e8647e115f11a6bf10c911734ea41a87677/src/test/java/com/johngrib/example/ComplexNumberTest.java
[example-kor]: https://github.com/johngrib/example-junit5/blob/56811e8647e115f11a6bf10c911734ea41a87677/src/test/java/com/johngrib/example/ComplexNumberKoTest.java
