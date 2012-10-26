> <http://www.binarytides.com/php-tutorial-parsing-html-with-domdocument/>

# Php tutorial : Parsing html with Domdocument

The domdocument class of Php is a very handy one that can be used for a number of tasks like parsing xml, html and creating xml. It is documented [here](http://php.net/manual/en/class.domdocument.php).

In this tutorial we are going to see how to use this class to parse html content. The need to parse html happens when are you are for example writing scrapers, or similar data extraction scripts.

### Sample html

The following is the sample html file that we are going to use with DomDocument.

	<html>
		<body>
			<div id="mango">
				This is the mango div. It has some text and a form too.
				<form>
					<input type="text" name="first_name" value="Yahoo" />
					<input type="text" name="last_name" value="Bingo" />
				</form>
				
				<table class="inner">
					<tr><td>Happy</td><td>Sky</td></tr>
				</table>
			</div>
			
			<table id="data" class="outer">
				<tr><td>Happy</td><td>Sky</td></tr>
				<tr><td>Happy</td><td>Sky</td></tr>
				<tr><td>Happy</td><td>Sky</td></tr>
				<tr><td>Happy</td><td>Sky</td></tr>
				<tr><td>Happy</td><td>Sky</td></tr>
			</table>
		</body>
	</html>

### Loading the html

So the first thing to do would be to construct a domdocument object and load the html content in it. Lets see how to do that.

	// a new dom object
	$dom = new domDocument; 
	
	// load the html into the object
	$dom->loadHTML($html); 
	
	// discard white space
	$dom->preserveWhiteSpace = false;

Done. The $dom object has loaded the html content and can be used to extract contents from the whole html structure just like its done inside javascript. Most common functions are getElementsByTagName and getElementById.

Now that the html is loaded, its time to see how nodes and child elements can be accessed.

### Get an element by its html id

This will get hold of a node/element by using its ID.

	//get element by id
	$mango_div = $dom->getElementById('mango');
	
	if(!mango_div)
	{
		die("Element not found");
	}
	
	echo "element found";

**Getting the value/html of a node**

The "nodeValue" attribute of an node shall give its value but strip all html inside it. For example

	echo $mango_div->nodeValue;

The second method is to use the saveHTML function, that gets out the exact html inside that particular node.

	echo $dom->saveHTML($mango_div);

Note that the function saveHTML is called on the dom object and the node object is passed as a parameter. The saveHTML function will provide the whole html (outer html) of the node including the node's own html tags as well.

Another function called C14N does the same thing more quickly

	//echo the contents of mango_div element
	echo $mango_div->C14N();

**inner html**

To get just the inner html take the following approach. It adds up the html of all of the child nodes.

	$tables = $dom->getElementsByTagName('table');
	
	echo get_inner_html($tables->item(0));
	
	function get_inner_html( $node ) 
	{
		$innerHTML= '';
		$children = $node->childNodes;
		
		foreach ($children as $child)
		{
			$innerHTML .= $child->ownerDocument->saveXML( $child );
		}
		
		return $innerHTML;
	}
	
The function get_inner_html gets the inner html of the html element. Note that we used the saveXML function instead of the saveHTML function. The property "childNodes" provides the child nodes of an element. These are the direct children.

### Getting elements by tagname

This will get elements by tag name.

	$tables = $dom->getElementsByTagName('table');
	
	foreach($tables as $table)
	{
		echo $dom->saveHTML($table);
	}
	
The function getElementsByTagName returns an object of type DomNodeList that can be read as an array of objects of type DomNode. Another way to fetch the nodes of the NodeList is by using the item function.

	$tables = $dom->getElementsByTagName('table');
	
	echo "Found : ".$tables->length. " items";
	
	$i = 0;
	while($table = $tables->item($i++))
	{
		echo $dom->saveHTML($table);
	}

The item function takes the index of the item to be fetched. The length attribute of the DomNodeList gives the number of objects found.

### Get the attributes of an element

Every DomNode has an attribute called "attributes" that is a collection of all the html attributes of that node.
Here is a quick example

	$tables = $dom->getElementsByTagName('table');
	
	$i = 0;
	
	while($table = $tables->item($i++))
	{
		foreach($table->attributes as $attr)
		{
			echo $attr->name . " " . $attr->value . "<br />";
		}
	}

To get a particular attribute using its name, use the "getNamedItem" function on the attributes object.

	$tables = $dom->getElementsByTagName('table');
	
	$i = 0;
	
	while($table = $tables->item($i++))
	{
		$class_node = $table->attributes->getNamedItem('class');
		
		if($class_node)
		{
			echo "Class is : " . $table->attributes->getNamedItem('class')->value . PHP_EOL;
		}
	}

### Children of a node

A DomNode has the following properties that provide access to its children

1. childNodes
2. firstChild
3. lastChild

code:

	$tables = $dom->getElementsByTagName('table');
	
	$table = $tables->item(1);
	
	//get the number of rows in the 2nd table
	echo $table->childNodes->length; 
	
	//content of each child
	foreach($table->childNodes as $child)
	{
		echo $child->ownerDocument->saveHTML($child);
	}

**Checking if child nodes exist**

The hasChildNodes function can be used to check if a node has any children at all.  
Quick example

	if( $table->hasChildNodes() )
	{
		//print content of children
		foreach($table->childNodes as $child)
		{
			echo $child->ownerDocument->saveHTML($child);
		}
	}

### Comparing 2 elements for equality

It might be needed to check if the element in 1 variable is the same as the element in another variable. The function "isSameNode" is used for this. The function is called on one node, and the other node is passed as the parameter. If the nodes are same, then boolean true is returned.

	$tables = $dom->getElementsByTagName('table');
	$table = $tables->item(1);
	$table2 = $dom->getElementById('data');
	var_dump($table->isSameNode($table2));

The var_dump would show true , indicating that the tables in both $table and $table2 are the same.

### Conclusion

The above examples showed how Domdocument can be used to access elements in an html document in an object oriented manner. Domdocument can not only parse html but also create/modify html and xml. In later articles we shall see how to do that.






