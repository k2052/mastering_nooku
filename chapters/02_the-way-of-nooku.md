# The request model

The first thing we hit is as expected the plugin.

The plugin first loods up the core class for path/version handling and then KLoader for loading the rest of the classes.

```php 
JLoader::import('libraries.koowa.koowa', JPATH_ROOT); 
JLoader::import('libraries.koowa.loader.loader', JPATH_ROOT);
```

KLoader itself has a few basic prerequisites. Nothing complicated is done to load these, its jut done using a require_once
and with Koowa::getPath();. First it grabs exception handling,

As a side note both Exception and identifier are misspelled.

It looks like Koowa follows a strict convention of first creating interfaces and then implementations. I'm not sure if I
like the obsessive level of this, but I imagine it accomplishes allot though; 1. Makes the core easily extendable. 2. Makes
sure there is thought put thought into API changes. 3. Gives a good overview of the actual interface without the extra code
to read. You cant have missing methods etc. Actually that might not be accurate I cant remember if interface implementation
methods are optional in PHP or not.  

## KIdentifier

It looks like pretty much everything isn't an actual object until its ran through KFactory. An indentifeir is usually used in
the form of a string which is then parsed transparently into an object, so tthe laoder classes like KFactory, KLoader etc
only have to split up that string once.

e.g

```php
echo KFactory::get($this->getView())
    ->setLayout($layout)
    ->display();
```    
 
## Adapters

Once KIdentifier is ready the Kloader class is loaded. Everything passed into a loader is in the form of and identifier string.

In loader a new concept is introduced, that of Adapters. Adapters are
a further level of abstraction which allows multiple implementations of the same class. Potentially allowing not just
cross-version compatibility but also cross-platform compatibility. Seems to me I remember talks of a WP version of Nooku
awhile back.   

Adapters have some sort prefix system internalized. I'm guessing this is to hold the platform name i.e 'Joomla'.     

Adapters in teh context of KLoaders are used to abstract away the paths, so Koowa know where to get its
models,views,controllers etc no matter if its on J1.5 or J1.7 or WP 2.5. That allows stuff like
`KLoader::load('site::com.harbour.mappings');` to make sense to Koowa not matter what the platform.

