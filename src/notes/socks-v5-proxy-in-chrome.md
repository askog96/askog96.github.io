# Chrome with SOCKS v5 proxy

These are the settings I use when I want Chrome to send traffic through a SOCKS v5 proxy.

## Why I use this

- I work a lot with SSH tunnels and needed a way to access different GUIs in the browser through them.
- Placeholder: why I used Chrome with a SOCKS proxy instead of changing system proxy settings
- Placeholder: what kind of environment I used this in

## Launch Chrome with a SOCKS proxy

To send Chrome traffic through a SOCKS v5 proxy on `myproxy:8080`, start Chrome with:

```bash
--proxy-server="socks5://myproxy:8080"
--host-resolver-rules="MAP * 0.0.0.0 , EXCLUDE myproxy"
```

## What to check when debugging

The first thing I usually check is the Proxy tab in Chrome's net internals page to see what Chrome is actually using:

- `chrome://net-internals/#proxy`

Then I check the DNS tab to make sure Chrome is not doing local DNS lookups:

- `chrome://net-internals/#dns`

If I want to trace how Chrome handled a specific request, I use the Events tab:

- `chrome://net-internals/#events`

## Example

Create a local SOCKS proxy with SSH:

```bash
ssh -C2qTnN -D 8080 user@domain.com
```

Then launch Chrome with:

```bash
google-chrome --proxy-server="socks5://127.0.0.1:8080" --host-resolver-rules="MAP * ~NOTFOUND , EXCLUDE 127.0.0.1"
```

## My setup

On macOS I use:

```bash
open -a Google\ Chrome --args --proxy-server="socks5://127.0.0.1:8080" --host-resolver-rules="MAP * ~NOTFOUND , EXCLUDE 127.0.0.1"
```

## NET::ERR_CERT_INVALID

If Chrome shows `NET::ERR_CERT_INVALID`, there is a hidden bypass on the error page. Click anywhere on the page and type:

```text
thisisunsafe
```

I would only use that for testing in an environment I trust.

## Notes to add later

- Placeholder: what tripped me up the first time
- Placeholder: how I verified DNS was going through the proxy
- Placeholder: any difference I noticed between Linux and macOS
