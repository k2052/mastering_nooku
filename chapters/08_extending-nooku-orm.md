# Models   
 
Nooku lacks a datamapper-esque ORM in my humble opinion its the biggest hole in Nooku's arsenal. It would offer a few
benefits namely, no need to create our DB schemas, Agnostic DB schemas, powerful querying language etc. Lets create one. 

First of all we want to emulate as closely as possible Datamapper. We can envision something along the lines of

```php
class PostModel extends TOrm
{   
  public function _initialize()
  {    
    parent::_initialize();
    $this->key('title', 'string', array('default' => 'Post Title'));
    $this->key('created_at', 'datetime');      
  }
}
``` 

Since we are going to be using a different sort of state model we need to start all the way back at KModelAbstract

```php
abstract class TOrmAbstract extends KObject implements KObjectIdentifiable      
{
  
}
```

For now we will leave this blank and start implementing things in TOrm. The first thing we will need to do is pull out the keys. 