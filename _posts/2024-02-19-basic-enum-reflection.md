---
layout: post
title: Basic enum reflection 
---

As basic and simple as it can be[^1]. For a full reflection with heavy compiler torture and spending a basket of CPU cycles for it, use magic_enum or reflect-cpp or similar.

```
#include <cassert>
#include <cstdint>
#include <cstddef>
#include <source_location>
#include <string>
#include <string_view>
#include <type_traits>
#include <utility>

#include <cxxabi.h>
#include <cstdlib>

namespace lpt {

namespace impl {

// constexpr auto lpt::impl::name_and_val() [with E = main()::Color; E V = main::Color::eRED]
constexpr std::string_view enum_val_name(std::string_view name) noexcept 
{
    int i(name.length() - 1);
    for (i; i >=0; --i) 
    {
        if (name[i] == ' ')
        {
            break;
        }
    }

    return std::string_view(&name[i], name.length() -  1 - i); // main::Color::eRED
}

template <typename E, E V>
constexpr auto name_and_val() noexcept 
{
    static_assert(std::is_enum_v<E>, "not an enum");
    return enum_val_name(std::source_location::current().function_name());
}

template <typename E, E V>
constexpr auto enum_name() noexcept 
{
    constexpr auto name = name_and_val<E, V>();
    return std::string_view{name}; 
}

template <typename E, E V>
inline constexpr auto enum_name_v = enum_name<E, V>();

}  // namespace impl


/**
   Stringizing an enum
   \code
       #include <lpt/enum_tools.hpp>
       enum class Color { eRED=-199, eGREEN=0, eYELLOW=5 };
       std::cout << lpt::to_string<Color::eRED>();
   \endcode
 */

template <auto V>
requires (std::is_enum_v<decltype(V)>)
constexpr auto to_string() noexcept 
{
    using D = std::decay_t<decltype(V)>;
    constexpr std::string_view name = impl::enum_name_v<D, V>;
    static_assert( ! name.empty(), "enum value has no name");
    return name;
}

} //namespace lpt
```

[^1]: [https://github.com/melintea/lpt-tools/blob/main/include/lpt/enum_tools.hpp](https://github.com/melintea/lpt-tools/blob/main/include/lpt/enum_tools.hpp)
