# Troubleshoot Your Network Load Balancer<a name="load-balancer-troubleshooting"></a>

The following information can help you troubleshoot issues with your Network Load Balancer\.

## A registered target is not in service<a name="target-not-in-service"></a>

If a target is taking longer than expected to enter the `InService` state, it might be failing health checks\. Your target is not in service until it passes one health check\. For more information, see [Health Checks for Your Target Groups](target-group-health-checks.md)\.

Verify that your instance is failing health checks and then check for the following:

**A security group does not allow traffic**  
The security groups associated with an instance must allow traffic from the load balancer using the health check port and health check protocol\.

**A network access control list \(ACL\) does not allow traffic**  
The network ACL associated with the subnets for your instances must allow inbound traffic on the health check port and outbound traffic on the ephemeral ports \(1024\-65535\)\. The network ACL associated with the subnets for your load balancer nodes must allow inbound traffic on the ephemeral ports and outbound traffic on the health check and ephemeral ports\.

## Requests are not routed to targets<a name="requests-not-routed"></a>

Check for the following:

**A security group does not allow traffic**  
The security groups associated with the instances must allow traffic on the listener port from client IP addresses \(if targets are specified by instance ID\) or load balancer nodes \(if targets are specified by IP address\)\.

**A network access control list \(ACL\) does not allow traffic**  
The network ACLs associated with the subnets for your VPC must allow the load balancer and targets to communicate in both directions on the listener port\.

**The targets are in an Availability Zone that is not enabled**  
If you register targets in an Availability Zone but do not enable the Availability Zone, these registered targets do not receive traffic from the load balancer\.

**The instance is in a peered VPC**  
If you have instances in a peered VPC, you must register them with your load balancer by IP address, not by instance ID\.

## Targets receive more health check requests than expected<a name="health-check-interval"></a>

Health checks for a Network Load Balancer are distributed and use a consensus mechanism to determine target health\. Therefore, targets can receive more than the number of health checks configured through the `HealthCheckIntervalSeconds` setting\.

## Targets receive fewer health check requests than expected<a name="too-few-health-checks"></a>

Check whether `net.ipv4.tcp_tw_recycle` is enabled\. This setting is known to cause issues with load balancers\. The `net.ipv4.tcp_tw_reuse` setting is considered a safer alternative\.

## Unhealthy targets receive requests from the load balancer<a name="no-healthy-targets"></a>

If there is at least one healthy registered target for your load balancer, the load balancer routes requests only to its healthy registered targets\. If there are only unhealthy registered targets, the load balancer routes requests to all registered targets\.

## Connections time out for requests from a target to its load balancer<a name="loopback-timeout"></a>

Check whether you have an internal load balancer with targets registered by instance ID\. Internal load balancers do not support hairpinning or loopback\. When you register targets by instance ID, the source IP addresses of clients are preserved\. If an instance is a client of an internal load balancer that it's registered with by instance ID, the connection succeeds only if the request is routed to a different instance\. Otherwise, the source and destination IP addresses are the same and the connection times out\.

If an instance must send requests to a load balancer that it's registered with, do one of the following:
+ Register instances by IP address instead of instance ID\. When using Amazon Elastic Container Service, use the `awsvpc` network mode with your tasks to ensure that target groups require registration by IP address\.
+ Ensure that containers that must communicate are on different container instances\.
+ Use an Internet\-facing load balancer\.

## Performance decreases when moving targets to a Network Load Balancer<a name="load-balancer-performance"></a>

Both Classic Load Balancers and Application Load Balancers use connection multiplexing, but Network Load Balancers do not\. Therefore, your targets can receive more TCP connections behind a Network Load Balancer\. Be sure that your targets are prepared to handle the volume of connection requests they might receive\.

## Port allocation errors connecting through AWS PrivateLink<a name="port-allocation-errors-privatelink"></a>

If your Network Load Balancer is associated with a VPC endpoint service, it supports 55,000 simultaneous connections or about 55,000 connections per minute to each unique target \(IP address and port\)\. If you exceed these connections, there is an increased chance of port allocation errors\. To fix the port allocation errors, add more targets to the target group\.