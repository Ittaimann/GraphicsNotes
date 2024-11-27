# Vulkan Synchronization

[khronos docs](https://www.khronos.org/blog/understanding-vulkan-synchronization)

## QueueSync
### CPU-GPU
on VkQueueSubmit one of the input/outputs is a VkFence. VkFence allows the cpu to check for completion of the queue. 

CPU                 GPU                     CPU
submit          -> does work             -> do more work
fence not done  -> fence set to complete -> wait until fences done

### GPU-GPU aka multi Queue sync
On VkQueueSubmit in the VkSubmitInfo there is a VkSempahore count/array combo for waits and signals. Once a queue submit completesits work any queues waiting on the semaphores will begin to execute. 

submit queue 1, no waits, signals semaphore 1
submit queue 2, waits on semaphore 1, no signals

in this case submitting queue 1 and 2 at the same time will be ok as queue 2 won't complete until
queue 1 is done with all its operations
[docs](https://registry.khronos.org/vulkan/specs/1.3-extensions/html/vkspec.html#synchronization-semaphores-signaling)
