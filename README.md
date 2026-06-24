# C++ Primer
## The Modern C++ Programming Language

Latest (2023/2026) or unsupported features will be marked explicitly or omitted.

### Literal
Literal is a fixed value written directly into the source code.
C++ categorizes literals based on their data type:

#### 1. Integer Literals  

[integer_literal](https://en.cppreference.com/cpp/language/integer_literal)

  Type prefixes:
  | Prefix | Base |
  | ------ | ---- |
  | None     | base-10 |
  | `0b or 0B` | base-2  |
  | `0`        | base-8  |
  | `0x or 0X` | base-16 |
  
  Type suffixes:
  | Suffix | Type |
  | ------ | ---- |
  | None               | int         |
  |`U or u`            | unsigned int|
  |`L or l`            | long        |
  |`LL or ll`          | long long   |
  |`ul or uL or Ul or UL`    | unsigned long      |
  |`ull or uLL or Ull or ULL`| unsigned long long |
  |`Z or z`                  | size_t/ptrdiff_t (C++23) |

#### 2. Floating-Point Literals

[floating_literal](https://en.cppreference.com/cpp/language/floating_literal)

  Decimal or scientific notation (3.14159e-42)
  Type suffixes:
  | Suffix | Type |
  | ------ | ---- |
  | None     | double |
  |`F or f`  | float  |
  |`L or l`  | long double |
  
  - One can use single quotes (`'`, ignored by compiler) as digit separators to improve the readability of long numbers.
  - For exact precision modeling without decimal rounding errors, one can express floating-point literals in hexadecimal. They begin with a 0x or 0X prefix, and the exponent is designated by p or P (mandatory), representing a power of 2. `double hex_lit = 0x1.4p3;`

#### 3. Character Literals 
[character_literal](https://en.cppreference.com/cpp/language/character_literal)

| Prefix | Char type | String type (`s` suffix)| Encoding |
| --- | --- | --- | --- |
| `L'.'`  | `wchar_t`  | `std::wstring`   | platform dependent (UTF-16 for Win, UTF-32 for *nix) |
| `u8'.'` | `char8_t`  | `std::u8string`  | UTF-8  |
| `u'.'`  | `char16_t` | `std::u16string` | UTF-16 |
| `U'.'`  | `char32_t` | `std::u32string` | UTF-32 |

#### 4. String Literals
[string_literal](https://en.cppreference.com/cpp/language/string_literal)

String literals can be used to initialize character arrays. 
`L"..."` has type `const wchar_t[]`, `u8"..."` - `const char8_t[]`, `u"..."` - `const char16_t[]`, `U"..."`- `const char32_t[]`, \0 is appended.

`R` prefix can be combined with encoding prefixes to bypass character escaping rules, syntax is R"delimiter(text)delimiter", delimiter is needed when the string contains '(' or ')'.
`R"fn(y = f(x))fn"` // forms `"y = f(x)"`

Type suffixes:
`s`: `std:string`, `sv`: `std::string_view`
TODO: What are the benefits of using `sv` suffix?

Adjacent string literals are concatenated.
`"Hello, " "world!"` // the 2 string literals form `"Hello, world!"`

If the two string literals are of the same kind, the concatenated string literal is also of that kind. If an ordinary string literal is adjacent to a non-ordinary (with prefixes L, u8, u or U) string literal, the concatenated string literal is of the kind of the latter.

### User-defined literals
[user_literal](https://en.cppreference.com/cpp/language/user_literal)

One can produce objects of user-defined type by defining a user-defined suffix for integer, floating, character and string literals.
Examples: `12_km`, `0.5_Pa`, `'c'_X`, `"abd"_L` or `u"xyz"_M`

Implemented as `operator""_suffix()`, parameters are for integer literal: `unsigned long long`, floating point: `long double`, character: `char, wchar_t, char16_t, char32_t`, string: `(const char*, std::size_t)`. 
Can be implemented as `ReturnType operator"" _suffix(const char*)` or `template<char...> ReturnType operator"" _suffix()` (only for integer and floating literals).

Space between `"" _suffix` is obsolete in C++23.
TODO: Why it was needed previously?

Suffixes defined by standard library:
| Suffix | Type |
| --- | --- |
|`h, min, s, ms, us, ns` | `std::chrono::duration` |
|`y` | `std::chrono::year` |
|`d` | `std::chrono::day`  |
|`i, if, il` | `std::complex<double>, std::complex<float>, std::complex<long double>` |
|`s` | `std::string`         |
|`sv` | `std::string_view`   |

Template example:
```cpp
template<char... Chars>
struct BinaryParser;

template<>
struct BinaryParser<> {
    static constexpr int value = 0;
};

template<char Head, char... Tail>
struct BinaryParser<Head, Tail...> {
    static_assert(Head == '0' || Head == '1', "Error: Only '0' and '1' allowed!");
    static constexpr int value = ((Head - '0') << sizeof...(Tail)) + BinaryParser<Tail...>::value;
};

// Raw literal template operator
template<char... Chars>
constexpr int operator"" _b() {
    return BinaryParser<Chars...>::value;
}

int main() {
    constexpr int parsed = 1101_b; // Valid compilation: evaluates to 13
    std::cout << parsed << std::endl;

    // auto faulty = 1021_b; 
    // ^ Uncommenting this triggers a compile time error via static_assert
}
```

### Using keyword
[using_keyword](https://en.cppreference.com/cpp/keyword/using)

#### Using directive

`using namespace std;`

The using-directive pulls an entire namespace into the current scope, meaning you can reference its members without a prefix.

#### Using declaration

`using std::cout; // cout << "...";`

The using-declaration pulls only a single, specific item from a namespace into the current scope.

#### Type alias

`using vector_of_strings = std::vector<std::string>;`

Create alias for types, modern alternative for `typedef`.

#### Template alias
```
template <typename T>
using map_string = std::map<std::string, T>;
map_string<int> phone_book; // equivalent to std::map<std::string, int> phone_book;
```cpp

Create a partial template alias using a `using` keyword.

#### Unhide base class overloads

If a derived class defines a function with the same name as a function in the base class, it hides all overloaded variants of that function from the base class. One can use `using` to bring the base class overloads back into scope.

```cpp
class Base {
public:
    void print(int x) {}
    void print(double y) {}
};

class Derived : public Base {
public:
    using Base::print; // Brings both int and double overloads into scope
    void print(std::string s) {} // Does not hide base overloads anymore
};
```

#### Change access specifiers

One can change the visibility of inherited members in a derived class using a `using` declaration.

```cpp
class Base {
protected:
    int internalValue;
};

class Derived : private Base {
public:
    using Base::internalValue; // Promotes internalValue to public visibility
};
```

#### Inheriting Constructors

One can inherit all constructors of a base class without rewriting them manually.

```cpp
class Base {
public:
    Base(int arg) {}
    Base(double arg, int arg2) {}
};

class Derived : public Base {
public:
    using Base::Base; // Automatically inherits both constructors
};
```

#### Using enum declarations

This feature allows to pull the enumerators of a scoped enum directly into your local scope, cutting down on verbose prefixes (C++20).

```cpp
enum class Status {Active, Inactive, Pending};
void checkStatus(Status s) {
    using enum Status; // Pulls Active, Inactive, Pending into local scope
    if (s == Active) { /* ... */ }
}
```

## Variadic templates

Variadic templates are templates with one or more parameter packs as a template type parameters.
Syntax `typename... Args` declares a pack that holds zero or more type parameters.
The `sizeof...` operator returns a std::size_t corresponding to the number of elements in a parameter pack.

### Parameter pack expansion

Here is the list of context where parameter pack can be expanded.

#### Function parameter list

```cpp
template<typename...T> 
std::tuple<T...> create_tuple(T&&...t) {
    return std::make_tuple<T...>(std::forward<T>(t)...);
}
const auto tuple = create_tuple(0, "word"s);
```

#### Initializer list

```cpp
template<std::same_as<int>...T>
void aggregate(T...t) {
    for (auto i: std::initializer_list<int>{t...}) {
    }
}
```

#### Base initializer list + mem-initializer list in constructor

```cpp
template<typename ...Base>
    struct MyStruct :  Base... {
        MyStruct();
    };
template<typename ...Base>
MyStruct<Base...>::MyStruct() : Base()... {}
```

#### Template argument list

`std::make_tuple<T...>`

In the following case, the ellipsis plays double-duty, serving at once to expand one parameter pack and capture another:
```cpp
template<typename ...T> struct Outer {
    template<T...V> struct Inner {
      };
    };
```

#### Using declaration

```cpp
template<typename ...Base>
    struct Derived :  Base... {
        using Base::fn...;
    };
```

#### Lambda capture list

```cpp
void fn(auto...arg) {
    auto with_copy = [arg...]{
    };
    with_copy();

    auto with_reference = [&arg...]{
    };
    with_reference();
}
```

#### Folds
Special form of a pack expansion

 Let ⊕ stand for any binary operator in the C++ grammar (.*, ->*, *, /, %, +, -, <<, >>, <=>, <, <=, >, >=, ==, !=, &, ^, |, &&, ||, =, +=, -=, *=, /=, %=, <<=, >>=, &=, ^=, |=, or the comma operator “,”)

 | Fold type | Form | Equivalent expression|
 |---|---|---|
 |binary left fold | (e⊕...⊕pat) | (((e ⊕ p1) ⊕ p2) ⊕ ⋯) ⊕ pn |
 |unary left fold  | (...⊕pat) | ((p1 ⊕ p2) ⊕ ⋯) ⊕ pn |
 |binary right fold | (pat⊕...⊕e) | p1⊕(p2⊕(⋯⊕(pn⊕e))) |
 |unary right fold | (pat⊕...) | p1⊕(p2⊕(⋯ ⊕ pn)) |


### Capturing parameter packs

#### Template parameter packs
#### Function parameter packs
Function parameter packs consist of values in the argument list of a function. There is a big restriction that a function parameter pack must either itself be a pack expansion `void aggregate(T...t)` or else contain the placeholder `auto`: `void aggregate(std::convertible_to<int> auto ...t).`.
