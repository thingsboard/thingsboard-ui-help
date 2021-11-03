# ThingsBoard UI Help Assets

Repository with assets used by ThingsBoard UI help engine.

# Development mode
 
You may run a simple web server to host the files (assuming you are in the project folder):

```shell
sudo npm install --global http-server
http-server . -p 8081 --cors
```

Now the help assets are available using url: [http://localhost:8081](http://localhost:8081).

Launch ThingsBoard with configuration parameter: 

```shell
UI_HELP_BASE_URL=http://localhost:8081
```