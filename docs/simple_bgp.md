# Simple BGP 

Topology as per the repo ```source <--> xr-1 <--> xr-2 <--> dest```

## XR-Compose 

Run the xr-compose script to generate the ```docker-compose.yml``` from the ```docker-compose.xr.yml```

```
lab@xrdlab:~/github/xrd-tools/samples/xr_compose_topos/simple-bgp$ sudo ~/xr-compose -i localhost/ios-xr:7.7.1
INFO - Writing output docker-compose YAML to docker-compose.yml
lab@xrdlab:~/github/xrd-tools/samples/xr_compose_topos/simple-bgp$
```
**Note:** I copied the xrd-tools scripts in the home directory

## Launch Topology

Launch the topology with docker-compose

```
lab@xrdlab:~/github/xrd-tools/samples/xr_compose_topos/simple-bgp$ sudo docker-compose up -d
Creating network "simple-bgp_xrd-2-dest" with the default driver
Creating network "simple-bgp_source-xrd-1" with the default driver
Creating network "simple-bgp_mgmt" with the default driver
Creating network "xr-1-gi1-xr-2-gi0" with the default driver
Creating volume "xr-1" with default driver
Creating volume "xr-2" with default driver
Pulling dest (alpine:3.15)...
3.15: Pulling from library/alpine
9621f1afde84: Pull complete
Digest: sha256:69463fdff1f025c908939e86d4714b4d5518776954ca627cbeff4c74bcea5b22
Status: Downloaded newer image for alpine:3.15
Creating xr-2   ... done
Creating xr-1   ... done
Creating dest   ... done
Creating source ... done
lab@xrdlab:~/github/xrd-tools/samples/xr_compose_topos/simple-bgp$
```

The docker compose file uses an ```alpine:3.15``` docker image to emulate as hosts, which is pulled from the docker repository when used for the first time

### Containers  

```
lab@xrdlab:~/github/xrd-tools/samples/xr_compose_topos/simple-bgp$ sudo docker ps
CONTAINER ID   IMAGE                    COMMAND                  CREATED              STATUS              PORTS     NAMES
d9c64d9342f4   localhost/ios-xr:7.7.1   "/bin/sh -c /sbin/xr…"   About a minute ago   Up About a minute             xr-1
c06ada3e07b1   alpine:3.15              "/bin/sh -c 'ip rout…"   About a minute ago   Up About a minute             source
5a7981d7fde3   alpine:3.15              "/bin/sh -c 'ip rout…"   About a minute ago   Up About a minute             dest
96ae55e77f82   localhost/ios-xr:7.7.1   "/bin/sh -c /sbin/xr…"   About a minute ago   Up About a minute             xr-2
lab@xrdlab:~/github/xrd-tools/samples/xr_compose_topos/simple-bgp$
```

### Container Networks

```
lab@xrdlab:~/github/xrd-tools/samples/xr_compose_topos/simple-bgp$ sudo docker network ls
NETWORK ID     NAME                      DRIVER    SCOPE
6e695b3d51ed   bridge                    bridge    local
3fa096a64551   host                      host      local
98eac983e612   none                      null      local
916aed1cab06   simple-bgp_mgmt           bridge    local
569004635c3f   simple-bgp_source-xrd-1   bridge    local
57eb8370acc0   simple-bgp_xrd-2-dest     bridge    local
5749aabe0b90   xr-1-gi1-xr-2-gi0         bridge    local
lab@xrdlab:~/github/xrd-tools/samples/xr_compose_topos/simple-bgp$
```

## End to End Network Reachability 

As per the docker-compose comments, The IP on `source` is 10.1.1.2 and on `dest` is 10.3.1.3

```
lab@xrdlab:~/github/xrd-tools/samples/xr_compose_topos/simple-bgp$ sudo docker exec source traceroute 10.3.1.3
traceroute to 10.3.1.3 (10.3.1.3), 30 hops max, 46 byte packets
 1  xr-1.simple-bgp_source-xrd-1 (10.1.1.3)  1.585 ms  1.090 ms  0.857 ms
 2  10.2.1.3 (10.2.1.3)  2.722 ms  2.814 ms  2.174 ms
 3  10.3.1.3 (10.3.1.3)  2.377 ms  2.797 ms  2.457 ms
lab@xrdlab:~/github/xrd-tools/samples/xr_compose_topos/simple-bgp$
```

## Cisco XRd Checks

I couldn't find an easier way to login to the docker containers running XRd, other than using ```docker attach``` as below, used ```ctrl-\``` as escape character to detach from the container

```
lab@xrdlab:~/github/xrd-tools/samples/xr_compose_topos/simple-bgp$ sudo docker attach --detach-keys=ctrl-\\ xr-1

!!!!!!!!!!!!!!!!!!!! NO root-system username is configured. Need to configure root-system username. !!!!!!!!!!!!!!!!!!!!

         --- Administrative User Dialog ---


  Enter root-system username: cisco
  Enter secret:
  Enter secret again:
Use the 'configure' command to modify this configuration.
User Access Verification

Username: cisco
Password:


RP/0/RP0/CPU0:ios#sh route
Sat Aug 20 19:46:12.151 UTC

Codes: C - connected, S - static, R - RIP, B - BGP, (>) - Diversion path
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2, E - EGP
       i - ISIS, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, su - IS-IS summary null, * - candidate default
       U - per-user static route, o - ODR, L - local, G  - DAGR, l - LISP
       A - access/subscriber, a - Application route
       M - mobile route, r - RPL, t - Traffic Engineering, (!) - FRR Backup path

Gateway of last resort is not set

C    10.1.1.0/24 is directly connected, 00:01:24, GigabitEthernet0/0/0/0
L    10.1.1.3/32 is directly connected, 00:01:24, GigabitEthernet0/0/0/0
C    10.2.1.0/24 is directly connected, 00:01:24, GigabitEthernet0/0/0/1
L    10.2.1.2/32 is directly connected, 00:01:24, GigabitEthernet0/0/0/1
B    10.3.1.0/24 [200/0] via 10.2.1.3, 00:00:46
C    172.30.0.0/24 is directly connected, 00:01:24, MgmtEth0/RP0/CPU0/0
L    172.30.0.2/32 is directly connected, 00:01:24, MgmtEth0/RP0/CPU0/0
RP/0/RP0/CPU0:ios#sh bgp summary
Sat Aug 20 19:46:14.642 UTC
BGP router identifier 10.2.1.2, local AS number 100
BGP generic scan interval 60 secs
Non-stop routing is enabled
BGP table state: Active
Table ID: 0xe0000000   RD version: 9
BGP main routing table version 9
BGP NSR Initial initsync version 5 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0
BGP scan interval 60 secs

BGP is operating in STANDALONE mode.


Process       RcvTblVer   bRIB/RIB   LabelVer  ImportVer  SendTblVer  StandbyVer
Speaker               9          9          9          9           9           0

Neighbor        Spk    AS MsgRcvd MsgSent   TblVer  InQ OutQ  Up/Down  St/PfxRcd
10.2.1.3          0   100       4       4        9    0    0 00:00:54          3

RP/0/RP0/CPU0:ios#sh bgp neighbor 10.2.1.3 routes
Sat Aug 20 19:46:20.202 UTC
BGP router identifier 10.2.1.2, local AS number 100
BGP generic scan interval 60 secs
Non-stop routing is enabled
BGP table state: Active
Table ID: 0xe0000000   RD version: 9
BGP main routing table version 9
BGP NSR Initial initsync version 5 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0
BGP scan interval 60 secs

Status codes: s suppressed, d damped, h history, * valid, > best
              i - internal, r RIB-failure, S stale, N Nexthop-discard
Origin codes: i - IGP, e - EGP, ? - incomplete
   Network            Next Hop            Metric LocPrf Weight Path
* i10.2.1.0/24        10.2.1.3                 0    100      0 ?
*>i10.3.1.0/24        10.2.1.3                 0    100      0 ?
* i172.30.0.0/24      10.2.1.3                 0    100      0 ?

Processed 3 prefixes, 3 paths
RP/0/RP0/CPU0:ios#sh bgp neighbor 10.2.1.3 advertised-routes
Sat Aug 20 19:46:23.621 UTC
Network            Next Hop        From            AS Path
10.1.1.0/24        10.2.1.2        Local           ?
10.2.1.0/24        10.2.1.2        Local           ?
172.30.0.0/24      10.2.1.2        Local           ?

Processed 3 prefixes, 3 paths
RP/0/RP0/CPU0:ios#read escape sequence
lab@xrdlab:~/github/xrd-tools/samples/xr_compose_topos/simple-bgp$
```

## Shutdown Topology

```
lab@xrdlab:~/github/xrd-tools/samples/xr_compose_topos/simple-bgp$ sudo docker-compose down
Stopping xr-1   ... done
Stopping source ... done
Stopping dest   ... done
Stopping xr-2   ... done
Removing xr-1   ... done
Removing source ... done
Removing dest   ... done
Removing xr-2   ... done
Removing network simple-bgp_xrd-2-dest
Removing network simple-bgp_source-xrd-1
Removing network simple-bgp_mgmt
Removing network xr-1-gi1-xr-2-gi0
lab@xrdlab:~/github/xrd-tools/samples/xr_compose_topos/simple-bgp$
```


## Thank You
