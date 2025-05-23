## ModAPI.hooks
ModAPI.hooks is the global generated by actual code.
It has quite a few propeties, but most of them are just foundations for more user friendly parts of the ModAPI, so I won't explain them.

### Property: ModAPI.hooks.methods
`ModAPI.hooks.methods` is a String-to-method dictionary/object of every java method. This allows you to do pretty much whatever you want in terms of modifying and hooking into code.
It's properties are defined by the package of the java class, as well as that methods name. For example, `net.minecraft.world.World`'s `spawnEntityInWorld` method will be named `ModAPI.hooks.methods.nmw_World_spawnEntityInWorld`, where `nmw` is `net.minecraft.world`. You can automate finding this by using `ModAPI.util.getMethodFromPackage("com.package.abc.MyClass", "myMethod")`, which is an alorithm that automatically finds the key in `ModAPI.hooks.methods`.

Due to the injector's patches, all game code runs by calling these ModAPI.hooks.methods functions, which allows you to call, patch, replace and modify the game's source code in realtime.

To call a method, use `ModAPI.hooks.methods[ModAPI.util.getMethodFromPackage("com.package.abc.MyClass", "myMethod")](argument1, argument2, etc);`.

Complexities with calling methods:
Methods that are described in EaglercraftX's source code as `static` can be called by replacing boolean values `true`/`false` with `1` and `0`, respectively. Strings have to be converted by using `ModAPI.util.str("string")` or one of it's aliases. To pass data from a ModAPI proxy you have to use .getRef(), eg: `ModAPI.player.getRef(); //raw java player object`.

Non static methods must be called with the object they are running on (the value of the `this` keyword in the java source). Do this by using the object as the first argument. For example, look at this code:

```javascript
ModAPI.require("world");
ModAPI.hooks.methods.nmw_World_spawnEntityInWorld(ModAPI.world.getRef(), *an entity object*);
```

- To replace a method with another, you can use:
    - `ModAPI.hooks.methods[ModAPI.util.getMethodFromPackage("com.package.abc.MyClass", "myMethod")] = function () {}`
- To intercept inputs to a method, you can us
    - ```javascript
        var myMethodName = ModAPI.util.getMethodFromPackage("com.package.abc.MyClass", "myMethod");
        const originalMethod = ModAPI.hooks.methods[myMethodName];
        ModAPI.hooks.methods[myMethodName] = function (...args) {
            //args is the array of arguments passed in.
            //on an instance method, this first (args[0]) will almost always
            //be $this, ie: the object the method is being run on
            return originalMethod.apply(this, args);
        }
        ```

Please note that if you are just calling methods, I recommend attempting to call the method directly on the object. `Eg: ModAPI.world.spawnEntityInWorld(*an entity object*)` or using `ModAPI.reflect.getClassByName("MyClass").methods.methodName.method()`;

### Property: ModAPI.hooks._teavm
`ModAPI.hooks._teavm` is usually only used for internal purposes, but it is basically a collection of every internal TeaVM method. Keep in mind that this only stores references (for performance reasons), so modifying and editing it's contents will not affect the way the game runs.