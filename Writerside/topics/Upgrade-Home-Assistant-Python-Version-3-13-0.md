# Upgrade Home Assistant Python Version 3.13.0

<procedure>
<step>
<p>Check current python version</p>
<code-block lang="bash">
    python3 --version
</code-block>
</step>
<step>
<p>Download Python 3.13.0</p>
<code-block lang="bash">
    wget https://www.python.org/ftp/python/3.13.0/Python-3.13.0.tgz
</code-block>
</step>
<step>
<p>Unpack it.</p>
<code-block lang="bash">
    sudo tar --no-same-owner -xzf Python-3.13.0.tgz
</code-block>
</step>
<step>
<p>SqlLite3 Requirement</p>
<code-block lang="bash">
sudo systemctl stop home-assistant@homeassistant.service
wget https://sqlite.org/2021/sqlite-autoconf-3360000.tar.gz
tar -xvf sqlite-autoconf-3360000.tar.gz
./configure
make
sudo make install
sudo cp /usr/local/lib/*sql* /usr/lib/x86_64-linux-gnu/
</code-block>
</step>
<step>
<p> Make and install Python 3.13</p>
<code-block lang="bash">
cd Python-3.13.0/
sudo apt update
sudo apt install -y build-essential zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev libssl-dev libreadline-dev libffi-dev curl libbz2-dev
sudo ./configure --enable-optimization
sudo make -j$(nproc)
sudo make altinstall
sudo update-alternatives --install /usr/bin/python3 python3 /usr/local/bin/python3.13 1
sudo update-alternatives --config python3
0
</code-block>
<warning>
<p>You must first have the sqllite3 step completed before DOING THIS!!!</p>
</warning>
</step>
<step>
<p>validate build</p>
<code-block lang="bash">
python3 --version
</code-block>
</step>

<step>
<p>House Keeping</p>
<code-block lang="Bash">
sudo apt --fix-broken install
sudo apt install python3-minimal grub2-common
sudo python3 -m ensurepip --upgrade
sudo python3 -m pip install --upgrade pip
sudo apt update && sudo apt upgrade
sudo apt autoremove --purge
sudo apt clean
</code-block>
</step>

<step>
<p>Freeze current home assistant python packages</p>
<code-block lang="bash">
cd /home/homeassistant/.homeassistant
sudo su homeassistant
source /srv/homeassistant/bin/activate
pip3 freeze –local > requirements-2024.12.txt
exit
</code-block>
</step>

<step>
<p>Stop the HASS instance</p>
<code-block lang="Bash">
sudo systemctl stop home-assistant@homeassistant
</code-block>
</step>

<step>
<p>backup homeassistant config</p>
<code-block lang="bash">
cd /srv
sudo mv homeassistant homeassistantold-20241226
</code-block>
</step>

<step>
<p> make new home assistant folder</p>
<code-block lang="bash">
sudo mkdir /srv/homeassistant
sudo chown -R homeassistant:homeassistant homeassistant
</code-block>
</step>

<step>
<p>Create new virtual environment</p>
<code-block lang="bash">
sudo -u homeassistant -H -s
cd /srv/homeassistant
python3.13 -m venv .
</code-block>
</step>

<step>
<p>Activate the Python venv</p>
<code-block lang="bash">
source /srv/homeassistant/bin/activate
</code-block>
</step>

<step>
<p>Upgrade Requirements</p>
<code-block lang="bash">
sudo -u homeassistant -H -s
cd /home/homeassistant/.homeassistant
pip3 install --upgrade wheel
pip install --upgrade pip
</code-block>
<tip>
<p>Ensure you are in the virtual environment</p>
</tip>
<warning>
<p>Services might fail on restart, be sure to check the logs for other dependencies</p>
</warning>
</step>


<step>
<p>Upgrade HASS on the new Python version</p>
<code-block lang="bash">
cd /srv/homeassistant/lib
sudo -u homeassistant -H -s
source /srv/homeassistant/bin/activate
pip3 install --upgrade hass-nabucasa
pip3 install pyOpenSSL --upgrade
pip install --use-deprecated=legacy-resolver cryptography==42.0.0
pip3 install --upgrade homeassistant
pip install mysqlclient
pip install mysql-connector-python
</code-block>
</step>

<step>
<p>Update the Home Assistant runtime in the Systemd service</p>
<code-block lang="bash">
sudo nvim /etc/systemd/system/home-assistant\@homeassistant.service
</code-block>
<p>Change  the execution line runtime to python3.13:</p>
<code>
ExecStart=/srv/homeassistant/bin/python3.13 -m homeassistant --config /home/homeassistant/.homeassistant
</code>
</step>

<step>
<p>Reload the service</p>
<code-block lang="bash">
sudo systemctl daemon-reload
</code-block>
</step>

<step>
<p>Restart the service</p>
<code-block lang="bash">
sudo systemctl start home-assistant@homeassistant.service
</code-block>
</step>
</procedure>

