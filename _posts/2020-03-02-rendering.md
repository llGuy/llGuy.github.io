---
layout: post
title: "How I am handling rendering"
date: 2020-03-10 09:40:00
categories: "Game"
---

Rendering is quite a complex topic, and I'm pretty sure that everyone will give a different answer as to how they handle it in their engine. In the underlying engine of this game, I use am using quite a low level graphics API (Vulkan) which already will give the programmer quite a lot of control over exactly what will be going on over at the GPU. Therefore, I didn't really add a whole lot of abstraction over the API, to be able to conserve that level of control.

However, one thing that I wanted to make sure, was that outside of the renderer module, as little Vulkan code would be needed to make the game work (inside the module however, Vulkan code will be very present, even in parts that don't have to do with the API, like atmosphere, lighting, etc...).

One thing that I thought would be worth sharing is how meshes and vertex bindings / attributes are handled.

{% highlight c++ %}
struct mesh_t {
    mesh_buffer_t buffers[MAX_MESH_BUFFERS];
    uint32_t buffer_count;
    buffer_type_t buffer_type_stack[BT_INVALID_BUFFER_TYPE];

    // This is what will get passed to the vkCmdBindVertexBuffers
    uint32_t vertex_buffer_count;
    VkBuffer vertex_buffers_final[BT_INVALID_BUFFER_TYPE];
    VkDeviceSize vertex_buffers_offsets[BT_INVALID_BUFFER_TYPE];
    VkBuffer index_buffer;

    // Data needed to render
    uint32_t vertex_offset;
    uint32_t vertex_count;
    uint32_t first_index;
    uint32_t index_count;
    uint32_t index_offset;
    VkIndexType index_type;
};
{% endhighlight %}

Right here, is the mesh structure. One thing that I really wanted to have, was the ability for the programmer to easily control stuff like not having an index buffer, so that vkCmdDraw could be used for more trivially rasterised geometry (which the game will definitely have).

In this structure, there is an array of mesh buffers which simply contain the vulkan buffer, and what type of buffer it is.

{% highlight c++ %}
struct mesh_buffer_t {
    gpu_buffer_t gpu_buffer;
    buffer_type_t type;
};
{% endhighlight %}

And this buffer type, basically dictates what usage the buffer will serve for rendering the mesh.

{% highlight c++ %}
enum buffer_type_t : char {
    BT_INDICES,
    BT_VERTEX,
    BT_NORMAL,
    BT_UVS,
    BT_COLOR,
    BT_JOINT_WEIGHT,
    BT_JOINT_INDICES,
    BT_EXTRA_V3,
    BT_EXTRA_V2,
    BT_EXTRA_V1,
    BT_INVALID_BUFFER_TYPE
};
{% endhighlight %}

The way the programmer would add a buffer to this structure (say a `BT_VERTEX` type, which will be simply for the vertex positions), they would push onto the buffer type stack, the type of the buffer they want (`BT_VERTEX`), which will increment the `buffer_count` variable. The actual buffer object goes to the array `buffers[]` at index `BT_VERTEX`, like this: `buffers[BT_VERTEX] = mesh_buffer_t{...}`.

For this operation, there is a very simple function: 

{% highlight c++ %}
void push_buffer_to_mesh(
    buffer_type_t buffer_type,
    mesh_t *mesh);
{% endhighlight %}

This function, however, simply adds space for some mesh_buffer that will be usable. To actually fill the `mesh_buffer_t` object with an actual GPU buffer containing data, the programmer will have to access this `mesh_buffer_t`. To do that, they would have to request a pointer to the buffer at index `buffer_type_t` with this function: 

{% highlight c++ %}
mesh_buffer_t *get_mesh_buffer(
    buffer_type_t buffer_type,
    mesh_t *mesh);
{% endhighlight %}

Which will return `&mesh->buffers[buffer_type]` if the `buffer_type` has been pushed to the buffer type stack.

Now, here comes the part that's cool: from this `mesh_t` structure, we can create all the necessary binding and attribute information. 

{% highlight c++ %}
struct shader_binding_info_t {
    uint32_t binding_count;
    VkVertexInputBindingDescription *binding_descriptions;

    uint32_t attribute_count;
    VkVertexInputAttributeDescription *attribute_descriptions;
};

shader_binding_info_t create_mesh_binding_info(
    mesh_t *mesh);
{% endhighlight %}

In this function, depending on the order in which you allocated your buffer types in the stack of `buffer_type_t`, the binding / attribute information will be created in that order.

This binding info structure will then get passed to the function which creates a graphics pipeline for 3D rendering.

When it comes to submitting the mesh:

{% highlight c++ %}
void submit_mesh(
    VkCommandBuffer command_buffer,
    mesh_t *mesh,
    shader_t *shader,
    mesh_render_data_t *render_data);
{% endhighlight %}

`mesh_render_data_t` contains push constant information. In this function, depending on whether the mesh contains a `BT_INDICES` buffer, it will use `vkCmdDraw` or `vkCmdDrawIndexed`. 

And that's it!

Here's some code to create a sphere and submit it.

{% highlight c++ %}
mesh_t sphere = {};
shader_binding_info_t sphere_info = {};
   
load_mesh_internal(
    IM_SPHERE,
    &sphere,
    &sphere_info);
    
const char *paths[] = { "../shaders/SPV/mesh.vert.spv", "../shaders/SPV/mesh.frag.spv" };
shader_t sphere_shader = create_mesh_shader(
    &sphere_info,
    paths,
    VK_SHADER_STAGE_VERTEX_BIT | VK_SHADER_STAGE_FRAGMENT_BIT);

mesh_render_data_t render_data = {};
render_data.model = matrix4_t(1.0f);
render_data.color = vector4_t(0.5f, 0.0f ,0.0f, 1.0f);
render_data.pbr_info.x = 0.2f;
render_data.pbr_info.y = 0.8;

// Some command buffer that was created when frame began rendering.
submit_mesh(command_buffer, &sphere, &sphere_shader, &render_data);
{% endhighlight %}

The two most important things that this system is currently missing is a way to render instanced geometry, and a way to merge the buffers into one big buffer. Once that gets added if it is needed, I will definitely write another post about it.
