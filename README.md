### localhost subdomains

Map subdomains of localhost to ports.

```
  hello.localhost → localhost:1234
     db.localhost → localhost:5432
printer.localhost → localhost:9100
```

This is a direct continuation of Charles' process:
https://inclouds.space/localhost-domains

For this to work, we'll need to do two things:
1. redirect `*.localhost` to `127.0.0.1`
2. have a server on `127.0.0.1` that reverse proxies to the right port

This was tested on Ubuntu 22. Your mileage may vary.

#### redirect with `dnsmasq`

_NB: You might not actually need this, `systemd-resolved` might already be doing
the redirecting! See [hacker news](https://news.ycombinator.com/item?id=43644434)_.

To get `dnsmasq` to redirect all subdomains (*.localhost) to 127.0.0.1, install
it (via brew or apt) and then configure it as follows:

```
echo 'port=5353' | sudo tee -a /etc/dnsmasq.conf
echo 'address=/localhost/127.0.0.1' | sudo tee -a /etc/dnsmasq.conf
sudo systemctl restart dnsmasq
```

NB: we use port 5353 to avoid conflicts with the `systemd-resolved`

Next, to further avoid conflicts with `systemd-resolved` add `nameserver
127.0.0.1` to the top of `/etc/resolv.conf` so that `dnsmasq` becomes the
primary DNS resolver.

#### serve with `caddy` + `localhost`

Then, we use `caddy` to direct subdomains to particular ports.

You can do this by writing a Caddyfile by hand. I've written a little bash
script with Claude's help to do this for me. You can add the bash script
`localhost` to your `PATH`. In my case, I added the following line to my
`.zshrc`.

```
export PATH="$PATH:$HOME/dev/localhost"
```

Now you can `localhost add hello 8000` or `localhost remove`.

#### testing it out

Run a local server!
```
echo 'hello.localhost!' > index.html
python3 -m http.server 1234
```

Name the port!
```
localhost add hello 1234
```

Now open `hello.localhost` in your browser!
