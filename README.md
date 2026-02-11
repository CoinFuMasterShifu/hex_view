# hex_view
A C++23 wrapper of `std::string_view` for simple hexadecimal parsing. Written for the Warthog crypto platform.

## Features:
- Header only ([hex_view.hxx](hex_view.hxx))
- On-the-fly conversion
- Ranges support
- Configurable exceptions
- `constexpr` support
- Iterators `begin()` and `end()` for accessing parsed bytes
- Support to write to an arbitrary `std::output_iterator` like `std::back_inserter_iterator`

## Example Usage
See `./example.cpp`:
```c++
#include "hex_view.hxx"

#include <algorithm>
#include <cassert>
#include <stdexcept>
#include <string_view>
#include <vector>

int main() {

  // Can use byte type of your project like std::byte, char, unsinged char etc.
  using byte_t = std::byte;
  using HexView = hex_view<byte_t, 
    // function that generates exception when called with no arguments
    [] static { return std::runtime_error("Invalid hex string"); }
    >;

  // Constexpr support
  constexpr HexView hex("a1b2c3d4");
  
  // Iterator support
  std::vector vec1(hex.begin(), hex.end());

  // Ranges support
  static_assert(std::ranges::sized_range<HexView>);
  static_assert(std::ranges::forward_range<HexView>);
  auto vec2(hex | std::ranges::to<std::vector>());

  // Conversion to std::array throwing if lengths don't match
  std::array<byte_t, 4> arr1(hex);
  assert(std::ranges::equal(arr1, hex));

  // Supports writing to std::output_iterator
  std::vector<byte_t> vec3;
  hex.insert_into(std::back_inserter(vec3));

  // Supports writing to std::span<byte_t>
  std::array<byte_t, 4> arr2;
  if (!hex.parse_to(arr2)) {
    /* handle error, invalid hex or length */
  }

  // Also constexpr support
  constexpr auto arr3{[&] {
    std::array<byte_t, 4> arr{};
    hex.place_into_throw(arr);
    return arr;
  }()};

  // Check at compile time, it holds "a1b2c3d4".
  static_assert(arr3[0] == byte_t(0xa1) && arr3[1] == byte_t(0xb2) &&
                arr3[2] == byte_t(0xc3) && arr3[3] == byte_t(0xd4));
}
```
