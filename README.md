dsnet is a simple tool to manage a centralised wireguard VPN. Think wg-quick
but quicker.

    Usage: dsnet <cmd>

    Available commands:

    	init   : Create /etc/dsnetconfig.json containing default configuration + new keys without loading. Edit to taste.
    	add    : Add a new peer + sync
    	up     : Create the interface, run pre/post up, sync
    	report : Generate a JSON status report to the location configured in /etc/dsnetconfig.json.
    	remove : Remove a peer by hostname provided as argument + sync
    	down   : Destroy the interface, run pre/post down
    	sync   : Update wireguard configuration from /etc/dsnetconfig.json after validating


Quick start -- install wireguard and dsnet, then:

    sudo dsnet init
    sudo dsnet up
    # edit /etc/dsnetconfig.json to taste
	dsnet add banana > dsnet-banana.conf
	dsnet add apple > dsnet-apple.conf

Copy the generated configuration file to your device and connect!

To send configurations, ffsend (with separately transferred password) or a local QR code generator may be used.

# Walkthrough of /etc/dsnetconfig.json

dsnetconfig.json is the only file the server needs to run the VPN. It contains
the server keys, peer public/shared keys and IP settings.

Currently its location is fixed as all my deployments are for a single network.
I may add a feature to allow setting of the location via environment variable
in the future to support multiple networks on a single host.

Main configuration, generated by `dsnet init` and edited manually:

    {
        "ExternalIP": "198.51.100.2",

This is the external IP that will be the value of Endpoint for the server peer
in client configs. It is automatically detected by opening a socket or using an
external IP discovery service -- the first to give a valid public IPv4 will
win.


        "ListenPort": 51820,

The port wiregard should listen on.

        "Domain": "dsnet",

The domain to copy to the report file. Not used for anything else; it's useful
for DNS integration. At one site I have a script to add hosts to a zone upon
connection by polling the report file.

        "InterfaceName": "dsnet",

The wireguard interface name.

        "Network": "10.164.236.0/22",

The CIDR network to use when allocating IPs to peers. This subnet, a `/22` in
the `10.0.0.0/16` block is generated randomly to (probably) avoid collisions
with other networks. There are 1022 addresses available. Addresses are
allocated to peers when peers are added with `dsnet add` using the lowest
available address.

        "IP": "10.164.236.1",

This is the private VPN IP of the server peer. It is the first address in the
above pool.

        "DNS": "",

If defined, this IP address will be set in the generated peer wg-quick config
files.

        "Networks": [],

This is a list of additional CIDR-notated networks that can be routed through
the server peer. They will be added under the server peer under `AllowedIPs` in
addition to the private network defined in `Network` above. If you want to
route the whole internet through the server peer, add `0.0.0.0/0` to the list
before adding peers. For more advanced options and theory, see
<https://www.wireguard.com/netns/>.

        "ReportFile": "/var/lib/dsnetreport.json",

This is the location of the report file generated with `dsnet report`. It is
suggested that this command is run via a cron job; the report can be safely
consumed by a web service or DNS integration script, for instance.

The report contains no sensitive information. At one site I use it together
with [hugo](https://gohugo.io/)
[shortcodes](https://gohugo.io/templates/shortcode-templates/) to generate a
network overview page.

        "PrivateKey": "uC+xz3v1mfjWBHepwiCgAmPebZcY+EdhaHAvqX2r7U8=",

The server private key, automatically generated and very sensitive!

        "Peers": []

The list of peers managed by `dsnet add` and `dsnet remove`. See below for format.

    }

The configuration file can be manually/programatically managed outside of dsnet
if desired; `dsnet sync` will update wireguard.

Peer configuration, `Peers: []` in `dsnetconfig.json`:

        {
            "Hostname": "test",

The hostname given via `dsnet add <hostname>`. It is used to identify the peer
in the report and for peer removal via `dsnet remove <hostname>`. It can also
be used to update a DNS zone via a custom script that operates on the report
file as mentioned above.

            "Owner": "naggie",

The owner of the peer, copied to the report file.

            "Description": "Home server",

A description of the peer, copied to the report file; the lack of which in
`wq-quick` is what inspired me to write dsnet in the first place.


            "IP": "10.164.236.2",

The private VPN IP allocated by dsnet for this peer. It is the lowest available
IP in the pool from `Network`, above.

            "Added": "2020-05-07T10:04:46.336286992+01:00",

The timestamp of when the peer was added by dsnet.

            "Networks": [],

Any other CIDR networks that can be routed through this peer.

            "PublicKey": "altJeQ/V52JZQrGcA9RiKcpZusYU6zMUJhl7Wbd9rX0=",

The public key derived from the private key generated by dsnet when the peer
was added.

            "PresharedKey": "GcUtlze0BMuxo3iVEjpOahKdTf8xVfF8hDW3Ylw5az0="

The pre-shared key for this peer. The peer has the same key defined as the
pre-shared key for the server peer. This is optional in wireguard but not for
dsnet due to the extra (post quantum!) security it provides.


        }

# Report file overview

# FAQ

> Does dsnet support IPv6?

Not currently but this is a [planned feature](https://github.com/naggie/dsnet/issues/1).

> Is dsnet production ready?

Absolutely, it's just a configuration generator so your VPN does not depend on
dsnet after adding peers. I use it in production at 2 companies so far.

Note that before version 1.0, the config file schema may change. Changes will
be made clear in release notes.

> Why are their very few issues?

I'm tracking development elsewhere using
[dstask](https://github.com/naggie/dstask). I keep public initiated issues on
github though, and will probably migrate issues over if this gains use outside
of what I'm doing.
