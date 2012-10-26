> <http://www.binarytides.com/fill-text-templates-using-arrays-in-php/>

# Fill text templates using arrays in php

Text templates are often used in various applications when generating content like notification emails, invoices filled with the details of a customer or user. The best example would be a bulk email program that sends out emails to multiple users filling the details of the particular user in every individual mail. So how does a template look like. Here is a simple example ...

	$template = "
				Hello {{customer_name}}
				
				Thank you for making the purchase of ${{amount}}. 
				Your order shall be processed shortly.
				
				Regards
				SuperOnline Store
				";

That kind of an email might go out from the ecommerce platform of a store once the order of any customer is punched into the system. It might be an sms as well.

Now the values wrapped in {{ }} are the values that are dynamic and come from a database. They have to be filled every time a message is to be generated for a new customer or new order. The biggest advantage of using such templates is that they are "configurable" from the application admin area and do not require any coding skills for the administrator to edit.

So to fill them up with array here is a simple and short php function that uses preg_replace

	function bind_to_template($replacements, $template) 
	{
		return preg_replace_callback('/{{(.+?)}}/', function($matches) use ($replacements) 
		{
			return $replacements[$matches[1]];
		}, $template);
	}

[Source](https://gist.github.com/2896042)

**usage**

$template = "
			Hello {{customer_name}}
			
			Thank you for making the purchase of $ {{amount}}. 
			Your order shall be processed shortly.
			
			Regards
			SuperOnline Store
			";

# Your template tags + replacements
$row = array(
	'customer_name' => 'Mike',
	'amount' => 400,
);

	function bind_to_template($replacements, $template) 
	{
		return preg_replace_callback('/{{(.+?)}}/', function($matches) use ($replacements) 
		{
			return $replacements[$matches[1]];
		}, $template);
	}
	
	echo bind_to_template($row, $template); 

So in the above example the template is filled with values from the variable $row. This $row could have been fetched from the database for example.

Here is an alternative function that is smaller and has a simpler syntax.

	<?php
	
	$template = "
				Hello {{customer_name}}
				
				Thank you for making the purchase of $ {{amount}}. 
				Your order shall be processed shortly.
				
				Regards
				SuperOnline Store
				";
	
	# Your template tags + replacements
	$row = array(
		'customer_name' => 'Mike',
		'amount' => 400,
	);
	
	function fill_template($template, $replacements)
	{
		$final = preg_replace('#{{([^{}]+)}}#sie' , "\$replacements['\${1}']" , $template);
		
		return $final;
	}
	
	echo fill_template($template, $row);

This new function achieves the text replacement in just 1 line of code, and uses preg_replace instead of preg_replace_callback as in the earlier example.

