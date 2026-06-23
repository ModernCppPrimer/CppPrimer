#### CppPrimer
The Modern C++ Programming Language
Latest (2023/2026) or unsupported features will be marked explicitly or omitted.

# String literals
String literal prefixes change the underlying character type and string encoding:
- L"..."  wchar_t std::wstring, platform dependent (UTF-16 for Win, UTF-32 for *nix)
- u8"..." char8_t std::u8string, UTF-8
- u"..." char16_t std::u16string, UTF-16
- U"..." char32_t std::u32string, UTF-32
