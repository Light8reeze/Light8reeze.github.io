---
title: "C++ 17 기능 정리"
categories:
  - C++
tags:
  - C++
  - C++17
last_modified_at: 2020-08-08T14:25:52-05:00
---
## Constexpr if

constexpr if는 컴파일 타임에 결정되는 if문이다

```c++
template<typename T>
void foo(T value)
{
	if constexpr (std::is_pointer<T>)
		std::cout << "T Is Pointer" << std::endl;
	else constexpr
		std::cout << "T Is Not Pointer" << std::endl;
}
```

위 코드에서 T의 타입에 따라 컴파일이 결정된다. T가 포인터 타입일 경우 std::is_pointer<T> 아래의 식이 컴파일이 되고 조건에 맞지 않는 식은 컴파일에서 제외된다.



## template auto

template auto는 템플릿 타입을 자동으로 추론하는 키워드이다. 다음과 같이 사용 가능하다.

```c++
template<typename T, T value>
class CConstant
{
	constexpr T constant = value;
};

template<auto T>
class CConstantAuto
{
	constexpr auto constant = T;
};

int main()
{
	// template<auto>를 사용하지 않을 경우 타입을 넣어야 한다.
	CConstant<int, 1> aConst;

	CConstantAuto<1> aConstAuto;
	return 0;
}
```



## std::variant<A,B,C,...>

 std::varient<A, B, C, ...>은 타입에 안전한 Union 클래스이다. 

기본 사용방법은 다음과 같다.

```c++
using var = std::variant<int, float>;
var a = 5;

std::cout << std::get<int>(a) << std::endl; // 5
std::cout << std::get<0>(a) << std::endl; // int 형 값 반환(5)
std::cout << std::get<float>(a) << std::endl; // std::bad_variant_access 예외 발생
std::cout << std::get<1>(a) << std::endl; // std::bad_variant_access 예외 발생

// int형 값을 설정했으므로 int형 인덱스 0이 반환된다.
std::cout << a.index() << std::endl;

std::cout << std::get_if<int>(&a) << std::endl; // int* 형 값을 반환한다.
std::cout << std::get_if<float>(&a) << std::endl; // nullptr을 반환한다.
```

std::variant는 std::visit를 이용하여 variant에 있는 타입에 따른 오버로딩을 처리할 수 있다.

```c++
template<class... Ts> struct overloaded : Ts... { using Ts::operator()...; };
template<class... Ts> overloaded(Ts...)->overloaded<Ts...>;

int main()
{
	using var = std::variant<int, float>;
	var a = 5.5f;

	// float가 출력된다.
	std::visit([](auto&& arg)
	{
		using t = std::decay_t<decltype(arg)>;

		if constexpr (std::is_same<t, int>)
			std::cout << "int" << std::endl;
		else if constexpr (std::is_same<t, float>)
			std::cout << "float" << std::endl;
	}, a);

	var a = 5;
	// int가 출력된다.
	std::visit(overloaded
	{
		[](float& pArgs) { std::cout << "float" << std::endl; },
		[](int& pArgs) { std::cout << "int" << std::endl; }
	}, a);

	return 0;
}
```

위와 같이 std::variant에서 타입별로 로직 처리가 가능하다.



## std::any

std::any는 아무 값이나 담을 수 있는 클래스이다. 그러나 복사 가능한 형식만 담을 수 있다.

```c++
std::any a;
a.emplace<int>(5); // a = 5로도 사용 가능하다.

std::cout << a.type().name() << std::endl; // int 출력
std::cout << std::any_cast<int>(a) << std::endl; // 5 출력
```

위 코드처럼 아무 값이나 넣을 수 있고, std::any_cast를 이용하여 값을 사용 할 수 있다. std::any_cast는 std::any를 특정 형식으로 변환시켜주는 함수이다. 형식에 맞지 않을경우 std::bad_any_cast예외를 발생한다. 이러한 형식 안정성 때문에 std::any를 안전한 void*라고 하기도 한다. std::any를 이용하여 std::vector, 배열을 이용하여 다양한 값을 담을 수 있다.

```c++
int			a = 5;
double		b = 5.3;
std::string c = "asdf";
// std::any를 사용하지 않았을 경우
std::vector<void*> void_vector = { &a, &b, &c};
// std::any를 사용할 경우
std::vector<std::any> any_vector = { a, b, c};
```



## Reference

- [[C++ Korea 2nd Seminar] C++17 Key Features Summary](https://www.slideshare.net/utilforever/c-korea-2nd-seminar-c17-key-features-summary)
- [cpp17_in_TTs](https://github.com/tvaneerd/cpp17_in_TTs)