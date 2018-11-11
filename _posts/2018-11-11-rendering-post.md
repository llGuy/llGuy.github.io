---
layout: post
title: "New Rendering System"
date: 2018-11-11 22:35:00 +0000
categories: "GameEngine"
---

{% highlight c++ %}

/* initializing materials */
material_light_info light{ glm::vec3(1.0f), glm::vec3(0.7f), glm::vec3(0.5f), 20.0f, 0.2f };
material_prototype monkey_skin{ light, shaders[shader_handle("shader.low_poly")], lights };
monkey_skin.get_textures_2D().push_back(textures.get_texture("texture.player"));
monkey_skin.get_textures_cubemap().push_back(textures.get_texture("texture.sky"));
renderer.set_material_prototype(monkey_skin);

renderer.set_projection(projection_matrix);

/*rendering materials*/
glsl_program * low_poly_shader = renderer.get_shader();
low_poly_shader->bind();
low_poly_shader->send_uniform_vec3("camera_position", glm::value_ptr(camera.get_position()), 1);
low_poly_shader->send_uniform_mat4("view_matrix", glm::value_ptr(view_matrix), 1);

renderer.render();

{% endhighlight %}

#### This was how rendering in my engine was done before... and it makes me gag

## Everything wrong with the old system

- creating a new material prototype was a `huge pain` : materials had to be sorted manually into their respective renderers
- to access a material one must go through a renderer that uses that specific material prototype
- rendering materials is a huge pain
- in order to update the view and projection matrices, the user must manually access the shader of the rendererer(material prototype of the renderer)

{% highlight c++ %}

/* material handler sorts all materials and renderers and stores them in an accessible way */
class material_handler materials;

/* initializing */
auto sky_material = materials.add_material("material.sky"
		, material_light_info() /* no lighting */
		, shaders[shader_handle("shader.sky")]
		, lights
		, &world.get_scene_camera());

/* rendering */
materials.render_all();

{% endhighlight %}

#### This is how rendering in my engine is done now... and it makes me so happy

## Everything great with the new system

- initializing a new material prototype is so so so much easier as the material_handler automatically sorts all the rendering shenanigans
- rendering is... I don't even need to say it as it speaks for itself - it's 1 line (don't worry the rendering is still very customizable with the initializations prior to the game loop)

![photo](/assets/rendering_system.PNG)
