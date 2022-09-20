# Balena cloudflare tunnel example

At balena, we have the concept of a [public URL](https://www.balena.io/docs/learn/develop/runtime/#public-device-urls), which routes your device's web ports over our VPN, and gives it a URL to access them.

This is extremely useful for development, but not fit for running mission critical applications, as we don't have an SLA or guarentee of uptime on the public device URL feature.

By using cloudflare tunnel, you are getting a near 1-1 equivilent of the [public URL](https://www.balena.io/docs/learn/develop/runtime/#public-device-urls) feature, but with _many_ more options. You would want to look at using this service for rolling out business critical access to your devices in the field.

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

You will need to get a `TUNNEL_TOKEN` environment variable set for your device, which you get from 'Connect your app' in your Cloudflare tunnel. You can find instructions on setting it up [here](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-guide/remote/)
