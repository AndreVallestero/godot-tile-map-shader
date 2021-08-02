# godot-tile-map-shader

```
shader_type canvas_item;

uniform float txr_width;
uniform float txr_height;
uniform float tile_width;
uniform float tile_height;
uniform float tile_count;

void fragment() {
	vec2 txr_size = vec2(txr_width, txr_height);
	COLOR = texture(TEXTURE,
		(mod(UV * txr_size, 1) * tile_width // offset
		+ vec2(floor(texture(TEXTURE, UV).r * tile_count) * tile_width., txr_height - tile_height)) // map
		/ txr_size // normalize
	);
}
```

**The problem**

While creating my [Godot tiles benchmark](https://github.com/AndreVallestero/godot-static-tiles-benchmark), I wondered about the feasibility of a shader-based tile renderer. This concept is far from new, and it's often considered as the ideal way to render a large amount of tiles. The reason why no one uses it in Godot is because of Godot's shader system. The traditional way to create a tile map shader is to pass an array of tile IDs for the shader to render, however, Godot [currently lacks this functionality](https://github.com/godotengine/godot-proposals/issues/931). This means that we can't specify which tile IDs to render so it has to be hardcoded in the shader.

**How did you solve it?**

I tackled this problem by storing the tile set and the tile map in the same texture and using a shader texture read to get the index and draw the appropriate tile. Additionally, by attaching an `ImageTexture` to a `Sprite` node, we are able to dynamically change the source texture, effectively mimicking the behavior of a dynamic tile map.

**What does this mean for me?**

When rendering 8000 tiles, there's a 30% performance improvement (2700fps -> 3500fps) relative to a `TileMap` of equivalent size, and this will likely scale better with even more tiles. Godot's `TileMap` groups tiles into batches of draw calls (which is great), but still doesn't do it all in a single draw call. For reference, 8000 tiles invoke 40 draw calls using Godot's `TileMap`. This shader draws every tile in 1 draw call.

**So you did all of this work to make something that already exists?**

Yes. I'm not saying you should use this, you should probably use the `TileMap` already built into Godot, but if you're somehow performance limited by how quickly you can render tiles, at least you have an option (until uniform arrays are supported). Here are the advantages of each approach:

TileMap
- support and documentation
- iteration time
- simplicity

Shader
- performance

![](https://user-images.githubusercontent.com/39736205/127790522-7165176f-43ad-4253-a55a-fc470cadf651.png)

![](https://user-images.githubusercontent.com/39736205/127790494-2756716b-1b87-4375-8ca8-0bfdd442cabd.png)
