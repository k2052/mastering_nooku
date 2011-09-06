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

It looks like Koowa follows a strict convention of first creating interfaces and then implementations. I'm not sure if I like
the obsessive level of this, but I imagine it accomplishes allot though; 1. Makes the core easily extendable. 2. Makes sure
there is thought put thought into API changes. 3. Gives a good overview of the actual interface without the extra code to
read. 4. You cant have missing methods etc. 5. Allows class type checking due to inheritance. Actually that might not be
accurate I cant remember if interface implementation methods are optional in PHP or not.

## KIdentifier: How a path is made.

It looks like pretty much everything isn't an actual object until its ran through KFactory. An identifier is usually used in
the form of a string which is then parsed transparently into an object. Its parsed so that the loader classes like KFactory,
KLoader etc only have to "split" up that string once. 

e.g

```php
echo KFactory::get($this->getView())
    ->setLayout($layout)
    ->display();
```

Remember that concept of Joomla! applications, the distinguishing between site/administrator etc? Well thats very useful
here. You see the KIdentifier string is first and foremost abstracted at the app level. 

Lets examine an example:      

`KLoader::load('admin::com.harbour.mappings');` first becomes `root_path/admin_app_path` which is `root_path/administrator`.
                                                                        
The next declaration is an extension type declaration component, module, plugin etc. So the string now becomes
`root_path/administrator/components`. Next the extension name is added and the path becomes
`root_path/administrator/components/com_harbour`. Next the string is split at each dot '.' and each item is appended to the
path `root_path/administrator/components/com_harbour/mappings`.

The final item in a path is special and treated as a file. So the final path is:
`root_path/administrator/components/com_harbour/mappings/mappings.php`.   
 
## Adapters

Once KIdentifier is ready the Kloader class is loaded. Everything passed into a loader is in the form of and identifier string.

In loader a new concept is introduced, that of Adapters. Adapters are
are further level of abstraction which allows multiple implementations of the same class. Potentially allowing not just
cross-version compatibility but also cross-platform compatibility. Seems to me I remember talks of a WP version of Nooku
awhile back.   

Adapters have some sort prefix system internalized. I'm guessing this is to hold the platform name i.e 'Joomla'.  

Adapters in the context of KLoader's are used to abstract away the paths, so Koowa knows where to get its
models,views,controllers etc no matter if its running on J1.5 or J1.7 or WP 2.5. That allows stuff like
`KLoader::load('site::com.harbour.mappings');` to make sense to Koowa regardless of the platform.    


When we add an adapter to KLoader its added to the KLoader::_adapters array and keyed by prefix. 
     
```php  
KLoader::addAdapter(new KLoaderAdapterKoowa(Koowa::getPath()));
KLoader::addAdapter(new KLoaderAdapterJoomla(JPATH_LIBRARIES));
```     

The end result of the above is an internal KLoader array like this `$_adapters = array('K' => $adapterObject, 'J' => $adapterObject);`

This is how Koowa determines both the path to a file and its class name.
                       
Now remember those KIdentfier's? Did you wonder how it determined what was the app path? You probably assumed it was
hardcoded into the KIdentifier class. Its actually abstracted at the Joomla! plugin level. In the plugin itself before
loading up Koowa we first register the application names and their paths.

```php
KIdentifier::registerApplication('site' , JPATH_SITE);
KIdentifier::registerApplication('admin', JPATH_ADMINISTRATOR);
```  

This is cool because it allows us to extend the KIdentfier if we so desire with custom paths. Like for example
`KIdentifier::registerApplication('foo' , JPATH_SITE.DS.'specialawesomefoopath');`. 

In Koowa's case it was used to abstract away the paths on different versions of Joomla! and different platforms. Imagine
doing something like `KIdentifier::registerApplication('wpadmin', WORDPRESS_ADMIN_PATH);`, the possibilities are endless!


Finally we add Adapters to KFactory. I'm not sure exactly what the purposes are for the KFactory abstractions. But it
appears as if it allows one to alias mixins. That means we can effectively call one object with one set of methods on one
platform and then on another instantiate another object with an entirely different set of methods, but still keep the same
api.

This also allows us to to create one model and use it on both frontend and admin, but have aliases that make it map
correctly. E.g `KFactory::map('site::com.harbour.model.boats', 'admin::com.harbour.model.boats');`       

## Koowa the bodyguard: What happens before every request.

Now after the core of Koowa is loaded up it registers itself so tat it can sit in front of all the requests performing magic
before the request is handed off to the extension. `JPluginHelper::importPlugin('koowa', null, true,
KFactory::get('lib.koowa.event.dispatcher'));`

The heart of is the Koowa event dispatcher which is a custom event dispatcher. [:todo] Need to figure out what exactly this
does and write some more details. Doesn't appear to add any more hooks yet. So perhaps all thats called is
onAfterInitialise? [/:todo]    
                                
The first thing we hit (via an onAfterInitialise event) is the authorization/authentication of a user. This is the beginning
of the awesome magic Nooku provides. Nooku logs in the user and then the request is passed on to the next event.

## Dispatcher

Once Koowa has been loaded & the authentication phase handle the request is passed on to the root component file. Just like
a normal Joomla! request. Heres where the magic begins to really kick into gear.  

What we hit at this point is the Dispatcher which handles routing our requests. Now the dispatcher we are going to hit is
not from Koowa but rather the extended one from com_default. [:tip] Remember *com_default* we will be seeing allot of it in
the future. [/:tip]

To get a good idea whats happening lets examine the inheritance of a typical dispatcher.    

`ComHarbourDispatcher > ComDefaultDispatcher > KDispatcherDefault > KDispatcherAbstract > KControllerAbstract`      

Thats a mouthful isn't it? It may seem like allot but this amount of inheritance means we gain allot of power for free.   

If we take a look at KControllerAbstract we can start to gleam a great deal about how the dispatcher ultimately works. 

First and foremost every class has a construct that passes a KConfig object to the parent class. We'll examine KConfig in
more detail but for now just think of it as a fancy interface to an array. In other words, every object in KConfig maps to
KConfig::_data[$object_name].

This passing of the config allows Koowa to slowly build things up, allowing classes to independently do the work and pass
this information along in a consistent manner to later classes.    

Secondly, every class has an initialize method which also takes a KConfig object and then calls the parent initialize.
[:note] The initialize method is actually called by KObject which nearly all Koowa classes inherit from. [/:note]

In the case of the dispatcher the first that happens is KControllerAbstract gather information about the behaviors and sets
the current request object.

```php
// Set the table behaviors
if(!empty($config->behaviors)) {
    $this->addBehavior($config->behaviors);
} 

//Set the request
$this->setRequest((array) KConfig::toData($config->request));
```
KDispatcherAbstract/KDispatcherDefault determines the current controller and registers after callbacks [:see] see (add links
here) for info about callbacks.[/:see] for the dispatch method. It also takes the current request and appends it to the
config.

ComDefaultDispatcher is where the real stuff starts to occur. 