This stuff needs to go in a chapter but I haven't decided where yet.

# KObject

# KFactory       

You can get things using `KFactory::get()` and `KFactory::tmp()` but whats the difference? Well get only returns news
instance if it hasn't been created. And tmp always creates a new instance. So when should you use each one? When you need an
instance of a model for an individual item that you need persist you might do tmp. This way you don't accidentally load the
model and overwrite data on that instance.