## Logging nftables traffic

If your computer was infected you need to check your firewall logs. The best way to do it is to set up nftables. Netfilter doesn't logging traffic by default. You need to activate this function with a few easy steps.

### 1. Add rule to nftables.conf

```
vi /etc/nftables.conf
```

Add rule to chain output:

```
log prefix "New Output packets: ";
```
Now you nftables.conf file should looks like this:

```
#!/usr/sbin/nft -f

flush ruleset

table inet filter {
        chain input {
                type filter hook input priority 0; counter; policy accept;
        }
        chain forward {
                type filter hook forward priority 0; counter; policy accept;
        }
        chain output {
                type filter hook output priority 0; counter; policy accept;
                log prefix "New Output packets: ";
        }
}
```

Don't forget restart nftables:

```
systemctl restart nftables
```

When logging is enabled nftables starts write messages in var/log/kern.log (var/log/messages and var/log/syslog). It's not easy to read logging messages in one huge file. For that reason it's better to create separate file specially for nftables logs. Let's do it.

### 2. Create a file under /etc/rsyslog.d/my_nftables.conf containing:

```
:msg,contains,"New Output packets: " -/var/log/out_nftables.log
& stop
:msg,contains,"New Input packets: " -/var/log/in_nftables.log
& stop
```

Now rsyslogd deamon can find all messages contains keyword ("New Output packets: " and "New Input packets: ") and forward them to log files, created in var/log directory. 

Restart rsyslog deamon:

```
/etc/init.d/rsyslog restart
```

### 3. Analyze log files

Copy file from /var/log/nftables.log to your home user directory:

```
# cat /var/log/nftables.log > /home/user/Documents/my_nftableslog.txt
```

You need to analyze input and output packets when internet turn on and turn off:

```
systemctl stop networking
systemctl start networking
```


