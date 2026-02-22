
Used to Discover and Fingerprint IKE hosts (port 500 or 4500).
Discovery, by determining which hosts are running IKE.
Fingerprinting, by determining which IKE implementation the hosts are using.
Has the following flags:

- -A -> Agressive Mode, better at catching hashes
- -P -> Shows Agressive Mode pre-shared keys

---

An example of ike-scan command: 

```zsh
ike-scan -A <SERVER_IP>
```