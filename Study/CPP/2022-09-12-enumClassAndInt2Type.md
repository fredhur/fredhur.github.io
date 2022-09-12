---
layout: page
title: Enum Class To Type
description: >
  cpp 에 대한 내용들을 기록합니다.
hide_description: true
sitemap: false
permalink: /CPP/
---


# <font color="DodgerBlue">Introduction</font>

enum class 와 int2Type ( c++17 부터) 는 modern c++ 이 제공하는 강력한 기능이다.
이 글은 이 두가지를 조합하여 읽기 좋은 코드를 작성하는 방법을 다루려고 한다. 


# <font color="DodgerBlue">enum class</font>
enum class 는 modern c++ 에서 새로 나온 개념으로, enum 보다 더 강력한 기능을 제공한다. 
주요 특징은 아래 2가지로 정리 할 수 있다.

+ <font color="AquaBlue">별도의 namespace 를 가져서 같은 이름을 사용할 수 있다. </font>
   
~~~ cpp
#include <iostream>

enum class TrafficLight
{
    Red,
    Blue,
    Green
};
enum class Color
{
    Red,
    Blue,
    Green
};
int main()
{
    Color color = Color::Red;
    TrafficLight trafficLight = TrafficLight::Red;
}
~~~

+ <font color="AquaBlue">Type safe 하다.</font>

기존의 enum 과는 달리 int 로 자동 변환되지 않는다. 간혹 실수로 잘못된 enum 을 int 에 넣는걸 방지해줄 수 있다.

~~~ cpp
#include <cstdio>
enum class Color
{
    Red,
    Blue,
    Green
};

int main()
{
    int color = Color::Red; // compile error !!

}

~~~

# <font color="DodgerBlue">int2Type</font>

int2Type 자체만으로도 많은 내용을 다루어야한다. 페이스북의 모든 구조를 다 설계했다는 전설의 프로그래머가 만들었다고 배웠었는데, 그 분의 성함을 까먹었다.. 
어찌됐든 int2Type 은 아래처럼 타입의 값, 상수들로 다형성을 추구하기 위해서 만들었다고 우선 이해하자.

~~~ cpp

// int2Type 이 없다면, foo(0) 과 foo(1) 를 별도의 다른 함수로 분리가 불가능하다.

int foo(int n)
{
    if(n==0)
    {
        return 0;
    }

    else if(n==1)
    {
        return 1;
    }
    return -1;
}

int main()
{
    foo(0);
    foo(1);
}


~~~

~~~ cpp

// int2Type 을 활용한다면? 

#include <cstdio>
template <int N>
struct int2Type
{
    enum
    {
        value = N
    };
};

int foo(int)
{
    return -1;
}
int foo(int2Type<0>)
{
    return int2Type<0>::value;
}
int foo(int2Type<1>)
{
    return int2Type<1>::value;
}

int main()
{

    const auto t1 = foo(int2Type<0>::value);
    const auto t2 = foo(int2Type<0>());
    const auto t3 = foo(int2Type<1>());
    printf("t1 : %d\n", t1);
    printf("t2 : %d\n", t2);
    printf("t3 : %d\n", t3);

    // Result

    // t1 : -1
    // t2 : 0
    // t3 : 1

    return 0;
}


~~~

int2Type<0> 형식으로 사용해야 하지만 0 , 1 , 2 등의 정수 형으로 서로 다른 함수를 오버로딩 할 수 있다는 점이 매력적으로 다가온다.


# <font color="DodgerBlue"> enum class To type</font>

위 두가지 개념을 조합하여, enum class 로 선언된 변수를 타입으로 바꾸는 방법을 소개한다. 
딱히 명칭은 생각나지 않아 enumClass2Type 이라고 지칭하겠다. 

(c++11 부터는 integral_constant 라고 표준화 했다.)

~~~ cpp
#include <iostream>
using namespace std;

enum class TrafficLight
{
    Red,
    Blue,
    Green 
};

template <TrafficLight N> struct Enum2Type 
{
    static constexpr TrafficLight value = N;
};

void TurnTrafficLight(Enum2Type<TrafficLight::Red> )
{
    cout << "Turn Red" << endl;   
}
void TurnTrafficLight(Enum2Type<TrafficLight::Blue> )
{
    cout << "Turn Blue" << endl;   
}
void TurnTrafficLight(Enum2Type<TrafficLight::Green> )
{
    cout << "Turn Green" << endl;   
}

int main()
{

    constexpr TrafficLight light = TrafficLight::Blue;

    if constexpr(light == TrafficLight::Red)
    {
        TurnTrafficLight(Enum2Type<TrafficLight::Red>());
    }
    else if(light == TrafficLight::Green)
    {
        TurnTrafficLight(Enum2Type<TrafficLight::Blue>());
    }
    else 
    {
        TurnTrafficLight(Enum2Type<TrafficLight::Green>());
    }
    return 0;
}

~~~

# <font color="DodgerBlue">Usage</font>

int2Type intergral_constant 라는 명칭으로 modern c++ 에서 다양하게 쓰인다. is_pointer 처럼 내가 건네준 타입이 포인터인지 아닌지를 구분 할 수 있고 is_array 처럼 내가 준 타입이 배열인지 아닌지도 구분할 수 있게 해준다. 

나는 이걸 enum 타입이랑 같이 사용하여서 enum 변수 별 다른 함수를 호출 시킬때 쓰고 싶었다. 
이 부분이 정확히 어디에 잘 쓰일 수 있냐 라는 질문에는 "상황마다 다르다" 가 정답이겠지만, 나의 경우를 소개하자면 state pattern 으로 코딩할때 많이 사용하였다. 

각 state 의 이름을 enum class 타입으로 정의 해두고 각 state 마다 해야하는 행위를 함수 오버로딩으로 구현하였다.

# <font color="Crimson">참고 reference</font>

+ [8bitscoding 블로그](https://8bitscoding.github.io/cpp/template/int2type/)
+ [cppreference](https://en.cppreference.com/w/cpp/language/enum)