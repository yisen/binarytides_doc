> <http://www.binarytides.com/send-out-html-emails-in-php-using-the-mail-function/>

# Send out html emails in php using the mail function

The mail function of php can be used to send not only plain text emails, but html emails too. The [documentation](http://php.net/manual/en/function.mail.php) page shows how to do that.

Here is a easy to use function that does the task and has a form very similar to the mail function.

	function html_mail($to, $subject, $message, $options)
	{
		if(isset($options['from_name']))
		{
			$headers = "From: " . $options['from_name'] . "<".$options['from_email'].">" . "\r\n";
		}
		$headers .= "Reply-To: ". strip_tags($_POST['req-email']) . "\r\n";
		//$headers .= "CC: someone@example.com\r\n";
		$headers .= "MIME-Version: 1.0\r\n";
		$headers .= "Content-Type: text/html; charset=ISO-8859-1\r\n"; 
		
		mail($to, $subject, $message, $headers);
	}

The syntax of the function has been kept similar to the mail function to make it look more like a replacement. The 4th option however is not for headers, but additional options/parameters for the email which are processed by the function itself.

**Usage**

Here is an example of how the above function can be used.

	$to = 'm00n.silv3r@gmail.com';
	$subject = 'Welcome to website';
	$from_name = 'Sunny';
	$from_email = 'no-reply@example.com';
	
	$message = '<html><body>';
	$message .= '<table rules="all" style="border-color: #666;" cellpadding="10">';
	$message .= "<tr style='background: #eee;'><td><strong>Name:</strong> </td><td>Silver Moon</td></tr>";
	$message .= "<tr><td><strong>Email:</strong> </td><td>m00n.silv3r@gmail.com</td></tr>";
	$message .= "<tr><td><strong>Location:</strong> </td><td>Moon</td></tr>";
	$message .= "</table>";
	$message .= "</body></html>";
	
	html_mail($to, $subject, $message, array('from_email' => $from_email, 'from_name' => $from_name));

So all that needs to be done is make this function available to the php script and replace 'mail' with 'html_mail'. That much is enough to convert all plain text emails into html (and yes the content has to be html too).

I find this function great for single script programs that do some sort of maintenance task and send out email reports of the task status. This function can be put right into a script without the need for any extra includes.

This technique cannot be easily used for sending attachments or use a specific smtp server. For full scale features use a more robust email library like Phpmailer.
