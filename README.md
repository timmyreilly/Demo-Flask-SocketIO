# Demo-Flask-SocketIO

Took some time to explore the Flask-SocketIO from Miguel 

Look for the blog post here: <a href="http://timmyreilly.azurewebsites.net/flask-socketio-and-more/">TimmyReilly.com</a>

<span style="color: black;">This week I've been making progress on the Huggable Cloud Pillow website.</span>

<span style="color: black;">In the process I've learned about some sweet stuff you can do with Javascript, Python, and Flask-SocketIO.</span>

The first thing to take note of is Flask.

<span style="color: black;">Flask is the tiny server that allows us to host websites using Python to deliver content to the client. While on the server side you can manage complicated back ends or other processes using Python in conjunction with Flask.</span>

<span style="color: black;">This type of development is nice, because you can start seeing results right away but can take on big projects easily.</span>

<span style="color: black;">It might not be the most robust Framework, but its great for small projects…</span>

<span style="color: black;">If you want to get into Flask Web Development checkout this extensive <a href="http://www.microsoftvirtualacademy.com/training-courses/introduction-to-creating-websites-using-python-and-flask">MVA</a>.</span>

<span style="color: black;">Small and simple, Flask is static on its own. This allows us to present static content, like templates and images easily and deals with input from the user using RESTful calls to receive input. This is great for static things with lots of user actions, but if we want something a bit more dynamic we're going to need another tool.</span>

<span style="color: black;">In this case I've found Flask-SocketIO, similar to Flask-Sockets but with a few key differences highlighted by the creator (<a href="https://twitter.com/miguelgrinberg">Miguel Grinberg</a>) <a href="http://blog.miguelgrinberg.com/post/easy-websockets-with-flask-and-gevent">here</a>.</span>

<span style="color: black;">Sockets are great for is providing live information with low latency. Basically, you can get info on the webpage without reloading or waiting for long-polling.</span>

<span style="color: black;">There are lots of ways you can extend this functionality to sharing rooms and providing communication with users and all sorts of fun stuff that is highlighted on GitHub with a great chunk of <a href="https://github.com/miguelgrinberg/Flask-SocketIO/tree/master/example">sample code.</a> The following demo is based off of these samples. </span>

<span style="color: black;">For my project, I need the webpage to regularly check for differences in the state of the cloud and present them to the client, while also changing the image the user sees.</span>

<span style="color: black;">At first I tried to implement it using sockets passing information back and forth, but that wasn't very stable.</span>

<span style="color: black;">The solution I've found, uses a background thread that is constantly running while the Flask-SocketIO Application is running, it provides a loop that I use to constantly check state of our queue.</span>

<span style="color: black;">Let's break it down…</span>
a. I need my website to display the current state of the cloud.
b. The Flask application can get the state by query our azure queue.
c. Once we determine a change of state we can display that information to the webpage.
d. To display the new state to the webpage we need to use a socket.
e. And pass the msg to be displayed.

This demo intends to break down problem a, c, d, and e. 

I've created this little guide to help another developer get going quickly, with a nice piece of code available on GitHub.

<strong>The five steps to this little demo project are as follows:</strong>
1. Install Flask-SocketIO into our Virtual Environment
2. Create our background thread
3. Have it emit the current state to our client
4. Call the background thread when our page render_template's
5. Have the Javascript Catch the emit and format our HTML.
Celebrate! Its Friday!


<strong>1.</strong>

Flask-SocketIO is a python package that is available for download using

<code>pip install Flask-SocketIO</code>


<span style="color: black;">Make sure you instal it into a Virtual Environment. <a href="http://timmyreilly.azurewebsites.net/python-flask-windows-development-environment-setup/">Check out my earlier tutorial </a>if you need help with this step.
</span>

<strong>2.</strong>

Create our background thread. 
You'll see in the sample code from Flask-SocketIO's github a simple way to send data to the client regardless of their requests.

For this example we'll be changing the current time every second and display that to our client.

Background Thread:

[sourcecode language="python"]
def background_stuff():
     """ python code in main.py """
     print 'In background_stuff'
     while True:
         time.sleep(1)
         t = str(time.clock())
         socketio.emit('message', {'data': 'This is data', 'time': t}, namespace='/test')
[/sourcecode]

<strong>3</strong>

This is the emit statement from above, but is the meat of our interface with SocketIO. Notice how it breaks down...

[sourcecode language="python"]
socketio.emit('message', {'data': 'This is data', 'time': t}, namespace='/test')
socektio.emit('tag', 'data', namespace)
[/sourcecode]

<span style="color: black;">This emit will be sending to the client (Javascript) a message called 'message'.
</span>

<span style="color: black;">When the Javascript catches this message it will be able to pull from the python dicionary msg.data and msg.time to get the result of this package.
</span>

<strong>4</strong>

<span style="color: black;">So how do we call background_stuff?
</span>

<span style="color: black;">We can call it wherever we want, but for this simple example we'll put it right in our '/' base route. So when we navigate to 127.0.0.1:5000 (Local Host) we'll see the result of our background thread call. 
</span>

<span style="color: black;">Here's our route:
</span>

<code>
@app.route('/')
def index():
    global thread
    if thread is None:
        thread = Thread(target=background_stuff)
        thread.start()
    return render_template('index.html')
</code>

<span style="color: black;">Pretty simple… Notice global thread and target=background_stuff
</span>

<span style="color: black;">Creating different background threads is a good way to iterate through your changes.
</span>
<strong>5</strong>

<span style="color: black;">Next step is catching this on the other side…
</span>
<span style="color: black;">So for our Javascript…
</span>
<span style="color: black;">we'll be using the socket.on method.
</span>
<code>
[sourcecode language="javascript"]
socket.on('message', function(msg){
    $('#test').html('&lt;p&gt;' + msg.time + '&lt;/p&gt;');
});
[/sourcecode]

<span style="color: black;">When we receive the emit labeled 'message' we'll pick up the msg from the second parameter and have it be available to our JQuery work.
</span>

<span style="color: black;">Here's the small piece of HTML that we're selecting to edit.
</span>

[sourcecode language="html"]
<body>
    <p id='test'>Hello</p>
</body>
[/sourcecode]

<span style="color: black;"><a href="https://github.com/timmyreilly/Demo-Flask-SocketIO" target="_blank">I've posted all of this code at github</a>.
</span>

<span style="color: black;">Feel free to download it and start working with dynamic sites using SocketIO. Please let me know if you have any questions!
</span>
