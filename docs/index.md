# Cisco XRd and XRD-Tools

After struggling to instantiate an XRd control-plane container for a couple of days, I stumbled upon the [ios-xr/xrd-tools](https://github.com/ios-xr/xrd-tools) github repository and it was very easy from there onwards.

Here is a summary of observations after using it for a few hours. I tested only the sample topologies provided with the [ios-xr/xrd-tools](https://github.com/ios-xr/xrd-tools) repository

### Observations:
+ XRd is IOS-XR, so no need to ellaborate
+ I can get used to the ```xr-compose``` script, with ```docker-compose.xr.yml``` for creating topologies
+ I wish the image already had a username/password set
    + I will try to customize the image and test the original topologies with it
    + **Update:** Another layer to the exisitng image did not work but if you simply update the startup-config file with the credentials, it works. Tested with the simple-bgp topology
    + Added the following creds, username: cisco, password: ciscoxrd, deployed the lab and the creds worked
```
username cisco
 group root-lr
 group cisco-support
 secret 10 $6$qtiDT/ivnLyw3T/.$JlOj2V4BOGwTJgVvl6AgodCcE6QBYHF6nyXF3ySQiEmKFti5/51Bq42Om5XVd1HuSoSr0F.illObQIzqwcrdR.
!

```
+ The `docker attach --detach-keys=ctrl-\\` is good enough for me to attach and detach the containers
    + Thinking out loud, perhaps an ssh server in the custom image would work right out of the box on the mgmt ip, there wouldn't be any need for `attach`
+ XRD-Tools uses, only docker(docker-compose) data-plane infrastructure to deploy the topology, which is why there is a docker bridge network for each segment of the topology, which is different from another tool containerlab, which uses docker and base linux both, no complaints, just an observation
+ It would be a quick and easy way to test topologies, automated with netconf and grpc


**This is just the beginning and it is a step in the right direction!**

