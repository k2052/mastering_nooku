# An intro sort of thing: Definitions and things.

We will be writing a component called Forge. Its a component for managing & installing extensions. What makes it special? It
takes care of dependancies, auto updates & even keeps track of security alerts. 

Sound cool? Lets get started.

# Getting setup

The first thing we need to do is add ourself a lib directory and clone git://github.com/bookworm/forge.git. 

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

## Artifact

In Forge the concept of an artifact is essential, its a basically an extension listing. For now we can simplify the model and
allow it to exist entirely via API. But later on we might want to cache the data locally in a DB.   

Lets go ahead create our model file 

## Extensions       

## Registering the extension
 
The last thing we need to do is register our component.     

### Joomla 1.5   

```sql
INSERT INTO `#__components` ( `id`, `name`, `link`, `menuid`, `parent`, 
`admin_menu_link`, `admin_menu_alt`, `option`, `ordering`, `admin_menu_img`, 
`iscore`, `params`, `enabled`) 
VALUES(NULL, 'Forge', 'option=com_forge', 0, 0,'option=com_forge', 'Forge', 
'com_forge', 0, '', 0, '', 1)    
```
 
### Joomla 1.6                        

```sql
INSERT INTO `#__extensions`    ( `extension_id`, `name`, `type`, `element`,
`folder`, `client_id`, `enabled`, `access`, `protected`, `manifest_cache`, 
`params`, `custom_data`, `system_data`, `checked_out`, `checked_out_time`, 
`ordering`, `state`) VALUES(NULL, 'com_forge', 'component',
'com_forge', '', '1', '1', '1', '0', '', '', '', '', 0, '0000-00-00 00:00:00','0','0'    
```