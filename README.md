# Nginx + Let's Encrypt + Docker Registry in Docker Compose

This is an example `docker-compose.yaml` file that shows how you can run your own internal registry
to create your own private, secure registry with automatic SSL certificates courtesy of Let's Encrypt
with TLS termination on Nginx level.

## Configuring the registry

As a previous note, you should know that **the Docker registry has a non-persistent storage** for the
images pushed to it, which means that on the next `docker-compose up -d` where your configuration changed,
you'll loose the images you pushed before. It's possible to make it persistent by adding a volume mounted
to the filesystem by editing the `volumes` section under the `registry:2` definition and adding a line like:

```
  - ./registry/images:/var/lib/registry
```

Which will store the images inside the `registry` folder under `images`. Other alternatives more performant
are [available in the Docker documentation](https://docs.docker.com/registry/deploying/#storage-customization).

There are a couple of steps you need to follow through. First, create a `htpasswd` file with the user
and password you want to use for your registry. You can use the `htpasswd` command by itself if you
know how, or you can use the Docker provided alternative:

```
docker run --entrypoint htpasswd registry:2 -Bbn myuser mypassword > registry/auth/htpasswd
```

Change `myuser` for the username you want to set on your registry, and `mypassword` to the expected
password. The resulting file will be in your current directory, called `htpasswd`. Make sure the file
is at `registry/auth/htpasswd` (which means you should already have a clone of this repo, since that
folder exists here).

The second thing is, you will have to decide on a domain name you want to use with your registry and
all your other apps from the Docker Compose file. You can create `registry.example.com` by simply creating
an A or AAAA DNS record pointing to the IP of the machine that hosts your `docker-compose` stack.
Wildcard A / AAAA records can also be used so you can point `*.apps.example.com` besides the registry
domain to get a way to route additional services using the same Docker Compose stack.

By default, the Nginx proxy in this Docker Compose doesn't support the Registry out of the box. Initially
you might want to increase the body size and allow chunks via HTTP requests, to do so, create a file
named `registry.example.com_location` under `domains/vhost.d` changing, of course, the hostname in the
file name mentioned before, but leaving the `_location` part intact. The file should contain the following
instructions:

```conf
proxy_set_header Host $http_host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
proxy_read_timeout 900;
client_max_body_size 0;
chunked_transfer_encoding on;
```

And lastly, edit the `docker-compose.yml` file to match the hostname of your registry and / or add new
containers to the stack.

Have fun!
