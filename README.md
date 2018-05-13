# Computer Graphics - WebGL Experiments

@author Andres Frank
These are my notes and practice excercises for the course _Computer Graphics_ at _Universidad Nacional del Sur_.


---
### 00 - The most basic triangle


**Resources:**

* [Indigo Code: WebGL Tutorial 01 - Setup and Triangle](https://youtu.be/kB0ZVUrI4Aw)
* [WebGL Fundamentals - Fundamentals](https://webglfundamentals.org/webgl/lessons/webgl-fundamentals.html)
* [Universidad Nacional del Sur's computer graphics course notes](https://github.com/WEBglGraficaDCIC/)


---
### 01 - Interactive colored triangle


---
### 02 - Rotating 2D Triangle


---
### 03 - Rotating 3D Cube

This project starts using the Helper libraries located at the `99 - resources` folder, that's why you may not see all the file sources in the root folder for the project.

**Basic Animation**
> Traditionally, if you wanted to repeatedly execute specific tasks (functions) in JavaScript, you used the `methodsetInterval()`.
However, because this JavaScript method was designed before browsers started to support multiple tabs, it executes regardless of which tab is active. This can lead to performance problems, so a new method, `requestAnimationFrame()`, was recently introduced. The function scheduled using this method is only called when the tab in which it was defined is active.
<img src="https://i.imgur.com/I6bPjX8.jpg" alt="graphic figure from book" title="WebGL Programming Guide - Matsuda & Lea - Page 132" style="width: 650px;"/>
_from book: WebGL Programming Guide - Kouichi Matsuda & Rodger Lea_



**Resources:**

* [Indigo Code: WebGL Tutorial 02 - Rotating 3D Cube](https://youtu.be/3yLL9ADo-ko)
* http://glmatrix.net/

---
### 04 - Star Pyramid

**Using VAO Extension**

The concept here is as follows: Without Vertex Array Objects (VAOs) every time you want to draw geometry you have to set up all of the buffer state before each draw call. This is unfortunate overhead during your render loop that we'd rather avoid.

With VAOs we can define the buffer bindings and pointers for a mesh once during initialization and thereafter go back to that binding again at any time by binding a single object (the VAO). Essentially, when you bind a VAO object it "records" any bindBuffer and vertexAttribPointer calls that you make. Then when you bind the VAO again it resets the buffer and attribute state back to the same state. Note that VAOs don't keep track of things like what shader program was bound, what uniforms are set, or what textures are in use. Nor does it record calls to bufferData. It's only the buffer bindings and the attribute pointers.

WebGL2 supports VAOs natively, in order to use them in WebGL1 we use the extension `OES_vertex_array_object`.

```js
function setup() {
	// -- Vertex Array Object (VAO) Set up
	vaoExtension = gl.getExtension('OES_vertex_array_object'); // Obtain the extension
	pyramidVAO = vaoExtension.createVertexArrayOES(); // create the VAO
	vaoExtension.bindVertexArrayOES(pyramidVAO); // begin setting it up
	
	// Create and configure the usual VBO
	vertBuffer = gl.createBuffer();  
	gl.bindBuffer(gl.ARRAY_BUFFER, vertBuffer);  
	gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(verts), gl.STATIC_DRAW);  
	gl.enableVertexAttribArray(positionAttrib);  
	gl.vertexAttribPointer(positionAttrib, 3, gl.FLOAT, false, 12, 0);  
	
	// Create and configure the usual EBO
	indexBuffer = gl.createBuffer();  
	gl.bindBuffer(gl.ELEMENT_ARRAY_BUFFER, indexBuffer);  
	gl.bufferData(gl.ELEMENT_ARRAY_BUFFER, new Uint16Array(indices), gl.STATIC_DRAW);  
	
	// Finish and clean-up
	// IMPORTANT: Don't null the Array and Element buffer BEFORE the VAO or you will be unbinding them from the VAO
	vaoExtension.bindVertexArrayOES(null); // finished setting up VAO
	gl.bindBuffer(gl.ARRAY_BUFFER, null);
	gl.bindBuffer(gl.ELEMENT_ARRAY_BUFFER, null); 
}

function render() {
	gl.useProgram(shaderProgram);  

	ext.bindVertexArrayOES(vao);  
	 
	gl.drawElements(gl.TRIANGLES, triCount, gl.UNSIGNED_SHORT, 0);  
	 
	ext.bindVertexArrayOES(null);  
}
```

Source: https://blog.tojicode.com/2012/10/oesvertexarrayobject-extension.html


---
### 05 - Loading external OBJ models

**New concepts:**

* Using the OBJParser helper class to load external models.
* Playing with HTML color picker and passing its value to the fragment shader.

**Resources:**

* Free .obj models: 
	* https://free3d.com/3d-models/
	* https://www.blendswap.com/
	* https://sketchfab.com/

* [Alternative parser](https://github.com/vorg/geom-parse-obj)
	- support multiple meshes in one file via g sections in the OBJ file
	- returns only vertex data, so no face normals and face tex coords

---
### 06 - Basic Camera

Very basic idea on implementing a Key Listener to move the camera position in the usual X, Y and Z coordinates.

The movement ends up being very clunky because of the coordinates we are using.
See Spherical Camera next for a little more smooth camera using spherical coordinates (Phi, Theta, Rho).

**Resources:**

* https://webglfundamentals.org/webgl/lessons/webgl-3d-camera.html


---
### 07 - Spherical Camera

First approach at implementing a camera using spherical coordinates (Phi, Theta, Rho).

This code gets further worked on and improved, which allows for a much nicer modularization.
The final product can be found at `99 - Resources/SphericalCamera.js`. I have included all the instructions to use it inside that file. 
It ends up allowing you to add an interactive camera to any project just by implementing a function to make the link.

---
### 08 - Normals

Going forward we will start making use of normal vectors, which were there but were not used. They are crucial in calculating lighting.
Let's make sure our OBJ Parser is correctly pulling the normals and that we can use them, by setting the color to be defined by the normal value.


---
### 09 - Lighting


![](https://upload.wikimedia.org/wikipedia/commons/6/6b/Phong_components_version_4.png "wikipedia page on phong lighting")

**Resources:**

* [Indigo Code: WebGL Tutorial 05 - Phong Lighting](https://www.youtube.com/watch?v=33gn3_khXxw)
* [Shader Frog](https://shaderfrog.com/app/editor): A web editor to test shaders.

---
### 10 - Texture

Loading an image file is unfortunately not trivial.
Modern browsers refuse to load resources that do not provide from the same origin and throw a security error when attempted.
This is a similar issue that we ran into when trying to load the shaders from external files. 
In that case we chose to perform a little hack and define the shader code as a javascript variable, so it can be loaded from a script tag.

Now though, we are dealing with an image, so we are going to use a different hack:
We will re-encode the image into a Base64 format and put that directly into our HTML. 
This avoids having to load an actual file from the computer, so the browser won't complain.

In order to do this, just google any base64 encoder that can take files and encode your image into base64.
You can also do this in the terminal with openssl:
`$ openssl base64 -in texture.jpg -out texture.base64`
If you want to have it in a single line so it doesn't occupy a thousand lines in your html file you can use instead:
`$ cat texture.jpg | openssl base64 | tr -d '\n' > texture.singleline.base64`

Once you have the base64 encoded image, define an <img\> tag in the html with the folloning format:
```html
<img id="mytexture" src="data:image/jpg;base64, <encoded image string>">
```

Then we can obtain that image by just doing:
`var mytextureimg = document.getElementById("mytexture");`

At the moment of writing (2018-05-11), firefox does allows for an image to be simply grabbed. Chrome does not.
If you want to see how that would look like, it's as easy as this:
```js
var mytextureimg = new Image();
mytextureimg.crossOrigin = "anonymous";
mytextureimg.src = "texture1_256.jpg";
```
You don't event need to add anything to the HTML.
We want our app to work on any browser, so we will use the base64 method.

Yes, all these hacks are annoying but they allow us to execute the app without having to run a webserver.
Webservers also introduce some other annoyances, like the browser caching the page and not updating with your changes, so it's a compromise.

**Resources:**

* [Indigo Code:WebGL Tutorial 03 - Textured Cube](https://www.youtube.com/watch?v=hpnd11doMgc)
* https://open.gl/textures
* https://stackoverflow.com/questions/5532595/how-do-opengl-texture-coordinates-work

---
### x - 


---
### x - 



