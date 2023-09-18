On Linux, one way to generate a GUID (often referred to as UUID on Linux platforms) is to utilize the `libuuid` library. This library comes with the `uuid-dev` package on many distributions.

Here's a step-by-step guide:

1. **Install `libuuid` Development Libraries**:
   - On Debian-based systems (like Ubuntu):
     ```
     sudo apt-get install uuid-dev
     ```

   - On RedHat-based systems (like Fedora):
     ```
     sudo yum install libuuid-devel
     ```

2. **Write the C++ Code**:
```cpp
#include <iostream>
#include <uuid/uuid.h>

int main() {
    uuid_t uuid;
    char uuid_str[37];  // 36 characters plus the null terminator

    // Generate UUID
    uuid_generate_random(uuid);

    // Convert UUID to string
    uuid_unparse(uuid, uuid_str);

    std::cout << "Generated UUID: " << uuid_str << std::endl;

    return 0;
}
```

3. **Compile and Run**:
   ```
   g++ -o generate_uuid your_filename.cpp -luuid
   ./generate_uuid
   ```

This program should print out a UUID every time you run it.