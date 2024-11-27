# Function Loading

There are definietly better docs than this one, this just boils it down to qhat i've decided to do.

## Indirect and direct loading

Vulkan can load its fucntion pointers through 2 methods direct an indirect.

### Direct loading
Direct function loading occurs by immediately linking the vulkan.lib file statically into the executable. Its easy however it will not be consistent across any devices using your vulkan application. Its possible that vulkan driver on the users computer will not have the proper functions meaning it will sort of just fail. 

### Indirect loading
Indirect loading calls dlsym(posix) or GetModule(windows?) in order to load the vulkan system dynamic library. You then must manually load all the function pointers manually. [Volk](https://github.com/zeux/volk) is a good example of that. The benefits of this are three fold, you have the ability to be selective and catch errors in case the user pc doesn't support a function (old driver, different platforms), be explicit about multiple device support (aka non-homogenous gpu parallel workloads), and there are theoretical performance benefits.

#### Function tables
While loading functions each one falls into 3 levels, *global, instance, or device*. Global functions work at the loader level and don't require anything except the loading of the function pointer. Instance functions require a vkGetInstanceProcAddr with an instance attached. Any functions that take a VkInstance or a child (aka made with VkInstance) fall into this group. Device functions require a vkGetDeviceProcAddr with a device, any function requiring a VkDevice or children (aka made with VkDevice) are loaded via this path. According to volk the explicit loading of instance and device functions reduced overhead of function calls by up to 7% as the loader layer didn't need to determine instance and devices. 
