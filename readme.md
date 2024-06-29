## Step 0) 
Prerequisite - Remove/Uninstall/Delete Previous proxychains Files.
This might not be necessary, but to prevent any possible issues, I did this.
Delete any previously downloaded or installedproxychains/proxychains-ng files, especially if installed with brew or if you installed manually and ran make install.
There might be a better way to do this, but I just ran a find command and deleted any file containing the roxychai substring:
```
find / -name "*roxychai*" 2>/dev/null
/some/directory/proxychains.conf
/some/other/directory/proxychains4
sudo rm /some/directory/proxychains.conf
sudo rm /some/other/directory/proxychains4

# 用这句删除所有proxychains相关的文件
find / -name "*roxychai*" 2>/dev/null | xargs -I {} sudo rm {}


```
## Step 1)
# a) Download proxychains-ng with git and cd into the folder:
```
    git clone https://github.com/rofl0r/proxychains-ng 
    
    cd proxychains-ng
```
b) Set the CFLAGS to the following (different to above solution) and run ./configure (one-liner):
```
    CFLAGS="-arch arm64" LDFLAGS="-arch arm64" ./configure
```
c) Run 'make', but do NOT run 'install' or 'make install'. Just 'make':
```
    make
```
d) Rename the 'proxychains-ng' folder downloaded with the 'git' command in step a) (prevents issues in later step):
```
    cd ../
    mv proxychains-ng proxychains-ng-temp
```
## Step 2)
Perform pich4ya's solution to enable arm64e, as outlined below:

    a) Shutdown the device.

    b) Press and hold (long-press) the Power button to turn the device on and boot into Recovery mode.
       Hold the power button until it says 'Loading Recovery Mode' (or something similar).

    c) Click on Settings/Options.

    d) Click your user and enter password.

    e) Open a Terminal session with ⌘CMD+SHIFT+T.

    f) Disable csrutil with below command. You will need to type 'y' and hit enter, as well as entering your password.
       NOTE: This takes about a minute to execute, be patient.
        csrutil disable

    g) Reboot the device into normal mode (can use the same terminal session):
        reboot

    h) Once rebooted into normal mode and logged in, open a Terminal session and enable arm64e support:
        sudo nvram boot-args=-arm64e_preview_abi

    i) Once again, reboot into normal mode (can use terminal session):
        sudo reboot
## Step 3)
Similar to Step 1), but with a twist on the CFLAGS

1 Download proxychains-ng with git again (this is why we renamed the folder of the previous install) and cd into the folder:
```
git clone https://github.com/rofl0r/proxychains-ng 
        
cd proxychains-ng
```

2 Set the CFLAGS and run ./configure (as outlined in pich4ya's solution):

```
CFLAGS="-arch arm64e" LDFLAGS="-arch arm64e" ./configure --prefix=/usr/local --bindir=/usr/local/bin --libdir=/usr/local/lib --fat-binary-m1
```
3 Run 'make', but do NOT run 'install' or 'make install'. Just 'make':
```
make
```
## Step 4) THE FIX
Remember in Step 1) how we installed proxychains-ng, renamed the folder to proxychains-ng-temp and left it?
We basically need to replace the proxychains4 binary file we just compiled, with the binary we compiled earlier (in the proxychains-ng-temp folder).

1 Delete the newly-compiled proxychains4 file in the newly-downloaded `proxychains-ng` folder:
```
        ls -l | grep proxychains
        proxychains-ng
        proxychains-ng-test
        
        cd proxychains-ng
        
        rm proxychains4
```
2 Move the proxychains4 binary compiled earlier from the proxychains-ng-temp folder into the proxychains-ng folder:
```
        ls -l | grep proxychains
        proxychains-ng
        proxychains-ng-test
        
        mv proxychains-ng-test/proxychains4 proxychains-ng
```


IMPORTANT NOTES:
This should now work. You can delete the old proxychains-ng-test folder.
We disabled csrutil in an earlier step, which provides some critical system security.
I can confirm that you CAN now re-enable it and have proxychains work without issues.
Disabling csrutil is only required to run the command in Step 2) d), so we can re-enable it now.
Re-enable steps:

    # a) Shutdown the device.

    # b) Press and hold (long-press) the Power button to turn the device on and boot into Recovery mode.
    #    Hold the power button until it says 'Loading Recovery Mode' (or something similar).

    # c) Click on Settings/Options.

    # d) Click your user and enter password.

    # e) Open a Terminal session with ⌘CMD+SHIFT+T.

    # f) Enable csrutil with below command. You will need to type 'y' and hit enter, as well as entering your password.
    #    NOTE: This takes about a minute to execute, be patient.
        csrutil enable

    # g) Reboot the device into normal mode (can use the same terminal session):
        reboot
You need to specify the proxychains.conf configuration file every time you run proxychains.
We can specify this using the -f switch.
The proxychains.conf file is stored in the src folder of the proxychains-ng folder downloaded with git:
        pwd
        /Users/sorryyyy/path-to-proxychains-folder/proxychains-ng
        
        ./proxychains -f src/proxychains.conf [COMMAND]

## Step 5) 安装proxychyains

```
# 在proxychains-ng
make install
# 添加配置文件
sudo vim /etc/proxychains.conf

```
```
# proxychains.conf  VER 4.x
#
#        HTTP, SOCKS4a, SOCKS5 tunneling proxifier with DNS.


# The option below identifies how the ProxyList is treated.
# only one option should be uncommented at time,
# otherwise the last appearing option will be accepted
#
#dynamic_chain
#
# Dynamic - Each connection will be done via chained proxies
# all proxies chained in the order as they appear in the list
# at least one proxy must be online to play in chain
# (dead proxies are skipped)
# otherwise EINTR is returned to the app
#
strict_chain
#
# Strict - Each connection will be done via chained proxies
# all proxies chained in the order as they appear in the list
# all proxies must be online to play in chain
# otherwise EINTR is returned to the app
#
#round_robin_chain
#
# Round Robin - Each connection will be done via chained proxies
# of chain_len length
# all proxies chained in the order as they appear in the list
# at least one proxy must be online to play in chain
# (dead proxies are skipped).
# the start of the current proxy chain is the proxy after the last
# proxy in the previously invoked proxy chain.
# if the end of the proxy chain is reached while looking for proxies
# start at the beginning again.
# otherwise EINTR is returned to the app
# These semantics are not guaranteed in a multithreaded environment.
#
#random_chain
#
# Random - Each connection will be done via random proxy
# (or proxy chain, see  chain_len) from the list.
# this option is good to test your IDS :)

# Make sense only if random_chain or round_robin_chain
#chain_len = 2

# Quiet mode (no output from library)
#quiet_mode

## Proxy DNS requests - no leak for DNS data
# (disable all of the 3 items below to not proxy your DNS requests)

# method 1. this uses the proxychains4 style method to do remote dns:
# a thread is spawned that serves DNS requests and hands down an ip
# assigned from an internal list (via remote_dns_subnet).
# this is the easiest (setup-wise) and fastest method, however on
# systems with buggy libcs and very complex software like webbrowsers
# this might not work and/or cause crashes.
proxy_dns

# method 2. use the old proxyresolv script to proxy DNS requests
# in proxychains 3.1 style. requires `proxyresolv` in $PATH
# plus a dynamically linked `dig` binary.
# this is a lot slower than `proxy_dns`, doesn't support .onion URLs,
# but might be more compatible with complex software like webbrowsers.
#proxy_dns_old

# method 3. use proxychains4-daemon process to serve remote DNS requests.
# this is similar to the threaded `proxy_dns` method, however it requires
# that proxychains4-daemon is already running on the specified address.
# on the plus side it doesn't do malloc/threads so it should be quite
# compatible with complex, async-unsafe software.
# note that if you don't start proxychains4-daemon before using this,
# the process will simply hang.
#proxy_dns_daemon 127.0.0.1:1053

# set the class A subnet number to use for the internal remote DNS mapping
# we use the reserved 224.x.x.x range by default,
# if the proxified app does a DNS request, we will return an IP from that range.
# on further accesses to this ip we will send the saved DNS name to the proxy.
# in case some control-freak app checks the returned ip, and denies to 
# connect, you can use another subnet, e.g. 10.x.x.x or 127.x.x.x.
# of course you should make sure that the proxified app does not need
# *real* access to this subnet. 
# i.e. dont use the same subnet then in the localnet section
#remote_dns_subnet 127 
#remote_dns_subnet 10
remote_dns_subnet 224

# Some timeouts in milliseconds
tcp_read_time_out 15000
tcp_connect_time_out 8000

### Examples for localnet exclusion
## localnet ranges will *not* use a proxy to connect.
## note that localnet works only when plain IP addresses are passed to the app,
## the hostname resolves via /etc/hosts, or proxy_dns is disabled or proxy_dns_old used.

## Exclude connections to 192.168.1.0/24 with port 80
# localnet 192.168.1.0:80/255.255.255.0

## Exclude connections to 192.168.100.0/24
# localnet 192.168.100.0/255.255.255.0

## Exclude connections to ANYwhere with port 80
# localnet 0.0.0.0:80/0.0.0.0
# localnet [::]:80/0

## RFC6890 Loopback address range
## if you enable this, you have to make sure remote_dns_subnet is not 127
## you'll need to enable it if you want to use an application that 
## connects to localhost.
# localnet 127.0.0.0/255.0.0.0
# localnet ::1/128

## RFC1918 Private Address Ranges
# localnet 10.0.0.0/255.0.0.0
# localnet 172.16.0.0/255.240.0.0
# localnet 192.168.0.0/255.255.0.0

### Examples for dnat
## Trying to proxy connections to destinations which are dnatted,
## will result in proxying connections to the new given destinations.
## Whenever I connect to 1.1.1.1 on port 1234 actually connect to 1.1.1.2 on port 443
# dnat 1.1.1.1:1234  1.1.1.2:443

## Whenever I connect to 1.1.1.1 on port 443 actually connect to 1.1.1.2 on port 443
## (no need to write :443 again)
# dnat 1.1.1.2:443  1.1.1.2

## No matter what port I connect to on 1.1.1.1 port actually connect to 1.1.1.2 on port 443
# dnat 1.1.1.1  1.1.1.2:443

## Always, instead of connecting to 1.1.1.1, connect to 1.1.1.2
# dnat 1.1.1.1  1.1.1.2

# ProxyList format
#       type  ip  port [user pass]
#       (values separated by 'tab' or 'blank')
#
#       only numeric ipv4 addresses are valid
#
#
#        Examples:
#
#               socks5  192.168.67.78   1080    lamer   secret
#               http    192.168.89.3    8080    justu   hidden
#               socks4  192.168.1.49    1080
#               http    192.168.39.93   8080
#
#
#       proxy types: http, socks4, socks5, raw
#         * raw: The traffic is simply forwarded to the proxy without modification.
#        ( auth types supported: "basic"-http  "user/pass"-socks )
#
[ProxyList]
# add proxy here ...
# meanwile
# defaults set to "tor"
socks5  127.0.0.1 27890
```
