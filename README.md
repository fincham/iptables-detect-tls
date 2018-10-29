# iptables-detect-tls

Android contains code to try and log or prevent applications from using non-TLS TCP or UDP to commnuicate (https://android.googlesource.com/platform/system/netd/+/master/server/StrictController.cpp).

This is a re-implementation of the same matches in plain `iptables` syntax, allowing you to make more creative use of this capability.

## Usage

For instance, if you want to prevent your local machine from making a TCP or UDP connection without using TLS, you might do something like this as part of your startup or interface bringup scripts:

    fincham@postel:~$ sudo iptables-restore -n < v4
    fincham@postel:~$ sudo ip6tables-restore -n < v6
    fincham@postel:~$ sudo iptables -A OUTPUT -p tcp --dport 22 -j ACCEPT
    fincham@postel:~$ sudo iptables -A OUTPUT -p tcp --dport 53 -d 8.8.8.8 -j ACCEPT
    fincham@postel:~$ sudo iptables -A OUTPUT -p udp --dport 53 -d 8.8.8.8 -j ACCEPT
    fincham@postel:~$ sudo iptables -A OUTPUT -p tcp -j st_clear_detect
    fincham@postel:~$ sudo iptables -A OUTPUT -p udp -j st_clear_detect
    fincham@postel:~$ sudo ip6tables -A OUTPUT -p tcp --dport 22 -j ACCEPT
    fincham@postel:~$ sudo ip6tables -A OUTPUT -p tcp --dport 53 -d 2001:4860:4860::8888 -j ACCEPT
    fincham@postel:~$ sudo ip6tables -A OUTPUT -p udp --dport 53 -d 2001:4860:4860::8888 -j ACCEPT
    fincham@postel:~$ sudo ip6tables -A OUTPUT -p tcp -j st_clear_detect
    fincham@postel:~$ sudo ip6tables -A OUTPUT -p udp -j st_clear_detect

You will likely need to replace 8.8.8.8 and 2001:4860:4860::8888 with the address of the recursive nameserver your machine uses, or otherwise adapt the script to prevent any known non-TLS services you need (such as SSH, DNS or NTP) from being blocked.

Alternatively you may want to modify the `st_clear_caught` chain to just log or count the connections (`nfacct` would be useful, particularly when combined with https://github.com/catalyst/collectd-network/blob/master/netfilter_acct.py) instead of outright rejecting them.

It should be possible to use these rules in a stateful firewall as well, though this has not been extensively tested.

## Caveats

A number of things have not been tested by me. They include:

* Detecting non-TLS connections in a stateful firewall (there may be problems with v6 fragment handling, for instance)
* Detecting UDP TLS (DTLS)
* Particular accuracy of the rules (though they do seem to work)

Improvements and further test results would be gladly accepted :)
