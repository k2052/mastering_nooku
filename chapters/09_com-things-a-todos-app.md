# Intro

We need to com_tena through its paces what better way to do that than port another app to Joomla!. I've recently started to
utilize coffee-script & spine in my projects & it would be a real test of Nooku to develop a full fledged (albeit simple) web
app.

Lets port the example spine app [spine.todos](https://github.com/maccman/spine.todos) we will give it a simple grid interface
in the backend & full featured interface on the frontend.   

# The Shared   

## Models

We're going to only really need one model to hold the Todos.


```php
class ComThingsModelTodos extends ComTenaModelDefault
{
  public function __construct(KConfig $config)
	{
		parent::__construct($config);     
	} 
}   
```  
Lets add some keys.

```php
public function _initialize(KConfig $config)
{    
  parent::_initialize($config);
  $this->key('name', 'string')
    ->key('object_id', 'string')
    ->key('users_id', 'foreign_key')
    ->key('done', 'bool');
}
```     

Now the great thing about spine is that its persistence layer is RESTful. We only need to pass it the correct base url (the
controller path) and it will figure the magic out. {::see} [Spine
Docs](http://maccman.github.com/spine/#s-models-persistence) {:/see}

# The Backend  

Lets create our entry point file in site. `touch things.php`        

Then add the dispatcher

```php
echo KFactory::get('admin::com.things.dispatcher')->dispatch();       
```    

Lets load the todos model and play with it a bit.

# The Frontend   

Lets create our entry point file in site. `touch things.php`  

Then add the dispatcher

```php
echo KFactory::get('admin::com.things.dispatcher')->dispatch();       
```