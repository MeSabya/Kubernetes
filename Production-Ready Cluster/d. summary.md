Summary
We have a production-ready Kubernetes cluster running in AWS. Isn’t that something worth a celebration?

Kops proved to be relatively easy to use. We executed more aws than kops commands. If we exclude them, the whole cluster can be created with a single kops command. We can easily add or remove worker nodes. Upgrades are simple and reliable, if a bit long. The important part is that through rolling upgrades we can avoid downtime.

There are a few kops command we did not explore. We feel that now you know the important parts and that you will be able to figure out the rest through the documentation.

You might be inclined to think that you are ready to apply everything you learned so far. We are not done yet. There’s still one significant topic we need to explore.

We postponed the discussion about stateful services since we did not have the ability to use external drives. We did use volumes, but they were all local, and do not qualify as persistent. Failure of a single server would prove that. Now that we are running a cluster in AWS, we can explore how to deploy stateful applications.
