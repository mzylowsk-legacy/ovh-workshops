## LoadBalancing

This experiment requires:
* at least two VM's in GRA9 with private (vRack) network
* HTTP server configured and answering on those VM's

1. Create loadbalancer.
WAIT for LB status to be ACTIVE before any other command related to LB will be invoked.
```
openstack loadbalancer create --name lb1 --vip-network [private_network_id]
openstack loadbalancer show lb1 #check the provisioning_status, we need active state.
```
Please note that we have now one more VM in openstack server list. This is amphora VM - it is used for doing loadbalancing routing around added members (endpoints).

2. Create listener for given LB. We want to balance traffic for HTTP on port 80:
```
openstack loadbalancer listener create --protocol HTTP --protocol-port 80 --name listener1 lb1
```

3. Create pool for members to loadbalance. This is a pool where we use ROUND_ROBIN algorithm for choosing members (endpoints VM).
```
openstack loadbalancer pool create --lb-algorithm ROUND_ROBIN --listener listener1 --protocol HTTP --name pool1
```

4. It is time to add members to a pool:
```
openstack loadbalancer member create --subnet-id [SUBNET_ID] --address [VM_IP_ADDRESS] --protocol-port 80 pool1
openstack loadbalancer member create --subnet-id [SUBNET_ID] --address [VM_IP_ADDRESS] --protocol-port 80 pool1
```

5. Now our loadbalancer is working and we can curl LB VIP address. But before this we will create health monitor for our pool - this will protect from calling members that are dead or are not working properly:
```
openstack loadbalancer healthmonitor create --delay 5 --timeout 2 --max-retries 1 --type HTTP pool1
```

6. Let's curl our loadbalancer vip IP:
```
ubuntu@vm1:~$ curl [VIP_IP_HERE]
```

---

Clean Up Loadbalance objects:

We need to remove in this order: members/monitor, pool, listener, loadbalancer:

```
openstack loadbalancer member list pool1 #list members id, d79a3b49-860b-47b0-8a39-e3a5efcb4153, 5f725dc0-7c01-4d32-9a32-53c061ac0c43
openstack loadbalancer member delete pool1 d79a3b49-860b-47b0-8a39-e3a5efcb4153
openstack loadbalancer member delete pool1 5f725dc0-7c01-4d32-9a32-53c061ac0c43
openstack loadbalancer healthmonitor list # 30d2356d-0bd0-4cf9-9f7a-20c00ed3b5e5
openstack loadbalancer healthmonitor delete 30d2356d-0bd0-4cf9-9f7a-20c00ed3b5e5
openstack loadbalancer pool delete pool1
openstack loadbalancer listener delete listener1
openstack loadbalancer delete lb1
```
