
Used for sending HTTP requests to an URL. Has the following flags:

- -X method -> Specifies the request (GET, POST, PUT, ...)
- -H text -> Adds text to the header of the request
- -L -> Follows redirects and if the request is a POST, resends it
- -b DATA -> Passes data in the Cookie header to the HTTP server
- -d DATA -> Passes data in the POST request to the HTTP request
- --connect-timeout s -> Limits by s seconds the time the connection phase takes

---

An example of curl command:

```zsh
curl -X POST -d '{"search":"london"}' -b 'PHPSESSID=c1nsa6op7vtk7kdis7bcnbadf1' -H 'Content-Type: application/json' http://<SERVER_IP>:<PORT>/search.php
```