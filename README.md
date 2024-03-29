# Vulkan Memory Allocator

Easy to integrate Vulkan memory allocation library.

**Documentation:** See [Vulkan Memory Allocator](https://gpuopen-librariesandsdks.github.io/VulkanMemoryAllocator/html/) (generated from Doxygen-style comments in [src/vk_mem_alloc.h](src/vk_mem_alloc.h))

**License:** MIT. See [LICENSE.txt](LICENSE.txt)

**Changelog:** See [CHANGELOG.md](CHANGELOG.md)

**Product page:** [Vulkan Memory Allocator on GPUOpen](https://gpuopen.com/gaming-product/vulkan-memory-allocator/)

**Build status:**

- Windows: [![Build status](https://ci.appveyor.com/api/projects/status/4vlcrb0emkaio2pn/branch/master?svg=true)](https://ci.appveyor.com/project/adam-sawicki-amd/vulkanmemoryallocator/branch/master)  
- Linux: [![Build Status](https://travis-ci.org/GPUOpen-LibrariesAndSDKs/VulkanMemoryAllocator.svg?branch=master)](https://travis-ci.org/GPUOpen-LibrariesAndSDKs/VulkanMemoryAllocator)

**❗ Please help to shape the future of the library by participating in the survey:**  
**[Vulkan Memory Allocator Survey March 2019](http://goo.gl/forms/0c5DTDjwDAaZOmJM2)**

# Problem

Memory allocation and resource (buffer and image) creation in Vulkan is difficult (comparing to older graphics API-s, like D3D11 or OpenGL) for several reasons:

- It requires a lot of boilerplate code, just like everything else in Vulkan, because it is a low-level and high-performance API.
- There is additional level of indirection: `VkDeviceMemory` is allocated separately from creating `VkBuffer`/`VkImage` and they must be bound together.
- Driver must be queried for supported memory heaps and memory types. Different IHVs provide different types of it.
- It is recommended practice to allocate bigger chunks of memory and assign parts of them to particular resources.

# Features

This library can help game developers to manage memory allocations and resource creation by offering some higher-level functions:

1. Functions that help to choose correct and optimal memory type based on intended usage of the memory.
   - Required or preferred traits of the memory are expressed using higher-level description comparing to Vulkan flags.
2. Functions that allocate memory blocks, reserve and return parts of them (`VkDeviceMemory` + offset + size) to the user.
   - Library keeps track of allocated memory blocks, used and unused ranges inside them, finds best matching unused ranges for new allocations, respects all the rules of alignment and buffer/image granularity.
3. Functions that can create an image/buffer, allocate memory for it and bind them together - all in one call.

Additional features:

- Well-documented - description of all functions and structures provided, along with chapters that contain general description and example code.
- Thread-safety: Library is designed to be used by multithreaded code.
- Configuration: Fill optional members of CreateInfo structure to provide custom CPU memory allocator, pointers to Vulkan functions and other parameters.
- Customization: Predefine appropriate macros to provide your own implementation of all external facilities used by the library, from assert, mutex, and atomic, to vector and linked list. 
- Support for memory mapping, reference-counted internally. Support for persistently mapped memory: Just allocate with appropriate flag and you get access to mapped pointer.
- Support for non-coherent memory. Functions that flush/invalidate memory. `nonCoherentAtomSize` is respected automatically.
- Support for sparse binding and sparse residency: Convenience functions that allocate or free multiple memory pages at once.
- Custom memory pools: Create a pool with desired parameters (e.g. fixed or limited maximum size) and allocate memory out of it.
- Linear allocator: Create a pool with linear algorithm and use it for much faster allocations and deallocations in free-at-once, stack, double stack, or ring buffer fashion.
- Support for VK_KHR_dedicated_allocation extension: Just enable it and it will be used automatically by the library.
- Defragmentation of GPU and CPU memory: Let the library move data around to free some memory blocks and make your allocations better compacted.
- Lost allocations: Allocate memory with appropriate flags and let the library remove allocations that are not used for many frames to make room for new ones.
- Statistics: Obtain detailed statistics about the amount of memory used, unused, number of allocated blocks, number of allocations etc. - globally, per memory heap, and per memory type.
- Debug annotations: Associate string with name or opaque pointer to your own data with every allocation.
- JSON dump: Obtain a string in JSON format with detailed map of internal state, including list of allocations and gaps between them.
- Convert this JSON dump into a picture to visualize your memory. See [tools/VmaDumpVis](tools/VmaDumpVis/README.md).
- Debugging incorrect memory usage: Enable initialization of all allocated memory with a bit pattern to detect usage of uninitialized or freed memory. Enable validation of a magic number before and after every allocation to detect out-of-bounds memory corruption.
- Record and replay sequence of calls to library functions to a file to check correctness, measure performance, and gather statistics.

# Prequisites

- Self-contained C++ library in single header file. No external dependencies other than standard C and C++ library and of course Vulkan. STL containers are not used by default.
- Public interface in C, in same convention as Vulkan API. Implementation in C++.
- Error handling implemented by returning `VkResult` error codes - same way as in Vulkan.
- Interface documented using Doxygen-style comments.
- Platform-independent, but developed and tested on Windows using Visual Studio. Continuous integration setup for Windows and Linux. Used also on Android, MacOS, and other platforms.

# Example

Basic usage of this library is very simple. Advanced features are optional. After you created global `VmaAllocator` object, a complete code needed to create a buffer may look like this:

```cpp
VkBufferCreateInfo bufferInfo = { VK_STRUCTURE_TYPE_BUFFER_CREATE_INFO };
bufferInfo.size = 65536;
bufferInfo.usage = VK_BUFFER_USAGE_VERTEX_BUFFER_BIT | VK_BUFFER_USAGE_TRANSFER_DST_BIT;

VmaAllocationCreateInfo allocInfo = {};
allocInfo.usage = VMA_MEMORY_USAGE_GPU_ONLY;

VkBuffer buffer;
VmaAllocation allocation;
vmaCreateBuffer(allocator, &bufferInfo, &allocInfo, &buffer, &allocation, nullptr);
```

With this one function call:

1. `VkBuffer` is created.
2. `VkDeviceMemory` block is allocated if needed.
3. An unused region of the memory block is bound to this buffer.

`VmaAllocation` is an object that represents memory assigned to this buffer. It can be queried for parameters like Vulkan memory handle and offset.

# Read more

See **[Documentation](https://gpuopen-librariesandsdks.github.io/VulkanMemoryAllocator/html/)**.

# Software using this library

- **[Filament](https://github.com/google/filament)** - physically based rendering engine for Android, Windows, Linux and macOS, from Google. Apache License 2.0.
- **[Skia](https://github.com/google/skia)** - complete 2D graphic library for drawing Text, Geometries, and Images, from Google.
- **[PowerVR SDK](https://github.com/powervr-graphics/Native_SDK)** - C++ cross-platform 3D graphics SDK, from Imagination. License: MIT.
- **[Vulkan Best Practice for Mobile Developers](https://github.com/ARM-software/vulkan_best_practice_for_mobile_developers)** from ARM. License: MIT.
- **[Anvil](https://github.com/GPUOpen-LibrariesAndSDKs/Anvil)** - cross-platform framework for Vulkan. License: MIT.
- **[vkQuake2](https://github.com/kondrak/vkQuake2)** - vanilla Quake 2 with Vulkan support. License: GNU GPL.
- **[vkDOOM3](https://github.com/DustinHLand/vkDOOM3)** - Vulkan port of GPL DOOM 3 BFG Edition. License: GNU GPL.
- **[Lightweight Java Game Library (LWJGL)](https://www.lwjgl.org/)** - includes binding of the library for Java. License: BSD.
- **[The Forge](https://github.com/ConfettiFX/The-Forge)** - cross-platform rendering framework. Apache License 2.0.
- **[VK9](https://github.com/disks86/VK9)** - Direct3D 9 compatibility layer using Vulkan. Zlib lincese.

[Many other projects on GitHub](https://github.com/search?q=AMD_VULKAN_MEMORY_ALLOCATOR_H&type=Code) and some game development studios that use Vulkan in their games.

# See also

- **[Awesome Vulkan](https://github.com/vinjn/awesome-vulkan)** - a curated list of awesome Vulkan libraries, debuggers and resources.
- **[vk-mem](https://github.com/gwihlidal/vk-mem-rs)** - Rust binding for this library. Author: Graham Wihlidal. License: Apache 2.0 or MIT.
- **[PyVMA](https://github.com/realitix/pyvma)** - Python wrapper for this library. Author: Jean-Sébastien B. (@realitix). License: Apache 2.0.
- **[vma_sample_sdl](https://github.com/rextimmy/vma_sample_sdl)** - SDL port of the sample app of this library (with the goal of running it on multiple platforms, including MacOS). Author: @rextimmy. License: MIT.
- **[vulkan-malloc](https://github.com/dylanede/vulkan-malloc)** - Vulkan memory allocation library for Rust. Based on version 1 of this library. Author: Dylan Ede (@dylanede). License: MIT / Apache 2.0.
