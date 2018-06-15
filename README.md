# PHPNotifications
PHPNotifications is an online platform that allows you to create a network in your application. For example, you may have an application that allows the posting of threads. As a user, I might try and view the thread. If the creator of the thread, at any point, decides to delete the thread, PHPNotifications can make it easy to alert anyone viewing the thread that it has been deleted and alter what they see. Likewise with sending and recieving messages.

# Getting Started
Download or clone the project and move the content into your application. Require the PHPNotifications file and create a new instance in your factory.

```php
require_once( dirname( __FILE__ ) . '/PHPNotifications.php' );
$factory = ( object ) array(
  'PhpNotifier' => ( new PHPNotifications () )->setDebug( true )
);
```

setDebug() method will cast a logger. The logger is used in the Server listener when a requests comes through but the event cannot be triggered due to it not existing.

# Creating A Request
You can create a request using a PHPNotificationPacket retrieved by the getPacket() method. There are 4 types of methods, POST, GET, PUT, DELETE. Each packet has an action argument which the Server listener will use as a sort of packet header.

```php
$packet = $application	->notifier
			->getPacket ()
			->setTarget( array( 'Some unique chat tokens' ) )
			->setMethod( 'POST' )
			->setArgs( json_encode( array(
				'action' 	=> 'message',
				'from'	 	=> 'This users ID',
				'message' => 'This is an example'
			) ) );

$application->notifier
	->sendPacket( $packet );
```

# Creating A Listener
You can create a listener using the PHPNotificationListener retrieved by the getListener() method. You can then use the bindAction() method to create a route. Param 1 takes in the action, or the packet header as explained in the creating a request section. The second param takes in the type of request to stop forgery requests. The third param is a closure which takes in all of the arguments that're expected by the request. The final param is optional title for readabilty.

```php
$listener = $application->notifier->getListener();

$listener->bindAction('message', 'POST', function($from, $message) {
	// Handle incomming message and return packet
}, 'Message Recieved');

$response = $listener->match();

if($response &&
	is_callable($response['target']))
		call_user_func_array($response['target'], $response['params']);
```

# An Example Client Side Listener
Here, we're sentding a HTTP GET request to the server listener which will keep checking for any incomming packets.

```javascript
$(function() {
	setInterval(function() {
		$.get('/public/notifications').done(function(response) {
			// Handle incoming packets
		});
	}, 1000); // every 1 second in ms
});
```
