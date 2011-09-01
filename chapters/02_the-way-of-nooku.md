# The request model

The first thing we hit is as expected the plugin.

The plugin first loods up the core class for path/version handling and then KLoader for loading the rest of the classes.

```php JLoader::import('libraries.koowa.koowa', JPATH_ROOT); JLoader::import('libraries.koowa.loader.loader', JPATH_ROOT);
```

KLoader itself has a few basic perquisites. Nothing complicated is done to load these, its jut done using a require_once and
with Koowa::getPath();. First it grabs exception handling,

As a side note both Exception and identifier are misspelled.

It looks like Koowa follows a strict convention of first creating interfaces and then implementations. I'm not sure if I
like the obsessive level of this, but I imagine it accomplishes allot though; 1. Makes the core easily extendable. 2. Makes
sure there is thought put thought into API changes. 3. Gives a good overview of the actual interface without the extra code
to read. You cant have missing methods etc. Actually that might not be accurate I cant remember if interface implementation
methods are optional in PHP or not.  


## KIdentifier

It looks like pretty everything isn't an actual until its ran through KFactory. 