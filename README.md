# C++ Primer
## The Modern C++ Programming Language

Latest (2023/2026) or unsupported features will be marked explicitly or omitted.

### Literal
Literal is a fixed value written directly into the source code.
C++ categorizes literals based on their data type:

#### 1. Integer Literals  

  Type prefixes:
  | Prefix | Base |
  | ------ | ---- |
  | None     | base-10 |
  | 0b or 0B | base-2  |
  | 0        | base-8  |
  | 0x or 0X | base 16 |
  
  Type suffixes:
  | Suffix | Type |
  | ------ | ---- |
  | None     | int         |
  |U or u    | unsigned int|
  |L or l    | long        |
  |LL or ll  | long long   |
  |UL or ul  | unsigned long      |
  |ULL or ull| unsigned long long |
  |Z or z    | size_t/ ptrdiff_t (C++23) |

#### 2. Floating-Point Literals
  
  Decimal or scientific notation (3.14159e-42)
  Type suffixes:
  | Suffix | Type |
  | ------ | ---- |
  | None   | double |
  |F or f  | float  |
  |L or l  | long double |
  
  - One can use single quotes (`'`, ignored by compiler) as digit separators to improve the readability of long numbers.
  - For exact precision modeling without decimal rounding errors, one can express floating-point literals in hexadecimal. They begin with a 0x or 0X prefix, and the exponent is designated by p or P (mandatory), representing a power of 2. `double hex_lit = 0x1.4p3;`

#### 3. Character Literals 
[character_literal](https://en.cppreference.com/cpp/language/character_literal)

| Prefix | Char type | String type (`s` suffix)| Encoding |
| --- | --- | --- | --- |
| `L'.'`  | wchar_t  | std::wstring   | platform dependent (UTF-16 for Win, UTF-32 for *nix) |
| `u8'.'` | char8_t  | std::u8string  | UTF-8  |
| `u'.'`  | char16_t | std::u16string | UTF-16 |
| `U'.'`  | char32_t | std::u32string | UTF-32 |

#### 4. String Literals
[string_literal](https://en.cppreference.com/cpp/language/string_literal)

String literals can be used to initialize character arrays. 
`L"..."`: const wchar_t[], `u8"..."`: const char8_t[], `u"..."`: const char16_t[], `U"..."`: const char32_t[], \0 is appended.

R prefix can be combined with encoding prefixes to bypass character escaping rules, syntax is R"delimiter(text)delimiter", delimiter is needed when the string contains '(' or ')'.
`R"fn(y = f(x))fn"` // forms `"y = f(x)"`

Type suffixes:
s: std:string, sv: std::string_view

Adjacent string literals are concatenated.
`"Hello, " "world!"` // the 2 string literals form `"Hello, world!"`

If the two string literals are of the same kind, the concatenated string literal is also of that kind. If an ordinary string literal is adjacent to a non-ordinary (with prefixes L, u8, u or U) string literal, the concatenated string literal is of the kind of the latter.

### User-defined literals
[user_literal](https://en.cppreference.com/cpp/language/user_literal)

One can produce objects of user-defined type by defining a user-defined suffix for integer, floating, character and string literals.
Examples: `12_km`, `0.5_Pa`, `'c'_X`, `"abd"_L` or `u"xyz"_M`

Implemented as `operator""_suffix()`, parameters are for integer literal: unsigned long long, floating point: long double, character: char, wchar_t, char16_t, char32_t, string: `(const char*, std::size_t)`. Can be implemented as `ReturnType operator"" _suffix(const char*)` or `template<char...> ReturnType operator"" _suffix()` (only for integer and floating literals).

Space between `"" _suffix` is obsolete in C++23.

Suffixes defined by standard library:
`h, min, s, ms, us, ns`: std::chrono::duration
`y`: std::chrono::year
`d`: std::chrono::day
`i, if, il`: std::complex
`s`: std::string
`sv`: std::string_view

Template example:
```
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


