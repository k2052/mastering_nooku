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

# The nervous system of a Nooku component  

# The model.

All the component tutorials will begin with the model. Its the most important part.