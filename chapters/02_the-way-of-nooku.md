# The request model

The first thing we hit is as expected the plugin.

The plugin first loods up the core class for path/version handling and then KLoader for loading the rest of the classes. 

```php
JLoader::import('libraries.koowa.koowa', JPATH_ROOT);
JLoader::import('libraries.koowa.loader.loader', JPATH_ROOT);           
```

KLoader itself has a few basic perquisites. Nothing complicated is done to load these, its jut done using a require_once and with Koowa::getPath();. First it grabs exception handling,

As a side note both Exception and identifier are misspelled.

It looks like Koowa follows a very 