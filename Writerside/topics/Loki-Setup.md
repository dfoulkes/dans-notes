# Loki Setup


## Setup of Loki via Helm

## Configure Loki in Grafana

<warning>
 Remember to check the output of the helm deployment for the URL to use with Grafana.
</warning>

<warning>
By default Loki requires a `X-Scope-OrgID` header to be set in the request. This is to prevent unauthorized access to the logs.
</warning>

<procedure title="Configure Grafana">
<step>
Go to Data Sources in Grafana
</step>
<step>
Select Loki
</step>
<step>
Set the URL to the Loki URL something like http://loki-gateway.monitoring.svc.cluster.local
</step>
<step>
Set the `X-Scope-OrgID` header in Grafana, under 
datasources > Loki > HTTP headers
</step>
<step>
<table>

<tr>
<th>Header</th>
<th>Value</th>
</tr>


<tr>
<td>X-Scope-OrgID</td>
<td>default</td>
</tr>
</table>
</step>
</procedure>

