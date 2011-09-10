# An intro sort of thing: Definitions and things.

We will be writing a component called Forge. Its a component for managing & installing extensions. What makes it special? It
takes care of dependancies, auto updates & even keeps track of security alerts. 

Sound cool? Lets get started. 

# Getting setup   

You need to first create development folder and register your component. Refer to chapter three and get that done first.

Then we need to do is add ourself a lib directory and clone git://github.com/bookworm/forge.git. 

Do this from your code/administrator/component folder: `mkdir lib` `cd lib && git clone git://github.com/bookworm/forge.git`
then cd `forge`. The forge has a __forge_api__ so now do the following `git submodule init` and `git submodule update`. The
__forge_api__ also contains a submodule so now do `cd forge_api` and then `git submodule init` and `git submodule update`.

This will give us the agnostic forge libraries. The are a much for powerful version com_installer and the Joomla installer
helpers. All tasks for installing are split up into tasks so we can keep track of where we are it the process. This means no
fickle installing of large extensions. *cough* Joomla 1.6 *cough* [::note] I should briefly mention how influential the
[Akeeba](https://www.akeebabackup.com) codebase was on me when creating the Forge. Managing long running stuff in a php
script is hard and Nicholas did all the painful discovery work so I never had to. Thanks for your contributions to the
Joomla! world Nik! [:/note]  


# The model.     

Before we move on lets create our models dir, cd to code/administrator/component then `mkdir models`.     

## Settings.

We need a settings model to store keys and stuff, and stuff. We will just create some field in the DB and let Nooku handle
the magic for now; but like to declare classes anyway so I know what models I've.

First create the DB data 

```sql
CREATE TABLE IF NOT EXISTS `#__forge_settings` (     
  `forge_setting_id` SERIAL,
  `name` varchar(255) NOT NULL,
  `desc` mediumtext NOT NULL,
  `source_url` varchar(255) NOT NULL,
  `public_key` varchar(255) NOT NULL,
  `private_key` varchar(255) NOT NULL,
);
```  

No lets create placeholder class and file `touch models/settings.php`.

```php
class ComForgeModelSettings extends ComDefaultModelDefault
{
	public function __construct(KConfig $config)
	{
		parent::__construct($config);
	} 
} 
```

## Artifact

In Forge the concept of an artifact is essential, its a basically an extension listing. For now we can simplify the model and
allow it to get data entirely via API. But later on we might want to cache the data locally in a DB.   

Lets go ahead create our model file. Do `touch models/artifacts.php`.

Now declare the class.

```php
class ComForgeModelArtifacts extends ComDefaultModelDefault
{
	public function __construct(KConfig $config)
	{
		parent::__construct($config);
	} 
}
```

Now we're going to need access to our API, so in the constructor lets initialize an instance of Forge_API.
          
```php    
$settings = KFactory::tmp('admin::com.forge.model.settings');
$this->fapi = Forge_API::getInstance(null, null, null, $this->settings->getItems());
```   

Make sure to add `public $fapi;` to the class.  

## Extensions   

We are going to need some way to list the currently installed extensions. We are going to need to associate an extension
with not only with the joomla extension data but with and artifact as well. To do do that we need to introduce the idea of
foreign keys and joins. A one-to-many relationship needs to be establish between the extension, the artifact and Joomla
extensions table. We're also going to need to abstract away the differences on different Joomla! versions. {::note} On
Joomla! 1.6 all extensions are under jos_extensions and on 1.5 different types are split up between different tables.
{:/note}

What I'd like to have interface wise is the extension details from Joomla! tables accessible directly on the model, and the
artifact properties accessible within an object. `$model->name` and `$model->artifact->vulnerabilities` respectively.
{::note} Its important to have the artifact as an actual model instance so the we make API queries when we call its data.
{:/note} 

Now at the moment I've very little idea how joins and this relationship should be accomplished in Nooku. So let experiment a
little and see how they work. Digging around in google turns this thread
[thread](http://groups.google.com/group/nooku-framework/browse_thread/thread/e236e38d7e04071a) and their are some
suggestions made but I like to work with real world examples; so based on the second post I'm going to look to com_terms.

Pulling up the source of com_terms reveals that joins are built within `_buildQueryJoins()`. A simple search through Nooku
reveals that `buildQueryJoins()` is automatically added to our queries. So we can assemble our joins within that function,
but key is what happens to them afterwords. Do they automatically become model instances? My guess is no, we will have to do
that on our own. The question is can we abstract that out so its done automagically? I'm betting yes.

To get a picture of what happens to joins we're going to need to create some test code. What I like to keep around for this
is a copy of Nooku Server. Its got a few default components we can hackup to test things. We're going to add some stuff to
`com_articles`.  

If we open up `models/articles.php` we can see builQueryJoins() what we're going to do first is the model. Open up `articles.php` and add the following {::note} You may have to do `touch output.txt` first {:/note}

```php
$model = KFactory::get('admin::com.articles.model.articles');  
ob_start();
var_dump($model);
$out = ob_get_clean();     
file_put_contents(dirname(__FILE__) . DS .'output.txt', $out);
```  
Whoa thats allot of data! So, how can we reduce this and get something we can work with? Lets try changing the output
buffer to this:

```php
print_r(get_class_methods($model));       
print_r(get_object_vars($model));
``` 

Now have a list of stuff to get things from lets try dump `getState` and see what happens `print_r($model->getState()); `.
Interesting look at all the state vars. Now what weened is an actual item. My first though was to dump `$model->getList()`
but returning all the data results in something like 76 MB of data. So we need to get only one item. 

I'm going to try and treat this object as an array but I get not output selecting the first elem and dumping it. Which means
we probably have some sort of object. Lets debug that and get its type.

```php
$list = $model->getList();
echo gettype($list);
``` 

Yep just as I expected, its an object. Perhaps we have some querying helper methods available lets try dumping
`$model->getList()->first()` and `$model->getList()->first()`. No cigar, both return 500 errors. This probably means non
existent methods. So lets get a list of methods and try to find something we can use.

```php
print_r(get_class_methods($model->getList()));       
print_r(get_object_vars($model->getList()));
```

And voila, guess we should have done that first. Lets use the toArray method 

```php
$array = $model->getList()->toArray();  
print_r($array[0]);
```

`Undefined offset` Huh? Shouldn't we have an array of items? Lets print the whole thing `print_r($array)` and pray it
doesn't crash php. Hmm, pretty small output must mean that toArray is a true conversion, and grabs all the nested stuff. It
looks like our array keys are not ordered like I expected, they appear to be based on the DB ids. I'm beginning to wonder if
our individual items are actual model instances.

With our new found knowledge we can dump specific articles.

```php
$articles = $model->getList()->toArray();
print_r($articles[43]);
```    

Nothing but nested arrays. For this use case it seems we've discovered an inadequacy in the Nooku base classes. Lets fix it and extend them. This is covered in Chapter 8 but if you'd like to skip ahead just keep reading.

We need to download and install com_tena to continue onward..... Writing Chapt8 more here later. 
             
# The controller

## Settings

### Listing

### CRUD: Create, Read, Updating and Deleting Settings. 

## Extensions   

### Listing

### CRUD: Create, Read, Updating and Deleting Artifacts. 

## Extensions
