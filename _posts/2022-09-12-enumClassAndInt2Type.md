---
layout: post
title: Enum Class To Type
description: >
  enum class 와 int2Type 을 이용한 컨셉
hide_description: true
sitemap: true

---


## <font color="DodgerBlue">Introduction</font>

Enum class and int2Type are powerful grammar that modern c++ provides. 
This post proposes **EnumClass2Type** which is developed by using enum class and int2Type.

As can be inferred by title , This post deals with how to convert enum class to type.

## <font color="DodgerBlue">enum class</font>
The enum class is a powerful new concept in modern c++. <br>
The two main features are as follows.

+ <font color="AquaBlue">We can use same name since each name has different namespace scope. </font>
   
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

+ <font color="AquaBlue">Type safe concept.</font>

Unlike enum which used C, enum class does not convert automatically to int. It prevents the mistake of incorrectly assigning enum to int.

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

## <font color="DodgerBlue">int2Type</font>


The int2Type itself has a lot to do with it. I learned that it was made by a legendary programmer who designed all the structures of Facebook, but I forgot his name.
Anyway, let's first understand that int2Type was created to pursue polymorphism with type values ​​and constants as shown below.

~~~ cpp

// If there is no int2Type, foo(0) and foo(1) cannot be seperated.
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

// What if we use int2Type?

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

It should be used as int2Type<0> , but it is attractive that different functions can be overloaded with integer types such as 0 , 1 , 2 , etc.


## <font color="DodgerBlue"> enum class To type</font>

Combining the above two concepts, introduce a method to convert a variable declared as an enum class into a type.
I named this concept as **EnumClass2Type** 

(Since C++11, int2Type is standardized under the name integral_constant)

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

## <font color="DodgerBlue">Usage</font>

int2Type is called intergral_constant since c++11, and it is used variously in modern c++. Like **is_pointer**, it is possible to determine whether the type I passed is a pointer or not, and like **is_array**, it also allows out code to determine whether the type I gave is an array or not


I usually use Enumclass2Type when call function according to each enum type.
To the question of where exactly this part can be used well, the correct answer would be "every situation is different", but to introduce my case, it was used a lot when coding with state pattern.

The name of each state is defined as an enum class type, and the action to be performed in each state is implemented as function overloading.

## <font color="Crimson">Reference</font>

+ [8bitscoding Blog](https://8bitscoding.github.io/cpp/template/int2type/)
+ [cppreference](https://en.cppreference.com/w/cpp/language/enum)