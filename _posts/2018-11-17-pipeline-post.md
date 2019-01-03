---
layout: post
title: "Rendering Pipeline"
date: 2019-11-17 16:00:00
categories: "GameEngine"
---

A good and flexible rendering pipeline is essential for me as I often like to tweak it. I tried to make one that was relatively decoupled so that if I move the stages around it still work. The solution I had (not a very complicated one) was to simply have a stack of polymorphic render stages.

{% highligh c++ %}
class render_stage;
{% endhighligh %}

From this 2 class derive : `render_stage2D` and `render_stage3D`.
- The 2D render stage is for post processing effects as they simply need a 2D textured quad rendered of the scene.
- The 3D render stage is for stages that require the entire scene to be rendered again. For example the shadow mapping stage which would be at the beginning of the pipeline.

Creating a stage requires a `struct render_stage_create_info` structure passed into the init function.
It needs the dimensions of the texture target and the target textures or renderbuffers.

This is how the shadow mapping stage gets created:

{% highligh c++ %}
render_stage_create_info info;
info.width = shadow_handler::get_shadow_map_size();
info.height = shadow_handler::get_shadow_map_size();
info.color_name = "";
info.depth_name = "texture.shadow_map";
info.create_flags = RENDER_STAGE_CREATE_INFO_COLOR_NONE
	| RENDER_STAGE_CREATE_INFO_DEPTH_TEXTURE;

render_pipeline.add_render_stage<render_stage3D>("render_stage.shadow_pass", &materials, &lights.get_shadow_handler().get_shadow_camera(), time_handler);
render_pipeline.create_render_stage("render_stage.shadow_pass", info, renderbuffers, textures);
{% endhighligh %}

For a post processing stage it looks like this:

{% highligh c++ %}
render_stage_create_info info;

auto depth_dof = renderbuffers.add_renderbuffer("renderbuffer.bloom_depth");
create_depth_renderbuffer(*depth_dof, display_w, display_h);

info.width = display_w;
info.height = display_h;
info.color_name = "texture.bloom";
info.depth_name = "renderbuffer.bloom_depth";
info.create_flags = RENDER_STAGE_CREATE_INFO_COLOR_TEXTURE
	| RENDER_STAGE_CREATE_INFO_DEPTH_RENDERBUFFER;

auto bloom_stage = render_pipeline.add_render_stage<render_stage2D>("render_stage.bloom", shaders[shader_handle("shader.bloom")], &guis);
render_pipeline.create_render_stage("render_stage.bloom", info, renderbuffers, textures);

bloom_stage->add_texture2D_bind(textures.get_texture("texture.motion_blur")
	, textures.get_texture("texture.hblur1"));
bloom_stage->set_active_textures(active_texture_uniform_pair{ "diffuse", 0 }
	, active_texture_uniform_pair{ "brightness", 1 });
{% endhighligh %}
The 2D stage most likely needs dependencies of the previous stages.
This one needs `"texture.motion_blur"` and `"texture.hblur1"`. One could simply specify to use the previous stage's output as the input by passing `TEXTURE2D_BINDING_PREVIOUS_OUTPUT` into the `add_texture2D_bind` function.