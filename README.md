# TextureMerger
TextureMerger is a lightweight client-side vanilla-Javascript library that generates a [texture atlas](https://en.wikipedia.org/wiki/Texture_atlas) from provided Three.js textures and calculates the UV coordinates for each texture inside the atlas.

Ported from [TextureMerger class](https://github.com/oguzeroglu/ROYGBIV/blob/master/js/handler/TextureMerger.js) of [ROYGBIV Engine](https://github.com/oguzeroglu/ROYGBIV).

## Usage

### Include the library in your HTML

```HTML
<script src="path_to_TextureMerger.js"></script>
```

### Merge your Textures
```Javascript
var texture1, texture2, texture3; // Both instances of THREE.Texture
loadTextures(); // Your texture loading logic

var textureMerger = new TextureMerger({
	texture1: texture1,
	texture2: texture2,
	texture3: texture3
});

var atlas = textureMerger.mergedTexture;
var ranges = textureMerger.ranges;
// ranges:
// {
//		texture1: {startU: 0, endU: 0.25, startV: 1, endV: 0.6}
// 		texture2: { startU: .... endU: .... startV: ..., endV: ... }
//      texture3: { startU: .... endU: .... startV: ..., endV: ... }
// }
```
### In your shader

Pass textureMerger.mergedTexture to your shader as a Texture uniform. Pass the range of related texture as an attribute or uniform (attribute makes more sense as it's not dynamic):

For **gl.POINTS**
```GLSL
float coordX = ((gl_PointCoord.x) * (endU - startU)) + startU;
float coordY = ((1.0 - gl_PointCoord.y) * (endV - startV)) + startV;
vec4 textureColor = texture2D(texture, vec2(coordX, coordY));
```
For the rest:
```GLSL
// affine transformation on original UV of a vertex
float coordX = (uv.x * (endU - startU) + startU);
float coordY = (uv.y * (startV - endV) + endV);
vec4 textureColor = texture2D(texture, vec2(coordX, coordY));
```

## Prevent Texture Bleeding
Texture Bleeding is a common problem for visual applications that rely on Texture Atlases. In order to overcome this problem you can use [Half-texel edge correction method](http://drilian.com/2008/11/25/understanding-half-pixel-and-half-texel-offsets/):

In your vertex shader:

```GLSL
// Instead of the original uvCoordinates, pass the return value
// of this function to your Fragment Shader.
// uvCoordinates: [startU, startV, endU, endV]
vec4 fixTextureBleeding(vec4 uvCoordinates){
	// TEXTURE_SIZE is the size of each Atlas entry.
	// For instance if you created a Texture Atlas from 128x128
	// textures, TEXTURE_SIZE would be 128.
	float offset = 0.5 / float(TEXTURE_SIZE);
	return vec4(uvCoordinates[0] + offset, uvCoordinates[1] - offset, uvCoordinates[2] - offset, uvCoordinates[3] + offset);
}
```

## License
TextureMerger uses MIT license.