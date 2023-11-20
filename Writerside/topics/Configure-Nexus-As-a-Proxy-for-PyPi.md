# Configure Nexus As a Proxy for PyPi


## Setup in Nexus

Firstly, you will need to setup a proxy for Pypi repository in Nexus. 
This will act as a cache for the packages that are downloaded from Pypi.

To do this, follow the steps below:
<procedure title="Configure Pypi Proxy">
<step>
 From the Nexus Dashboard, click on the `Settings` cog in the top right corner.
</step>
<step>
    From the left hand menu, click on `Repositories`
</step>
<step>
    Click on the `Create repository` button
</step>
<step>
    Select `PyPi (proxy)`
</step>

<step>
<img src="nexus_proxy_pypi.png" alt=""/>
    Enter the following details:
    <ul>
        <li>Repository ID: `pypi-proxy`</li>
        <li>Repository Name: `PyPi Proxy`</li>
        <li>Remote Storage: `https://pypi.org/`</li>
    </ul>
</step>
</procedure>

Next, associate the proxy with a group, this will allow you to use the proxy as a source for packages
both from Pypi and from any private repositories you create.

<procedure title="Setup Nexus Group">
<step>
    From the Nexus Dashboard, click on the `Settings` cog in the top right corner.
</step>
<step>
    From the left hand menu, click on `Repositories`
</step>
<step>
    Click on the `Create repository` button
</step>
<step>
    Select `PyPi (group)`
</step>
<step>
    Enter the following details:
    <ul>
        <li>Repository ID: `pypi-group`</li>
        <li>Repository Name: `PyPi Group`</li>
    </ul>
</step>
<step>
    Click on the `Add` button in the `Member repositories` section
</step>
<step>
    Select the `pypi-proxy` repository and click on `Add selected`
</step>
</procedure>

<procedure title="Setting up pip.conf">
<step>
    Create a file called `pip.conf` either globally or within the venv.
</step>
<step>
    Open the file and add the following:
    <code-block xml:lang="ini">
      [global]
        index = https://$DOMAIN/repository/pypi-group/pypi
        index-url = https://$DOMAIN/repository/pypi-group/simple
      [pypi]
        repositroy: https://$DOMAIN/repository/pypi-group/
    </code-block>
</step>
<step>
    Replace `$DOMAIN` with the domain of your Nexus server. Now to test, run the following command:
    <code-block xml:lang="bash">
        pip -v install requests
    </code-block>
    verify that the package is downloaded via your Nexus server. by checking the domain.
</step>
</procedure>
