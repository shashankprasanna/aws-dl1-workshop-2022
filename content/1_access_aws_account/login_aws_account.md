---
title: "1.1 Login to your temporary workshop AWS Account"
weight: 1
chapter: false
---


#### Get your temporary AWS account

Click on the link at the bottom of the browser as show below.

![](/images/setup/setup1.jpg)

![](/images/setup/setup2.jpg)

![](/images/setup/setup3.jpg)

![](/images/setup/setup4.jpg)

![](/images/setup/setup5.jpg)

![](/images/setup/setup6.jpg)

![](/images/setup/setup7.jpg)

![](/images/setup/setup8.jpg)

#### If using Windows
Save file to a convenient location such as your Desktop

![](/images/setup/setup9.jpg)

#### If using MAC/Ubuntu
1. Save file to a convenient location such as Desktop
1. You'll need to perform update the file's permissions by running the code below the screenshot in your terminal

{{% notice tip %}}
Note: advanced users may optionally save it at `~/.ssh/` and make sure to use the correct path when using it to connect to your DL1 EC2 instance
{{% /notice %}}

![](/images/setup/setup10.jpg)

```
cd Desktop
chmod 400 labsuser.pem
```
{{% notice warning %}}
Note: If you saved the labsuser.pem at a different location, make sure to use the correct path.
{{% /notice %}}

![](/images/setup/setup12.jpg)

![](/images/setup/setup13-1.jpg)
