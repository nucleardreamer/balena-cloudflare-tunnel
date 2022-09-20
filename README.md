# Balena cloudflare tunnel example

At balena, we have the concept of a [public URL](https://www.balena.io/docs/learn/develop/runtime/#public-device-urls), which routes your device's web ports over our VPN, and gives it a URL to access them.

This is extremely useful for development, but not fit for running mission critical applications, as we don't have an SLA or guarentee of uptime on the public device URL feature.

By using cloudflare tunnel, you are getting a near 1-1 equivilent of the public URL feature, but with _many_ more options. You would want to look at using this service for rolling out business critical access to your devices in the field.

## How it works

Simply, the `cloudflared` client container runs along side your other application services, allowing you to specify what gets exposed, and to what domain/path. The flow looks something like this:

```

   Your Balena Device       Local LAN
┌───────────────────────┐
│                       │       ▲
│ ┌───────────────────┐ │       │
│ │  My App (port 80) ├─┼───────┘
│ └─────────▲─────────┘ │
│           │           │
│ ┌─────────┴─────────┐ │
│ │ Cloudflare Tunnel │◄├───── http://localhost/
│ └───────────────────┘ │              ▲
│                       │              │
└───────────────────────┘      Cloudflare Domain
                                  mysite.com
                                       ▲
                                       │
                            mysite.com/cool-device
```

So, your device is not only acting as normal on your local network, but is also tunneled and mapped to any specific location on your existing domain on the publicly assessible internet.

## Setup

You will need to set the `TUNNEL_TOKEN` environment variable for your device, which you get from 'Connect your app' in your Cloudflare tunnel. You can find instructions on setting it up [here](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-guide/remote/)

Here is an example of the setup to use in your `docker-compose.yml`:

```
version: '2'
services:
  
  tunnel:
    image: cloudflare/cloudflared:latest
    network_mode: host
    restart: always
    command: tunnel run
    privileged: true
    cap_add:
      - SYS_ADMIN
      - NET_ADMIN
    labels:
      io.balena.features.dbus: '1'
    environment:
      DBUS_SYSTEM_BUS_ADDRESS: "unix:path=/host/run/dbus/system_bus_socket"
      # the TUNNEL_TOKEN variable is given to you in the cloudflare tunnel setup. It will be picked up automatically by this container, and you will need to set it in the balena dashboard. You can keep your token in the docker-compose file, but know that it will not be hidden by anyone that has access to view your releases
      # TUNNEL_TOKEN: XXXXXXXX
```

## Architectures

If you are looking for support on most other architectures besides `amd64` and `arm64`, there is a community multiarch repository maintained by `@maggie0002` [here](https://github.com/maggie0002/cloudflared-docker-multiarch/pkgs/container/cloudflared)
