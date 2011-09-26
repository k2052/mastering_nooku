# Registering
 
The first thing we need to do is register our component.     

## Joomla 1.5   

```sql
INSERT INTO `#__components` ( `id`, `name`, `link`, `menuid`, `parent`, 
`admin_menu_link`, `admin_menu_alt`, `option`, `ordering`, `admin_menu_img`, 
`iscore`, `params`, `enabled`) 
VALUES(NULL, 'Forge', 'option=com_forge', 0, 0,'option=com_forge', 'Forge', 
'com_forge', 0, '', 0, '', 1)    
```
 
## Joomla 1.6                        

```sql
INSERT INTO `#__extensions`    ( `extension_id`, `name`, `type`, `element`,
`folder`, `client_id`, `enabled`, `access`, `protected`, `manifest_cache`, 
`params`, `custom_data`, `system_data`, `checked_out`, `checked_out_time`, 
`ordering`, `state`) VALUES(NULL, 'com_forge', 'component',
'com_forge', '', '1', '1', '1', '0', '', '', '', '', 0, '0000-00-00 00:00:00','0','0'     
```    

# Structuring your code for development.

Once we've got our code structured we can do sym links. cd into the root of your joomla install (or nooku server).
We can use the symlinker tool to create the symlinks, this shortens 3-5 commands into one. 

Run `symlinker /Users/kenerickson/com_tena/code PATH_TO_JOOMLA_INSTALL_ROOT`. This is actually a very simple add all it does
is `ln -s` to the (site or administrator)/components/com_tena

# The nervous system of a Nooku component  

# The model.

All the component tutorials will begin with the model. Its the most important part.