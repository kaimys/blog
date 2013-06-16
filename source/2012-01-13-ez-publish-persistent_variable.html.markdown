---
title: eZ Publishs persistent_variable unveiled
date: 2012-01-13
tags: english, ez-publish
---

Just because it took me a while to understand how to work with the persistent_variable in eZ Publish, I want to share the knowledge.
Usually you need persistent_variable if you want to pass a value from a template in node context to one in the pagelayout context. But some other extension might already use the variable, so you should not overwrite it.
If the persistent_variable is a hash you just can modify it in node context with.

    {set scope=global $persistent_variable = 
        $#persistent_variable | merge(hash('key', 'value')) }
    
This just adds a new key/value pair to it. It is important to use scope=global to set it and $# to access the old value in global scope.
It is easy to acces it in pagelayout context then

    {$module_result.content_info.persistent_variable.key}
    
If you are writing an operator in PHP you can access and modify it in node context with:

    $persistent_variable = array();
    if($tpl->hasVariable('persistent_variable'))
        $persistent_variable = $tpl->variable('persistent_variable');
    $persistent_variable['key'] = 'value';
    $tpl->setVariable('persistent_variable', $persistent_variable);
    
And in in pagelayout context with:

    $module_result = $tpl->variable('module_result');
    $persistent_variable = array();
    if(isset($module_result['content_info']['persistent_variable']))
        $persistent_variable = $module_result['content_info']['persistent_variable'];
    