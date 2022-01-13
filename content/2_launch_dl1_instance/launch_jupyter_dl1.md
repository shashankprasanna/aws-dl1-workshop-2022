---
title: "2.2 Launch Jupyterlab server"
weight: 3
chapter: false
---

{{% notice tip %}}
Feel free to follow along with the presenter on the stage or on livestream
{{% /notice %}}

#### Launch Jupyterlab server on your DL1 instance


![](/images/setup/setup21.jpg)

#### SSH to your DL1 instance
```
cd
cd Desktop
ssh -i labsuser.pem ubuntu@<PASTE_IP_ADDRESS>
```

{{% notice tip %}}
Note: Make sure you give the full path to the `labsuser.pem` file or navigate to the directory where you saved the file before running SSH
{{% /notice %}}

![](/images/setup/setup22.jpg)
![](/images/setup/setup23.jpg)

### Update `python3` to point to python3.7 with Habana support

```
sudo rm /usr/bin/python3
sudo ln -s /usr/bin/python3.7 /usr/bin/python3
```

### Install Jupyterlab server
```
pip3 install jupyterlab
```

### Launch Jupyterlab server and setup port forwarding

{{% notice warning %}}
Note: For this step to work, please disconnect from VPN or corporate network which may block port forwarding.
{{% /notice %}}

##### In the current terminal connected to DL1 instance, run the following:

```
jupyter notebook --ip='*' --NotebookApp.token='' --NotebookApp.password=''
```
##### This should launch the Jupyter server. Output should like this:
![](/images/setup/setup23-1.jpg)

##### In a second new terminal window setup port forwarding. This step is needed for your computer to access Jupyter server on Amazon EC2
```
ssh -i labsuser.pem -N -L 0.0.0.0:8888:localhost:8888 -L 0.0.0.0:6006:localhost:6006 ubuntu@<IP_ADDRESS>
```
### You must have 2 terminal windows open. 1st with Jupyter Server running and 2nd with port forwarding.

![](/images/setup/setup24.jpg)

### Open a browser and type
```
localhost:8888/lab
```
### You should see Jupyter lab client on your browser, with the server running on DL1 instance

{{% notice tip %}}
Use the discord chatroom if you have trouble setting up Jupyter Lab. One of the TAs will assist you.
{{% /notice %}}

![](/images/setup/setup25.jpg)
