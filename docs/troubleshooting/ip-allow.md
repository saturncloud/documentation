# IP Allow Lists

## IP Allow Lists

If you are finding that your services that need to connect with Saturn Cloud have limits on what IP addresses may access them, you'll need to add Saturn Cloud IP addresses to the "allow list". There is no way to identify the machines that will be spun up in Saturn, because they are created programmatically. However, all of the traffic from a Saturn deployment comes from an Internet Gateway that lives inside the Amazon Virtual Private Cloud holding the EC2 instances you create. 

<em>If you're a Hosted user struggling with this issue, please <a href="/docs">contact us</a> and we can discuss how to help.</em>

You can find your IP address for Saturn Cloud in your AWS console, as shown below.

On the EC2 Dashboard, you can click on "Elastic IPs" on the left hand menu.

<img src="/images/docs/aws-network-pane.webp" style="width: 300px" alt="Screenshot inside AWS account EC2 Dashboard, showing side menu" class="doc-image">

Look for an Elastic IP labeled `saturn-cluster-${company-name}-vpc`.  The IP Address listed there is the Public IPv4 address that all data services you run will see when users of Saturn are trying to connect.

<img src="/images/docs/aws-network-pane2.webp" alt="Screenshot inside AWS account EC2 Dashboard, list of Elastic IP Addresses" class="doc-image">

You can validate this in the Jupyter notebook.

<img src="/images/docs/verify-ip.webp" alt="Screenshot of Jupyter chunk inside Saturn Cloud, showing code to verify an IP address" class="doc-image">

<!-- TODO remove this screenshot and change to a code chunk -->
