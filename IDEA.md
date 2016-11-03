# Ideas

Good ideas that should not be forgotten and should be taken into account when developing.

## server

### Bootstrapper
A separate bootstrapper application should be provided that makes server configuration easier and can update the server to the latest version. 

GUI application is preferred using `JavaFX`.

### One-time invite tokens - **DONE** :white_check_mark:
Players of a particular server can be invited using a one-time invite token. The token is an arbitrary character sequence. A unique token must be generated for each player.

The token must be entered when connecting to the server.

After a specfied amount of time, the token expires and a new one must be generated.

### Parallel code execution with merge
Player-defined codes are executed in a parallel fashion and changes to the game world are then merged. This is done using a clever trick. All API calls by different users are caught by proxies and collected. After the code finished executing, the calls can be subsequently applied to the game world and conflicts can be eliminated without the need of additional care implied by parallelism.

### Server integration tests
The server should be tested using a custom `WebSocket` client that issues requests and asserts the response.  