# Project in DH2323 VT24
<img width="460" alt="Skärmavbild 2024-05-17 kl  00 32 15" src="https://github.com/stan4dbunny/Shell-Texturing/assets/107579396/7958502d-18b8-4064-a517-531e2ffb2199">
<img width="458" alt="BlinnPhongBunnyShade" src="https://github.com/stan4dbunny/Shell-Texturing/assets/107579396/83601e21-b5c4-4637-86c1-8f64311ff8c5">


# 1 Introduction
In this project we create a Unity project that implements basic
shell texturing to simulate fur using a custom made shader. We
extend the technique by implementing simple movement, con-
trol of the degree of fur on different parts of the model, as well
was Blinn-Phong shading.

Shell texturing is commonly in the gaming industry, be-
cause of its good performance and the simplicity of the tech-
nique. It can be used to easily make luscious fields of grass,
as well as animals covered with million strands of fur. The
technique has been used in games like Dark Souls and Genshin
Impact.

# 2 Implementation

# Random Noise
As a first step, a field of grass was implemented. To create the random noise required for this field, a script called ShellTexturing.cs was created in Unity. The script uses Unity’s random number generator to generate a float between 0 and 1 for each pixel in the texture. This number is then compared to some kind of boundary decided in the editor, which decides whether the texel should be filled in or not.

![fig_1](https://github.com/stan4dbunny/Shell-Texturing/assets/107579396/0df42e40-0b42-4276-af31-399157577b9e)
Fig. Randomly generated noise. 

# Layers

To create the illusion of height many of these layers should
be stacked on top of one another. For performance rea-
sons, this was done on the GPU. To accomplish this, Unity
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

![fig_3](https://github.com/stan4dbunny/Shell-Texturing/assets/107579396/240e8fd3-32f6-454a-9153-659747dee7c0)
Fig. Different layers.

The black pixels were then discarded, giving the result in the figure below. As can be seen in the pictures, the distance between layers is clearly noticeable. This distance can be tweaked, but with this technique the distance will always be there. This is a limitation in the shell texturing technique that can be improved upon, and will be discussed later.

![fig_4](https://github.com/stan4dbunny/Shell-Texturing/assets/107579396/cf7916ce-ccee-4c0f-a5bd-b6e87420f016)
Fig. Black pixels discarded

# Basic lighting 
To make the use case more realistic, and to be able to see more details, we added some simple lighting and shading. The colour of each pixel is multiplied with the height of the grass to make the grass darker at its base and lighter at the top. The lighting will be improved upon later.
![fig_6](https://github.com/stan4dbunny/Shell-Texturing/assets/107579396/804c8e88-4e02-408c-83dc-2b9838b40998)

# Cylindrical shape
Hair strands are not shaped like rectangles, but rather rounded and cone shaped; bigger at the bottom and smaller at the top. This is the next thing that was modelled. The distance to the centre of the grass blade is calculated for each pixel, and if that distance is bigger than a certain predetermined thickness the pixel is discarded. In order to calculate the distance to the centre, some steps needed to be taken. First, the fractal part of the UV coordinate is multiplied with the resolution. This gives us the result seen in the figure below.
![fig_7](https://github.com/stan4dbunny/Shell-Texturing/assets/107579396/5da7fcc6-3835-48fc-9dc5-21df871c0e64)

Then, each UV coordinate was moved so that the origin is in the middle of the texel, instead of in the lower left corner. The result is in the figure below.

![fig_8](https://github.com/stan4dbunny/Shell-Texturing/assets/107579396/4bd68500-a9c5-401b-97fc-09bf1ead7944)

From this, the distance from the origin, the middle of the grass blade, to the pixel can be calculated. The result of this is in the figure below.

![fig_9](https://github.com/stan4dbunny/Shell-Texturing/assets/107579396/f5be9138-78e3-495c-a69c-737e846b57fe)

# Cone shape

To create the cone shape, more pixels needed to be checked and discarded. If the distance to the centre of the blade is bigger than the thickness, multiplied with the randomly generated number subtracted by the height of the previous layer, the pixel should be discarded. As the layers stack up, the variable representing the height of the previous layer will become bigger, leading to the total thickness becoming smaller and smaller, and the blade becoming thinner and thinner. 
![fig_10](https://github.com/stan4dbunny/Shell-Texturing/assets/107579396/fbd6b57e-d57b-485d-b3cc-de807511395f)
![fig_11](https://github.com/stan4dbunny/Shell-Texturing/assets/107579396/eba463de-adca-42aa-900a-a48be4824bed)


<img width="460" alt="fig_20" src="https://github.com/GamerErre/Shell-Texturing/assets/107579396/d9fa7b91-4af0-42f0-abbf-9537c512c2f3">

<img width="460" alt="fig_19" src="https://github.com/GamerErre/Shell-Texturing/assets/107579396/2cb641ba-5ad0-4135-af89-03a3b0ea067b">

