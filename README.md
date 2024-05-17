# Project in DH2323 VT24
<img width="460" height = "400" alt="Skärmavbild 2024-05-17 kl  00 32 15" src="https://github.com/stan4dbunny/Shell-Texturing/assets/107579396/7958502d-18b8-4064-a517-531e2ffb2199">
<img width="460" height = "400 alt="BlinnPhongBunnyShade" src="https://github.com/stan4dbunny/Shell-Texturing/assets/107579396/83601e21-b5c4-4637-86c1-8f64311ff8c5">

<iframe frameborder="0" src="https://itch.io/embed/2713580" width="552" height="167"><a href="https://stan4dbunny.itch.io/shell-texturing">Shell texturing for fur simulation by stan4dbunny</a></iframe>

<div style="text-align: center;"><iframe id="ytplayer" type="text/html" width="640" height="360" src="https://www.youtube.com/embed/fNlnBNDZTOo&origin=https://stan4dbunny.github.io" frameborder="0"></iframe></div>

https://youtu.be/fNlnBNDZTOo?si=uimHbqeHfUmmQtk9

# 1 Introduction
In this project we create a Unity project that implements basic shell texturing to simulate fur using a custom made shader. We extend the technique by implementing simple movement, control of the degree of fur on different parts of the model, as well was Blinn-Phong shading.

Shell texturing is commonly in the gaming industry, because of its good performance and the simplicity of the technique. It can be used to easily make luscious fields of grass, as well as animals covered with million strands of fur. The technique has been used in games like Dark Souls and Genshin Impact.

# 2 Implementation

# Random Noise
As a first step, a field of grass was implemented. To create the random noise required for this field, a script called ShellTexturing.cs was created in Unity. The script uses Unity’s random number generator to generate a float between 0 and 1 for each pixel in the texture. This number is then compared to some kind of boundary decided in the editor, which decides whether the texel should be filled in or not.

<img width="460" height = "400" src="https://github.com/stan4dbunny/Shell-Texturing/assets/107579396/0df42e40-0b42-4276-af31-399157577b9e">

Fig. Randomly generated noise. 

# Layers

To create the illusion of height many of these layers should be stacked on top of one another. For performance reasons, this was done on the GPU. To accomplish this, Unity
Shaders were used. The shader code can be found in
FurShader.shader. When the layers were introduced,
it was instead needed to send the random value to the
shader, where the comparison takes place for each pixel. In
ShellTexturing.cs a loop in the the update function was
used to traverse the amount of layers. The spacing between
the layers was calculated from the height of the grass and the
determined amount of layers.

To optimise the layering, Unity’s Graphics.RenderMesh
was used to render a mesh at the right level, instead of creating
a new game object for each layer. In the shader, the number
generated is compared to the height of the previous layer. This
is because not all grass blades will have the same height, com-
paring it to the previous height will only allow some pixels to
continue their grass growth. The result of this can be seen in the figure below.

<img width="460" height = "400" src="https://github.com/stan4dbunny/Shell-Texturing/assets/107579396/240e8fd3-32f6-454a-9153-659747dee7c0">
Fig. Different layers.

The black pixels were then discarded, giving the result in the figure below. As can be seen in the pictures, the distance between layers is clearly noticeable. This distance can be tweaked, but with this technique the distance will always be there. This is a limitation in the shell texturing technique that can be improved upon, and will be discussed later.

<img width="460" height = "400" src="https://github.com/stan4dbunny/Shell-Texturing/assets/107579396/cf7916ce-ccee-4c0f-a5bd-b6e87420f016">
Fig. Black pixels discarded

# Basic lighting 
To make the use case more realistic, and to be able to see more details, we added some simple lighting and shading. The colour of each pixel is multiplied with the height of the grass to make the grass darker at its base and lighter at the top. The lighting will be improved upon later.
<img width="460" height = "400" src="https://github.com/stan4dbunny/Shell-Texturing/assets/107579396/804c8e88-4e02-408c-83dc-2b9838b40998">

# Cylindrical shape
Hair strands are not shaped like rectangles, but rather rounded and cone shaped; bigger at the bottom and smaller at the top. This is the next thing that was modelled. The distance to the centre of the grass blade is calculated for each pixel, and if that distance is bigger than a certain predetermined thickness the pixel is discarded. In order to calculate the distance to the centre, some steps needed to be taken. First, the fractal part of the UV coordinate is multiplied with the resolution. This gives us the result seen in the figure below.
<img width="460" height = "400" src="https://github.com/stan4dbunny/Shell-Texturing/assets/107579396/5da7fcc6-3835-48fc-9dc5-21df871c0e64">

Then, each UV coordinate was moved so that the origin is in the middle of the texel, instead of in the lower left corner. The result is in the figure below.

<img width="460" height = "400" src="https://github.com/stan4dbunny/Shell-Texturing/assets/107579396/4bd68500-a9c5-401b-97fc-09bf1ead7944">

From this, the distance from the origin, the middle of the grass blade, to the pixel can be calculated. The result of this is in the figure below.

<img width="460" height = "400" src="https://github.com/stan4dbunny/Shell-Texturing/assets/107579396/f5be9138-78e3-495c-a69c-737e846b57fe">

# Cone shape

To create the cone shape, more pixels needed to be checked and discarded. If the distance to the centre of the blade is bigger than the thickness, multiplied with the randomly generated number subtracted by the height of the previous layer, the pixel should be discarded. As the layers stack up, the variable representing the height of the previous layer will become bigger, leading to the total thickness becoming smaller and smaller, and the blade becoming thinner and thinner. 
<img width="460" height = "400" src="https://github.com/stan4dbunny/Shell-Texturing/assets/107579396/fbd6b57e-d57b-485d-b3cc-de807511395f">
<img width="460" height = "400" src="https://github.com/stan4dbunny/Shell-Texturing/assets/107579396/eba463de-adca-42aa-900a-a48be4824bed">

# Shape of the model
Up until this point, the implementation only supported a plane, but to extend it to any shape, the code and thinking had to be altered. For a sphere, one could imagine scaling the mesh for each layer, but this wouldn’t work for all types of shapes. To decide the location for the next layer for an arbitrary shape, the normals of the vertices need to be taken into account. In the case of the plane/quad, the position of the layers was determined by the number of the current layer multiplied with the determined height of the grass, divided by the amount of layers. For an arbitrary shape, the same distance will be moved along the multiplied normal’s direction, for each vertex.
<img width="460" height = "400" src="https://github.com/stan4dbunny/Shell-Texturing/assets/107579396/3cc3f880-e5cf-4922-8986-4c4f65ff548e">
<img width="460" height = "400" src="https://github.com/stan4dbunny/Shell-Texturing/assets/107579396/dd8fafae-6793-4828-a98d-d6473cfd0836">

# Colour of the fur
To make the implementation more usable, it was extended to make the fur the colour of the texture.  

# Fur placement
As was seen in the fluffy bunny above, the bunny has fur everywhere, but realistically it shouldn’t have fur in its eyes and on the inside of the ears. Also, it would be beneficial to be able to have various fur length on different places of the model. To solve this, the model was drawn on in a grayscale brush using TexturePaint in Blender. In the map, black is fur, white is no fur, and grey is some fur. Then, in the shader for each vertex, the greyscale texture is sampled and the value at that vertex is used to scale the height of the new layer. Per pixel, it's checked if the height is smaller than a predetermined, short height, and if it is so, the colour at that pixel is set to that of the model. 

<img width="460" alt="fig_19" src="https://github.com/GamerErre/Shell-Texturing/assets/107579396/2cb641ba-5ad0-4135-af89-03a3b0ea067b">
<img width="460" alt="fig_20" src="https://github.com/GamerErre/Shell-Texturing/assets/107579396/d9fa7b91-4af0-42f0-abbf-9537c512c2f3">
Fig. Various fur length. Shorter on ears, longer on body.

The figure also illustrates an artefact that shows up with the shell texturing technique. 

<img width="460" alt="fig_21 (1)" src="https://github.com/stan4dbunny/Shell-Texturing/assets/107579396/866e828f-4b59-4766-924f-5fc0c3fe0f44">
Fig. It can be seen that at the edge of the model, the layers become quite visible. The possible solutions to this will be discussed later. 

# Movement 
Realistically, the fur should move when the object is moved. The general idea is that the fur should move in the opposite direction of the movement. To do this, and to make the force affect the top of the strands more than the base, each vertex is displaced with a determined displacement strength, multiplied by the current layer height divided by the amount of total layers, and then this is multiplied with the negated velocity. This predicted displacement can however displace strands into the model, which is undesirable. 

<img width="460" alt="MovingLeftArtefact" src="https://github.com/stan4dbunny/Shell-Texturing/assets/107579396/0f9c30d9-6bb8-43df-af37-6c4ba39bd39a">
<img width="460" alt="MovingRightArtefact" src="https://github.com/stan4dbunny/Shell-Texturing/assets/107579396/e0698052-3837-4050-b865-c9e644011833">

When a strand is about to be displaced into the model, it means that the displacement vector and the normal of the surface are pointing in opposite directions.

<img width="460" alt="DisplacementDiagram" src="https://github.com/stan4dbunny/Shell-Texturing/assets/107579396/600e7e0e-0f25-4ea5-b873-1f11749d5ee6">

# Blinn-Phong
Since hair is specular, we want to take that into account. A good model for this is the Blinn-Phong model. While it is not physically accurate, it can provide good looking results. The diffuse, ambient, and specular part are calculated and then added together. The specular part is based mainly on the half angle between the view angle direction and the light direction. This angle is then raised to the specular power, and the product of this is multiplied with the specular strength. The specular power determines the focus of the specularity, and the strength determines the brightness. This factor is multiplied with the component wise multiplication of the colour of the light source and the specular colour. The diffuse part of the colour is determined by the colour of the surface multiplied component wise with the colour of the light. Then, the result of the component wise multiplication is multiplied with the dot product between the normal and direction of the light. The ambient part of the colour is calculated from Unit's ShadeSH9 function. This will make the colour depend on the surrounding environment. Additionally to the Blinn-Phong model, the colour is still multiplied at the end with the height of the previous layer, to make the hairs darker at the base and lighter at the top. 

<img width="460" height = "400" alt="Skärmavbild 2024-05-17 kl  00 32 15" src="https://github.com/stan4dbunny/Shell-Texturing/assets/107579396/7958502d-18b8-4064-a517-531e2ffb2199">
<img width="460" height = "400" alt="BlinnPhongBunnyShade" src="https://github.com/stan4dbunny/Shell-Texturing/assets/107579396/83601e21-b5c4-4637-86c1-8f64311ff8c5">

# Drooping
When the object is not in motion, the hairs should still be
drooping down slightly from gravity. This was implemented
through displacing each vertex slightly downwards.

<img width="460" alt="Skärmavbild 2024-05-17 kl  12 16 52" src="https://github.com/stan4dbunny/Shell-Texturing/assets/107579396/3c05fdfe-16b2-4c51-9a5e-525ab09c3d31">


