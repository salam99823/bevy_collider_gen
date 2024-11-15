# bevy_collider_gen

[![Crates.io](https://img.shields.io/crates/v/bevy_collider_gen.svg)](https://crates.io/crates/bevy_collider_gen)
[![Crates.io](https://img.shields.io/crates/d/bevy_collider_gen.svg)](https://crates.io/crates/bevy_collider_gen)
[![MIT/Apache 2.0](https://img.shields.io/badge/license-MIT%2FApache-blue.svg)](https://github.com/shnewto/bevy_collider_gen#license)

a library for generating 2d colliders, for bevy apps, from images with transparency

## specifying your dependency

by default, both bevy_rapier2d and avian2d (formerly bevy_xpbd_2d) are enabled. this is to help with the out of box experience, specifically, being able to run both examples and tinker.

but you'll probably only want to just use one of the physics engines supported so when you use it in your own crate fill in in the `bevy_collider_gen` dependencies with something like this for `bevy_rapier2d`

```toml
[dependencies.bevy_collider_gen]
# replace "*" with the most recent version of bevy_collider_gen
version = "*"
```

or this for `avian2d`

```toml
[dependencies.bevy_collider_gen]
# replace "*" with the most recent version of bevy_collider_gen
version = "*"
features = ["avian2d", "parallel"]
default-features = false
```

## example

![example with a car, terrain, and boulders](https://github.com/shnewto/bevy_collider_gen/blob/main/img/example-default.png?raw=true)

to see this in action you can run the example, with no args it generates a scene with various colliders using pngs in the `assets/sprite` directory

### bevy_rapier2d

#### note that you must have the rapier2d feature enabled

```sh
cargo run --example rapier2d_colliders -F "bevy_rapier2d/debug-render-2d"
```

### avian2d

#### note that you must have the avian2d feature enabled

```sh
cargo run --example avian2d_colliders -F "avian2d, avian2d/debug-plugin"
```

you can also specify a path to an image yourself the example will attempt to generate one or more convex_polyline colliders for the objects it finds

## about / why

i was looking for a way to iterate on some 2d scenes with colliders on things with more sophisticated shapes than simple
geometry, i figured there should be enough info in an image with transparency to generate colliders, and... there is! so i
packaged up my approach here in case anyone else could benefit.

## how it works

😄 head on over to the edges crate to learn more <https://github.com/shnewto/edges>

## caveats

- as mentioned here and there in these docs, this implementation requires images to have transparency in order to distinguish object from non-object :)
- i imagine for generating things at a larger scale, i.e. colliders for sets of sprites bigger than pixel counts in the hundreds, this implementation won't be performant to do at runtime. i'll suggest serializing the colliders you like and deserializing in your app instead of doing all the number crunching on load when you need a performance boost

## examples of colliders generated for assets/sprite/car.png

(as in pictures of the sort of thing you can expect, not the runnable bevy app example. that's a couple headings up)

### convex polyline (bevy_raiper2d only)

![convex polyline collider on an upside down car sprite](https://github.com/shnewto/bevy_collider_gen/blob/main/img/convex-polyline.png?raw=true)

### polyline

![polyline collider on an upside down car sprite](https://github.com/shnewto/bevy_collider_gen/blob/main/img/polyline.png?raw=true)

### convex hull

![convex hull collider on an upside down car sprite](https://github.com/shnewto/bevy_collider_gen/blob/main/img/convex-hull.png?raw=true)

### heightfield

the current implementation does best if the image you're generating a heightfield from is either centered in the image
or spans the entire width of the image...

![heightfield collider on an upside down car sprite](https://github.com/shnewto/bevy_collider_gen/blob/main/img/heightfield.png?raw=true)

### convex decomposition

I didn't add support for convex decomposition directly because when sprites were small, and collisions were forceful, they were sort of unreliable (occasional panics because of bounds indexing in rapier's dependencies 💀). But if you wanted to use
convex decomposition colliders you could construct them with the edge coordinates from your image with something like this

```rust
let sprite_image = image_assets.get(sprite_handle.unwrap()).unwrap();
let edges = Edges::from(sprite_image)
let edge_coordinate_groups = edges.multi_image_edge_translated();
for coords in edge_coordinate_groups {
    let indices: Vec<[u32; 2]> = (0..coords.len()).map(|i| [i as u32, i as u32]).collect();
    let collider = Collider::convex_decomposition(&coords, &indices);
    commands.spawn((
        collider,
        RigidBody::Fixed,
        SpriteBundle {
            texture: sprite_handle.unwrap().clone(),
            ..default()
        },
    ));
}
```

![convex decomposition collider on a car sprite](https://github.com/shnewto/bevy_collider_gen/blob/main/img/convex-decomposition.png?raw=true)

## license

all code in this repository is dual-licensed under either:

- MIT License (LICENSE-MIT or <http://opensource.org/licenses/MIT>)
- Apache License, Version 2.0 (LICENSE-APACHE or <http://www.apache.org/licenses/LICENSE-2.0>)

at your option.
