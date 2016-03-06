# Introduction #

Please see below sample code for accessing openhab's REST API.


# jquery #

Accessing REST API via jquery (tested with jquery 2.0 and Chrome v26.0

Read state of an item:

```
function getState()
{
	var request = $.ajax
	({
		type       : "GET",
		url        : "http://192.168.100.21:8080/rest/items/MyLight/state"
	});

	request.done( function(data) 
	{ 
		console.log( "Success: Status=" + data );
	});

	request.fail( function(jqXHR, textStatus ) 
	{ 
		console.log( "Failure: " + textStatus );
	});
}
```

Set state of an item:

```
function setState( txtNewState )
{
	var request = $.ajax
	({
		type       : "PUT",
		url        : "http://192.168.100.21:8080/rest/items/MyLight/state",
		data       : txtNewState, 
		headers    : { "Content-Type": "text/plain" }
	});

	request.done( function(data) 
	{ 
		console.log( "Success" );
	});

	request.fail( function(jqXHR, textStatus ) 
	{ 
		console.log( "Failure: " + textStatus );
	});
}
```

Send command to an item:
```
function sendCommand( txtCommand )
{
	var request = $.ajax
	({
		type       : "POST",
		url        : "http://192.168.100.21:8080/rest/items/MyLight",
		data       : txtCommand,
		headers    : { 'Content-Type': 'text/plain' }
	});

	request.done( function(data) 
	{ 
		console.log( "Success: Status=" + data );
	});

	request.fail( function(jqXHR, textStatus ) 
	{ 
		console.log( "Failure: " + textStatus );
	});
}
```

# cURL #

Accessing REST API via [cURL](http://curl.haxx.se). cURL is useful on shell scripts (Win/Linux/OS X) or e.g. on Automator (OS X).

Get state:
```
curl http://192.168.100.21:8080/rest/items/MyLight/state
```

Set state:
```
curl --header "Content-Type: text/plain" --request PUT --data "OFF" http://192.168.100.21:8080/rest/items/MyLight/state
```

Send command:
```
curl --header "Content-Type: text/plain" --request POST --data "ON" http://192.168.100.21:8080/rest/items/MyLight
```

# PHP #

Simple PHP function to set a switch state using the REST interface.

Set state:
```
function doPostRequest($item, $data) {
  $url = "http://192.168.1.121:8080/rest/items/" . $item;

  $options = array(
    'http' => array(
        'header'  => "Content-type: text/plain\r\n",
        'method'  => 'POST',
        'content' => $data  //http_build_query($data),
    ),
  );

  $context  = stream_context_create($options);
  $result = file_get_contents($url, false, $context);

  return $result;
}
```

Example function use:
```
doPostRequest("doorbellSwitch", "ON");
```

If the post was successful the function will return the state you set, EG above returns "ON"