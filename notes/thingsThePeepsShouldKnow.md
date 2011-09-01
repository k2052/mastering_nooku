1. Class names are camel cased.
2. Class names reflect folder structures  
  e.g FooModelCats goes in `com_foo/models/boats`
3. Everything cascades with fallbacks along the way.
  e.g FooDispatcher > ComDefaultDispatcher > KDispatcherDefault > KDispatcher 
  
Seems to that it might useful to introduce people to the process of digging into a component or any php app to figure out how it works. My process of digging into things might interest people, especially those who learn in a similar manner.