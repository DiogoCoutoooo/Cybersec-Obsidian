
Used for Web Enumeration. Has the following commands:

- dir -> Enumerate directories/files

With the following flags (dir command):

- --debug -> Enables the debug output
- -u XXXX:YYYY ->  The target URL
- -w  PATH -> The wordlist to use (at PATH)
- -x aaa, bbb -> File extensions to search for
- -U -> Username for basich auth
- -P -> Password for basic auth
- -m -> The HTTP method to use (default GET)
- -H -> Specify HTTP headers ('Header1: val1')
- -r -> Enable follow redirects
- -c -> Cookies to use in the requests
- --retry -> Retry on request timeouts
- --retry-attemps int -> Number of attemps to retry (default 3)

---

An example of gobuster command:

```zsh
gobuster dir -u http://<SERVER_IP>:<PORT> -w /usr/share/dirbuster/wordlists/directory-list-2.3-small.txt -x php,html
```