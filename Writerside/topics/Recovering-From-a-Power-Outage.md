# Recovering From a Power Outage

## Introduction
In the case of a power out to the house, once power has been restored you will need to reboot the
big PC on the ground in order to for the kubernetes cluster to totally recover. 

<warning>
Just because the internet is working, doesn't mean it will continue too. The Kubernetes cluster 
that runs the DNS server relies on a MySQL database that is hosted on that PC. Although the cluster works
using high availability, it's not guaranteed to work if the database is down.
</warning>

In order to return the service here's some quick steps to follow:

<procedure title="Recover Cluster in the event of outage">
<step>
<p>Power on the big PC, using the power button on top.</p>
</step>
<step><p>Wait for approx 10 minutes for all the services to recover.</p></step>
<step><p>Verify the services are working visiting <a href="http://192.168.50.21/admin/login.php">http://192.168.50.21/admin/login.php</a></p>
</step>
</procedure>

## Conclusion
If you follow the above and are able to see pihole admin page, then the cluster has recovered. If not, 
then there's possibly a bigger problem related to standing up Pi Hole. 

##  Troubleshooting

<list>
<li>
<p>Guide for checking the DNS configuration on the cluster nodes.</p>
</li>
</list>
