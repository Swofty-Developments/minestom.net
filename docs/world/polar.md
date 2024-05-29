---
description: >-
  This page describes how to use the Polar world API for Minestom
---

## What is Polar?

Polar is a lightweight, fast world format for Minestom. It allows for the saving and parsing of worlds into bytes and can be found [on GitHub here](https://github.com/hollow-cube/polar).

## Getting a PolarWorld

Polar Worlds can be instanced in a couple ways, with the most comming being through converting from the [Anvil Format](/docs/world/anvilloader) into a `PolarWorld`, or reading a `byte[]` representing the world, often from a database.


```java
// The ChunkSelector is optional, specific Chunk radius' can be loaded through ChunkSelector#radius
PolarWorld anvilPolarWorld = AnvilPolar.anvilToPolar(Path.of("/path/to/anvil/world/dir"), ChunkSelector.all());

// worldBinary can be fetched from some form of persistent data storage, such as a database
PolarWorld bytesPolarWorld = PolarReader.read(worldBinary)
``` 

## Loading and Saving a PolarWorld

Polar Worlds after being instanced can be loaded as a ChunkLoader through the use of `PolarLoader`, the code looks as follows;

```java
PolarWorld world = ...

// Getting a SharedInstance
InstanceManager manager = MinecraftServer.getInstanceManager();
InstanceContainer instance = manager.createInstanceContainer();
SharedInstance sharedInstance = manager.createSharedInstance(instance);

// Setting the ChunkLoader
instance.setChunkLoader(new PolarLoader(polarWorld));
```

Polar Worlds can then be written back to `byte[]` through the `PolarWriter` class. This would then need to be saved persistently manually, an example of such is as follows;

```java
PolarWorld world = ...
byte[] polarWorldAsBytes = PolarWriter.write(world);
```

## User data & callbacks
By default, Polar only stores blocks, biomes, block entities, and light data. However, in many cases it is desirable to have some additional user specific data stored in the world. To accommodate this use case, Polar chunks each have a "user data" field, which can contain any arbitrary data. To work with it, you must implement PolarWorldAccess, for example, the following will write the time of save in each chunks user data:

```java
public class UpdateTimeWorldAccess implements PolarWorldAccess {
    private static final Logger logger = LoggerFactory.getLogger(UpdateTimeWorldAccess.class);

    @Override
    public void loadChunkData(@NotNull Chunk chunk, @Nullable NetworkBuffer userData) {
        if (userData == null) return; // No saved data, probably first load

        long lastSaveTime = userData.read(NetworkBuffer.LONG);
        logger.info("loading chunk {}, {} which was saved at {}.", chunk.getChunkX(), chunk.getChunkZ(), lastSaveTime);
    }

    @Override
    public void saveChunkData(@NotNull Chunk chunk, @NotNull NetworkBuffer userData) {
        userData.write(NetworkBuffer.LONG, System.currentTimeMillis());
    }
}
```

To use an implementation of `PolarWorldAccess`, you can pass it into the `PolarLoader`;

```java
PolarWorld world = ...
PolarWorldAccess exampleWorldAccess = new UpdateTimeWorldAccess(); // You would use your own here

// Getting a SharedInstance
InstanceManager manager = MinecraftServer.getInstanceManager();
InstanceContainer instance = manager.createInstanceContainer();
SharedInstance sharedInstance = manager.createSharedInstance(instance);

// Setting the ChunkLoader
PolarLoader loader = new PolarLoader(polarWorld)
loader.setWorldAccess(exampleWorldAccess);
instance.setChunkLoader(loader);
```