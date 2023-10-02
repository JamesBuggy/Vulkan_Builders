# Vulkan_Builders
This Repo contains a collection of classes which wrap Vulkan components and their associated [builders](https://en.wikipedia.org/wiki/Builder_pattern).

Using these builders provides several benefits:
- Code is more readable and self-explanatory.
- The complex Vulkan setup details are abstracted behind a clean and consistent API.
- The configuration of different Vulkan objects are separated into distinct builders, making it easier to manage and update specific parts of the setup.
- Extension and modification of the setup process can easily be done by adding or modifying builder methods as needed.

## Example Usage:
``` cpp
#ifdef _DEBUG
  std::vector<const char*> validationLayers = { "VK_LAYER_KHRONOS_validation" };
#else
  std::vector<const char*> validationLayers = {};
#endif

  instance = VulkanInstance::Builder()
    .withApiVersion(1, 3)
    .withInstanceExtensions(window.getRequiredVulkanInstanceExtensions())
    .withValidationLayers(validationLayers)
#ifdef _DEBUG
    .withDebugUtilities()
#endif
    .build();

  surface = VulkanSurface::Builder()
    .withInstance(instance.get())
    .withWindow(&window)
    .build();

  VulkanQueue::Builder vulkanQueueBuilder;
  device = VulkanDevice::Builder(vulkanQueueBuilder)
    .withApiVersionSupport(1, 3)
    .withQueueFamilySupport(VulkanDeviceQueueFamilies::GRAPHICS | VulkanDeviceQueueFamilies::PRESENTATION | VulkanDeviceQueueFamilies::TRANSFER | VulkanDeviceQueueFamilies::COMPUTE)
    .withSurfacePresentationSupport(surface.get())
    .withInstance(instance.get())
    .withValidationLayers(validationLayers)
    .build();

  VulkanImage::Builder vulkanImageBuilder;
  swapchain = VulkanSwapchain::Builder(vulkanImageBuilder)
    .withDevice(device.get())
    .withExtents(window.getWidth(), window.getHeight())
    .withSurface(surface.get())
    .build();

  VulkanFramebuffer::Builder vulkanFramebufferBuilder;
  renderPass = VulkanRenderPass::Builder(vulkanFramebufferBuilder)
    .withDevice(device.get())
    .withSwapchain(swapchain.get())
    .withColourClearValue(0.0f, 0.0f, 0.0f, 1.0f)
    .withDepthClearValue(1.0f)
    .build();

  graphicsCommandPool = VulkanCommandPool::Builder()
    .withDevice(device.get())
    .withFlags(VK_COMMAND_POOL_CREATE_RESET_COMMAND_BUFFER_BIT)
    .withQueueFamilyIndex(device->getGraphicsQueue()->getFamilyIndex())
    .build();

  graphicsCommandBuffers = VulkanCommandBuffer::Builder()
    .withCommandPool(graphicsCommandPool.get())
    .withDevice(device.get())
    .withLevel(VK_COMMAND_BUFFER_LEVEL_PRIMARY)
    .build(renderPass->getFramebuffers().size());

  transferCommandPool = VulkanCommandPool::Builder()
    .withDevice(device.get())
    .withFlags(VK_COMMAND_POOL_CREATE_RESET_COMMAND_BUFFER_BIT)
    .withQueueFamilyIndex(device->getTransferQueue()->getFamilyIndex())
    .build();

  uint32_t const maxFramesInFlight = swapchain->getMaxFramesInFlight();
  imageAvailable.resize(maxFramesInFlight);
  renderFinished.resize(maxFramesInFlight);
  drawFences.resize(maxFramesInFlight);

  VulkanSemaphore::Builder vulkanSemaphoreBuilder;
  vulkanSemaphoreBuilder.withDevice(device.get());
  VulkanFence::Builder vulkanFenceBuilder;
  vulkanFenceBuilder.withDevice(device.get());
  for (size_t i = 0; i < maxFramesInFlight; i++)
  {
      imageAvailable[i] = vulkanSemaphoreBuilder.build();
      renderFinished[i] = vulkanSemaphoreBuilder.build();
      drawFences[i] = vulkanFenceBuilder.build();
  }
```
