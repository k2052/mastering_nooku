# Models   
 
Nooku lacks a datamapper-esque ORM in my humble opinion its the biggest whole in Nooku's arsenal. It would offer a few
benefits namely, no need to create our DB schemas, Agnostic DB schemas, powerful querying language etc. Lets create one.

First of all we want to emulate as closely as possible Datamapper. We can envision something along the lines of

```php
class PostModel extends ORM
{   
  public function _initialize()
  {    
    parent::_initialize();
    $this->key('title', 'string', array('default' => 'Post Title'));
    $this->key('created_at', 'datetime');      
  }
}
``` 

