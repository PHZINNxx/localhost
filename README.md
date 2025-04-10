### localhost subdomains

Direct continuation of Charles' process:
https://inclouds.space/localhost-domains

#### setup `dnsmasq`

We'll use `dnsmasq` so all subdomains (*.localhost) get redirected to 127.0.01.
Install it (via brew or apt) and then configure it as follows:

```
echo 'port=5353' | sudo tee -a /etc/dnsmasq.conf
echo 'address=/localhost/127.0.0.1' | sudo tee -a /etc/dnsmasq.conf
sudo systemctl restart dnsmasq
```

Next, to further avoid conflicts with `systemd-resolver` add `nameserver
127.0.0.1` to the top of `/etc/resolv.conf` so that `dnsmasq` becomes the
primary DNS resolver.

#### setup caddy and manage your Caddyfile with `localhost`

Then, we use `caddy` to direct subdomains to particular ports.

You can do this by hand, but I've written a little bash script with Claude's
help to this for me. Tou can add the bash script `localhost` to our `PATH`. In
my case, I added the following line to my `.zshrc`.

```
export PATH="$PATH:$HOME/dev/localhost"
```

Then you can, e.g. `localhost add hello 8000` or `localhost remove`.

#### testing it out

```
localhost add hello 1234
echo 'hello.localhost!' > index.html
python3 -m http.server 1234
```

Now open `hello.localhost` in your browser!
