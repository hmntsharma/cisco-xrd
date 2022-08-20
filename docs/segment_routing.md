# Segment Routing

Topology and IP addresses as per the repo

```
               xrd-7(PCE)
               /        \
            xrd-3 --- xrd-4
             / |        | \
src --- xrd-1  |        |  xrd-2 --- dst
             \ |        | /
            xrd-5 --- xrd-6
               \        /
               xrd-8(vRR)

IP addresses
source:            10.1.1.2
xrd-1-GE2 (left ): 10.1.1.3
xrd-2-GE2 (right): 10.3.1.2
dest:              10.3.1.3
```


## XR-Compose 

Run the xr-compose script to generate the ```docker-compose.yml``` from the ```docker-compose.xr.yml```

```
lab@xrdlab:~/github/xrd-tools/samples/xr_compose_topos/segment-routing$ sudo ~/xr-compose -i localhost/ios-xr:7.7.1
INFO - Writing output docker-compose YAML to docker-compose.yml
lab@xrdlab:~/github/xrd-tools/samples/xr_compose_topos/segment-routing$
```
**Note:** I copied the xrd-tools scripts in the home directory

## Launch Topology

Launch the topology with docker-compose

```
lab@xrdlab:~/github/xrd-tools/samples/xr_compose_topos/segment-routing$ sudo docker-compose up -d
Creating network "segment-routing_xrd-2-dest" with the default driver
Creating network "segment-routing_source-xrd-1" with the default driver
Creating network "segment-routing_mgmt" with the default driver
Creating network "xrd-1-gi0-xrd-3-gi2" with the default driver
Creating network "xrd-1-gi1-xrd-5-gi2" with the default driver
Creating network "xrd-2-gi0-xrd-4-gi2" with the default driver
Creating network "xrd-2-gi1-xrd-6-gi2" with the default driver
Creating network "xrd-3-gi0-xrd-4-gi0" with the default driver
Creating network "xrd-3-gi1-xrd-5-gi1" with the default driver
Creating network "xrd-3-gi3-xrd-7-gi0" with the default driver
Creating network "xrd-4-gi1-xrd-6-gi1" with the default driver
Creating network "xrd-4-gi3-xrd-7-gi1" with the default driver
Creating network "xrd-5-gi0-xrd-6-gi0" with the default driver
Creating network "xrd-5-gi3-xrd-8-gi0" with the default driver
Creating network "xrd-6-gi3-xrd-8-gi1" with the default driver
Creating volume "xrd-1" with default driver
Creating volume "xrd-2" with default driver
Creating volume "xrd-3" with default driver
Creating volume "xrd-4" with default driver
Creating volume "xrd-5" with default driver
Creating volume "xrd-6" with default driver
Creating volume "xrd-7" with default driver
Creating volume "xrd-8" with default driver
Creating xrd-7  ... done
Creating xrd-5  ... done
Creating xrd-4  ... done
Creating xrd-8  ... done
Creating source ... done
Creating xrd-3  ... done
Creating xrd-1  ... done
Creating xrd-2  ... done
Creating xrd-6  ... done
Creating dest   ... done
lab@xrdlab:~/github/xrd-tools/samples/xr_compose_topos/segment-routing$
```

Give it another 3 to 4 minutes for the topology to become ready and all the protocols to converge to establish end to end connectivty. 

### Containers  

```
lab@xrdlab:~/github/xrd-tools/samples/xr_compose_topos/segment-routing$ sudo docker ps
CONTAINER ID   IMAGE                    COMMAND                  CREATED              STATUS              PORTS     NAMES
1cde11703e27   alpine:3.15              "/bin/sh -c 'ip rout…"   About a minute ago   Up About a minute             dest
9fc07e38d183   localhost/ios-xr:7.7.1   "/bin/sh -c /sbin/xr…"   About a minute ago   Up About a minute             xrd-1
2e24e776c2c3   localhost/ios-xr:7.7.1   "/bin/sh -c /sbin/xr…"   About a minute ago   Up About a minute             xrd-2
ee5543a989d3   localhost/ios-xr:7.7.1   "/bin/sh -c /sbin/xr…"   About a minute ago   Up About a minute             xrd-6
d3313d61ad20   localhost/ios-xr:7.7.1   "/bin/sh -c /sbin/xr…"   About a minute ago   Up About a minute             xrd-4
5a1a4f85aa4f   localhost/ios-xr:7.7.1   "/bin/sh -c /sbin/xr…"   About a minute ago   Up About a minute             xrd-3
63605a31c730   localhost/ios-xr:7.7.1   "/bin/sh -c /sbin/xr…"   About a minute ago   Up About a minute             xrd-8
8a58dc7a2b52   alpine:3.15              "/bin/sh -c 'ip rout…"   About a minute ago   Up About a minute             source
5857e851579c   localhost/ios-xr:7.7.1   "/bin/sh -c /sbin/xr…"   About a minute ago   Up About a minute             xrd-7
e895f5dffb1b   localhost/ios-xr:7.7.1   "/bin/sh -c /sbin/xr…"   About a minute ago   Up About a minute             xrd-5
lab@xrdlab:~/github/xrd-tools/samples/xr_compose_topos/segment-routing$

```

### Container Networks

```
lab@xrdlab:~/github/xrd-tools/samples/xr_compose_topos/segment-routing$ sudo docker network ls
NETWORK ID     NAME                           DRIVER    SCOPE
6e695b3d51ed   bridge                         bridge    local
3fa096a64551   host                           host      local
98eac983e612   none                           null      local
f77fc08c8ad3   segment-routing_mgmt           bridge    local
47546614c0f5   segment-routing_source-xrd-1   bridge    local
7122494bf9c0   segment-routing_xrd-2-dest     bridge    local
43b203d3505f   xrd-1-gi0-xrd-3-gi2            bridge    local
9c4b04abad23   xrd-1-gi1-xrd-5-gi2            bridge    local
843d61caa800   xrd-2-gi0-xrd-4-gi2            bridge    local
f6406a95c2d1   xrd-2-gi1-xrd-6-gi2            bridge    local
6ee4c5354bd5   xrd-3-gi0-xrd-4-gi0            bridge    local
9cd5784fcfff   xrd-3-gi1-xrd-5-gi1            bridge    local
a85d98350feb   xrd-3-gi3-xrd-7-gi0            bridge    local
4ffa878ef578   xrd-4-gi1-xrd-6-gi1            bridge    local
2e3c61a683c9   xrd-4-gi3-xrd-7-gi1            bridge    local
b271075062d5   xrd-5-gi0-xrd-6-gi0            bridge    local
eba72e670769   xrd-5-gi3-xrd-8-gi0            bridge    local
86805bf51a09   xrd-6-gi3-xrd-8-gi1            bridge    local
lab@xrdlab:~/github/xrd-tools/samples/xr_compose_topos/segment-routing$
```

## End to End Network Reachability 

As per the docker-compose comments, The IP on `source` is 10.1.1.2 and on `dest` is 10.3.1.3

```
lab@xrdlab:~/github/xrd-tools/samples/xr_compose_topos/segment-routing$ sudo docker exec source traceroute 10.3.1.3
traceroute to 10.3.1.3 (10.3.1.3), 30 hops max, 46 byte packets
 1  xrd-1.segment-routing_source-xrd-1 (10.1.1.3)  8.898 ms  1.140 ms  0.974 ms
 2  100.101.103.103 (100.101.103.103)  11.935 ms  6.642 ms  6.495 ms
 3  100.103.104.104 (100.103.104.104)  9.953 ms  6.934 ms  6.743 ms
 4  100.102.104.102 (100.102.104.102)  8.341 ms  5.984 ms  6.157 ms
 5  10.3.1.3 (10.3.1.3)  11.020 ms  7.204 ms  7.406 ms
lab@xrdlab:~/github/xrd-tools/samples/xr_compose_topos/segment-routing$
```

## Shutdown Topology

```
lab@xrdlab:~/github/xrd-tools/samples/xr_compose_topos/segment-routing$ sudo docker-compose down
Stopping dest   ... done
Stopping xrd-1  ... done
Stopping xrd-2  ... done
Stopping xrd-6  ... done
Stopping xrd-4  ... done
Stopping xrd-3  ... done
Stopping xrd-8  ... done
Stopping source ... done
Stopping xrd-7  ... done
Stopping xrd-5  ... done
Removing dest   ... done
Removing xrd-1  ... done
Removing xrd-2  ... done
Removing xrd-6  ... done
Removing xrd-4  ... done
Removing xrd-3  ... done
Removing xrd-8  ... done
Removing source ... done
Removing xrd-7  ... done
Removing xrd-5  ... done
Removing network segment-routing_xrd-2-dest
Removing network segment-routing_source-xrd-1
Removing network segment-routing_mgmt
Removing network xrd-1-gi0-xrd-3-gi2
Removing network xrd-1-gi1-xrd-5-gi2
Removing network xrd-2-gi0-xrd-4-gi2
Removing network xrd-2-gi1-xrd-6-gi2
Removing network xrd-3-gi0-xrd-4-gi0
Removing network xrd-3-gi1-xrd-5-gi1
Removing network xrd-3-gi3-xrd-7-gi0
Removing network xrd-4-gi1-xrd-6-gi1
Removing network xrd-4-gi3-xrd-7-gi1
Removing network xrd-5-gi0-xrd-6-gi0
Removing network xrd-5-gi3-xrd-8-gi0
Removing network xrd-6-gi3-xrd-8-gi1
lab@xrdlab:~/github/xrd-tools/samples/xr_compose_topos/segment-routing$
```

After Some time, I will take another look at the XRd outputs. So far, it is very promising!

# Thank You
