Minest - Modding - Lua Mapgen From Scratch
==========================================


0. About this
-------------

This tutorial is licensed under [Creative Commons Attribution-ShareAlike][CC-BY-SA].

The current Minetest version, as of writing this, is 0.4.12.


1. The Basics
-------------

The basics are really simple, you can hook into the generation process of
the world by registering a function with [`minetest.register_on_generated`][wiki-rog].
The registered function will be called everytime a new block of the world
is generated.


    minetest.register_on_generated(function(minp, maxp, block_seed)
        -- Logic goes here
	end)

The parameters are:

 * minp: The starting point, a [vector object][wiki-vector].
 * maxp: The end point, a [vector object][wiki-vector].
 * block_seed: the seed of the block, not of the map.

That's it regarding the basics.


2. Sculpting the world
----------------------

The best way to mass-manipulate the world is by using the [VoxelManip object][wiki-vm],
which is provided by the Minetest API. Using it is not as simple as our basics,
but let's look at the simplest example:

    local manipulator, emin, emax = minetest.get_mapgen_object("voxelmanip")
    local area = VoxelArea:new({
	    MinEdge = emin,
		MaxEdge = emax
    })

This how to get a new [VoxelManip object][wiki-vm], the [VoxelArea object][wiki-va]
is necessary as it will be used ot access the data of the manipulator. Now we
can get the data and actually start manipulating the block:

    local data = manipulator:get_data()
	local index = area:index(x, y, z)
	
	date[index] = node

Note that `x`, `y`, `z` are coordinates within the current block, also `node`
is the ID (acquired with [`minetest.get_content_id("mod:node")`][wiki-gci])
of the node that we want to set.

Saving the data is rather simple:

    manipulator:set_data(data)
	manipulator:write_to_map()

That's about it.


3. Saving is not enough
-----------------------

But there are a few steps missing in my previous example regarding saving
the data: Updating the map.

    manipulator:set_data(data)

    manipulator:set_lighting({
	    day = 1,
		night = 0
    })
	manipulator:calc_lighting()
	manipulator:update_liquids()
	manipulator:write_to_map()
	manipulator:update_map()

As the names of these functions suggest, they are performing various taks that
need to be done before the map is abck in a playable state. The setting of
the lighting options prevents lighting artifacts from appear.


4. Preparing the mapgen for us
------------------------------

But there is some more that I need to tell you about. If we want to create
a mapgen from scratch in Lua, we'll have to change a few basic settings before
the world is generated. Don't worry, we can do this easily with regsitering
another function with [`minetest.register_on_mapgen_init`][wiki-romi].

The registered function will be called when the mapgen is ready to generate
the world, but has yet not started.

    minetest.register_on_mapgen_init(function(mapgen_params)
        -- Manipulate params here
	end)

So, what do we need to tell the mapgen anyway?

 * to not use a predefined generator.
 * to not precalculate light.
 * that water is not an issue.

We can do this with the [`minetest.set_mapgen_params`][wiki-smp] function.

    minetest.register_on_mapgen_init(function(mapgen_params)
        minetest.set_mapgen_params({
            flags = "nolight",
			mgname = "singlenode",
			water_level = -31000
		})
	end)

We are setting the "nolight" flag because the [VoxelManip object][wiki-vm] will
be calculating the lighting anyway, it also prevents that the map is
fully light up when we are generating terrain, which might result in fully
light caves.

We set the name of the predefined mapgen to "singlenode", which is NOOP mapgen.
This is to make sure that we're the only system that is generating the terrain.

We set the water level to -31000, this is to prevent possible lighting
artifacts which might happen.


5. Summed up
------------

That's it, the complete code would look like this:

    minetest.register_on_mapgen_init(function(mapgen_params)
        minetest.set_mapgen_params({
            flags = "nolight",
			mgname = "singlenode",
			water_level = -31000
		})
	end)

    minetest.register_on_generated(function(minp, maxp, block_seed)
        local manipulator, emin, emax = minetest.get_mapgen_object("voxelmanip")
        local area = VoxelArea:new({
	        MinEdge = emin,
		    MaxEdge = emax
        })

		local data = manipulator:get_data()
	    local index = area:index(x, y, z)
	
	    date[index] = node

        manipulator:set_data(data)

        manipulator:set_lighting({
	        day = 1,
	        night = 0
        })
	    manipulator:calc_lighting()
	    manipulator:update_liquids()
	    manipulator:write_to_map()
	    manipulator:update_map()	
	end)


6. The MapManipulator from minetest-australopithecus-utils
----------------------------------------------------------

The [minetest-australopithecus-utils][mau] is a utils collection for Minetest
mods, which also contains the MapManipulator object, which defines an easier
interface for the [VoxelManip object][wiki-vm].

    local manipulator = MapManipulator:new()

	-- Perform operations on the data here, like this:
	manipulator:set_node(x, z, y, node)
	
	manipulator:set_data()

It will automatically perform the necessary steps to save the data and update
the map.


7. Frequently Made Mistakes
---------------------------

 1. If you want to do mass edits on the map always use
    the [VoxelManip object][wiki-vm].
 2. If you're using a [PerlinNoise object][wiki-pn] to assist in generating
    the world, initialize it once with the seed of the map. The `block_seed`
	is really only an additional seed for that specific block. If you
	reinitialize the [PerlinNoise object][wiki-pn] everytime a new, it can not
	give you a tileable noise output.
 3. Time your generation function and see if you can make it faster.
    At the moment generating a block is a "stop it all" function, so you want
	it to be fast.



 [CC-BY-SA]: http://creativecommons.org/licenses/by-sa/4.0/
 [wiki-rog]: http://dev.minetest.net/minetest.register_on_generated
 [wiki-vector]: http://dev.minetest.net/vector
 [wiki-vm]: http://dev.minetest.net/VoxelManip
 [wiki-va]: http://dev.minetest.net/VoxelManip#VoxelArea
 [wiki-gci]: http://dev.minetest.net/minetest.get_content_id
 [wiki-romi]: http://dev.minetest.net/minetest.register_on_mapgen_init
 [wiki-smp]: http://dev.minetest.net/minetest.set_mapgen_params
 [mau]: https://github.com/RobertZenz/minetest-australopithecus-utils
 [wiki-pn]: http://dev.minetest.net/PerlinNoise

