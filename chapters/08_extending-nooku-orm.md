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

# MISC

When I ran into problems with TOrm I was getting 500 errors. After fixing all the PPH errors I was still left with failure. This could mean only one thing my models were improperly implemented. The first thing I did was check this by inheriteing from `ComDefaultModelDefault` instead. It worked, so it must be my models.

Now I needed to somehow trace down what is was that Nooku needed from my model that it refused to return. 

What we can is do override the __call and __get method and see what is being called for. 


```php
public function __call($name, $args)
{ 
  ob_start();    
  var_dump($name);
  var_dump($args);
  $out = ob_get_clean();     
  file_put_contents(dirname(__FILE__) . DS .'bad.txt', $out);
  
  parent::__call($name, $args);
} 

public function __get($name)
{
  var_dump($name);
  
  parent::__get($name);
}
```      

No we change it back to the default model `ComDefaultModelDefault` and log again to a different file.

It seems at some point I (or less likely Nooku) make a __get to `package`. I'm going to bet that I'm making a call to get the
component name. Its going down the chain and ultimately failing. {::note} This is actually bad it should be caught at some
point and return false. My guess is at some point in my code I did something stupid. Like the equivalent of a redundant
sentence. {:/note}  

After a quick search I discovered I was calling $this->package in the constructor of TOrmDB. $this->package won't be
available then, so it needs to be in the initializer. Moving the call to the initializer fixes that issue.





